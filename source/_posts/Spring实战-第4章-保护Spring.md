---
title: 第4章 保护Spring
date: 2026-02-17 15:10:43
categories:
- [编程技术, Spring]
tags:
- Spring实战
---
## 本章概要

- 自动配置 Spring Security
- 自定义用户存储
- 自定义登录页面
- 防御 CSRF 攻击
- 了解你的用户

---

## 4.1 启用 Spring Security

- 添加 Spring Security 起步依赖即可启用安全功能：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

- 引入依赖后，Spring Boot 自动配置会提供以下默认安全行为：
    - 所有 HTTP 请求路径都需要认证
    - 不需要特定的角色或权限
    - 没有登录页面，使用 **HTTP Basic 认证**弹窗
    - 只有一个用户 `user`，密码在启动日志中随机生成

> 默认的安全配置只是一个起点，实际应用中需要自定义用户存储、登录页面和访问规则等。
> 

### 配置 Spring Security

- 创建一个配置类，使用 `@Configuration` 注解
- 定义 `SecurityFilterChain` Bean 来配置 HTTP 安全规则

```java
@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .authorizeRequests()
                .antMatchers("/design", "/orders").hasRole("USER")
                .antMatchers("/", "/**").permitAll()
            .and()
            .build();
    }
}
```

## 4.2 配置用户存储

- Spring Security 支持多种用户存储方式：
    - **基于内存的用户存储**
    - **基于 JDBC 的用户存储**
    - **基于 LDAP 的用户存储**
    - **自定义用户详情服务**

### 基于内存的用户存储

- 适用于测试或只有少量用户且不会改变的场景

```java
@Bean
public UserDetailsService userDetailsService(PasswordEncoder encoder) {
    List<UserDetails> usersList = new ArrayList<>();
    usersList.add(new User("buzz", encoder.encode("password"),
        Arrays.asList(new SimpleGrantedAuthority("ROLE_USER"))));
    usersList.add(new User("woody", encoder.encode("password"),
        Arrays.asList(new SimpleGrantedAuthority("ROLE_USER"))));
    return new InMemoryUserDetailsManager(usersList);
}
```

### 基于 JDBC 的用户存储

- 用户信息保存在关系型数据库中
- 使用 `JdbcUserDetailsManager` 实现

```java
@Bean
public UserDetailsService userDetailsService(DataSource dataSource) {
    return new JdbcUserDetailsManager(dataSource);
}
```

- 默认情况下会查找预定义的表结构（`users`、`authorities` 表）
- 可通过 `usersByUsernameQuery()` 和 `authoritiesByUsernameQuery()` 自定义查询 SQL

### 基于 LDAP 的用户存储

- 使用 `LdapUserDetailsManager` 来对 LDAP 服务器进行认证
- 需要配置 LDAP 服务器地址和搜索基础
- 支持密码比对和绑定认证两种方式
- 可使用嵌入式 LDAP 服务器进行开发和测试

### 自定义用户详情服务（推荐）

- 实现 `UserDetailsService` 接口，只需定义一个 `loadUserByUsername()` 方法
- 可以对接任何数据源（如 JPA Repository）

```java
@Service
public class UserRepositoryUserDetailsService implements UserDetailsService {

    private UserRepository userRepo;

    @Autowired
    public UserRepositoryUserDetailsService(UserRepository userRepo) {
        this.userRepo = userRepo;
    }

    @Override
    public UserDetails loadUserByUsername(String username)
            throws UsernameNotFoundException {
        User user = userRepo.findByUsername(username);
        if (user != null) {
            return user;
        }
        throw new UsernameNotFoundException("User '" + username + "' not found");
    }
}
```

- 自定义 `User` 实体类需要实现 `UserDetails` 接口，提供以下方法：
    - `getAuthorities()`：返回授权列表
    - `getPassword()`、`getUsername()`：返回凭证
    - `isAccountNonExpired()`、`isAccountNonLocked()`、`isCredentialsNonExpired()`、`isEnabled()`：账户状态判断

### 密码编码器

- Spring Security 要求密码必须经过编码存储，提供了多种 `PasswordEncoder` 实现：
    - `BCryptPasswordEncoder`：bcrypt 强哈希加密（**推荐**）
    - `NoOpPasswordEncoder`：不进行编码（仅用于测试）
    - `Pbkdf2PasswordEncoder`：PBKDF2 加密
    - `SCryptPasswordEncoder`：scrypt 加密
    - `StandardPasswordEncoder`：SHA-256 加密

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

## 4.3 保护 Web 请求

### 配置安全规则

- 使用 `SecurityFilterChain` 中的 `authorizeRequests()` 定义 URL 级别的访问规则
- 常用的访问控制方法：
    - `hasRole("ROLE")`：拥有指定角色
    - `hasAuthority("AUTHORITY")`：拥有指定权限
    - `permitAll()`：允许所有访问
    - `denyAll()`：拒绝所有访问
    - `authenticated()`：已认证即可
    - `anonymous()`：匿名用户可访问
- **规则顺序很重要**：先定义的规则优先级更高，应将最具体的规则放在前面

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    return http
        .authorizeRequests()
            .antMatchers("/design", "/orders").hasRole("USER")
            .antMatchers("/", "/**").permitAll()
        .and()
        .formLogin()
            .loginPage("/login")
        .and()
        .build();
}
```

### 自定义登录页面

- 调用 `formLogin()` 并使用 `loginPage()` 指定自定义登录页路径
- 默认情况下，Spring Security 监听 `/login` 的 POST 请求进行认证
- 可自定义：
    - `loginPage("/login")`：登录页面路径
    - `defaultSuccessUrl("/design")`：登录成功后默认跳转路径
    - `usernameParameter("username")`：用户名参数名
    - `passwordParameter("password")`：密码参数名

```java
.formLogin()
    .loginPage("/login")
    .defaultSuccessUrl("/design", true)
```

- 需要编写一个控制器将 `/login` 映射到登录视图：

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/login").setViewName("login");
    }
}
```

### 登出

- 调用 `logout()` 配置登出行为
- 默认拦截 `/logout` 的 POST 请求
- 登出后清除 Session 并重定向到登录页

```java
.and()
.logout()
    .logoutSuccessUrl("/")
```

## 4.4 防御 CSRF 攻击

- **CSRF（跨站请求伪造）**：攻击者利用用户已认证的身份，在用户不知情的情况下发送恶意请求
- Spring Security 默认启用 CSRF 防护
- 防护原理：为每个表单生成一个 **CSRF Token**，提交时服务端验证 Token 是否匹配
- 在 Thymeleaf 模板中，只要使用 `th:action` 属性，CSRF Token 会自动添加为隐藏字段

```html
<input type="hidden" name="_csrf" th:value="${_csrf.token}"/>
```

- **不要禁用 CSRF 防护**，除非你的客户端不是浏览器（如原生 App 或非浏览器客户端）

## 4.5 了解你的用户

- 获取当前登录用户信息的几种方式：

### 方式一：注入 Principal 对象

```java
@PostMapping
public String processOrder(@Valid Order order, Errors errors,
        Principal principal) {
    User user = userRepository.findByUsername(principal.getName());
    order.setUser(user);
    // ...
}
```

### 方式二：注入 Authentication 对象

```java
@PostMapping
public String processOrder(@Valid Order order, Errors errors,
        Authentication authentication) {
    User user = (User) authentication.getPrincipal();
    order.setUser(user);
    // ...
}
```

### 方式三：使用 @AuthenticationPrincipal 注解（推荐）

```java
@PostMapping
public String processOrder(@Valid Order order, Errors errors,
        @AuthenticationPrincipal User user) {
    order.setUser(user);
    // ...
}
```

- 这种方式最简洁，不依赖安全框架的具体类型，直接获取自定义 User 对象

### 方式四：使用 SecurityContextHolder

```java
Authentication authentication =
    SecurityContextHolder.getContext().getAuthentication();
User user = (User) authentication.getPrincipal();
```

- 适用于非控制器代码中获取用户信息（如服务层）

---

<aside>
💡

**小结**：Spring Security 通过自动配置提供了开箱即用的安全能力。实际开发中，需要自定义用户存储（推荐实现 `UserDetailsService`），配置 URL 级别的访问规则，自定义登录/登出页面，并始终保持 CSRF 防护开启。获取当前用户推荐使用 `@AuthenticationPrincipal` 注解。

</aside>