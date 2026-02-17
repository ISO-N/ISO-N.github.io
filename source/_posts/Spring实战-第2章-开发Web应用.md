---
title: 第2章 开发Web应用
date: 2026-02-17 15:10:42
categories:
- [编程技术, Spring]
tags:
- Spring实战
---
本章围绕 **Taco Cloud** 应用，介绍如何使用 Spring MVC 开发 Web 应用：展现模型数据、处理和校验表单输入、选择视图模板库。

---

## 2.1 展现信息

在 Spring Web 应用中：**控制器**负责获取和处理数据，**视图**负责将数据渲染为 HTML。

要实现 Taco 设计页面，需要以下组件：

- 定义 taco 配料属性的**领域类**
- 获取配料信息并传递给视图的 **Spring MVC 控制器类**
- 在浏览器中渲染配料列表的**视图模板**

### 2.1.1 构建领域类

- **领域（Domain）**：应用所要解决的主题范围，即影响应用理解的理念和概念
- Taco Cloud 的领域对象包括 taco 设计、组成设计的配料、顾客订单等
- 使用普通 Java 对象（POJO）定义领域类，可使用 **Lombok** 简化代码

```java
// 配料类
@Data
@RequiredArgsConstructor
public class Ingredient {
    private final String id;
    private final String name;
    private final Type type;

    public static enum Type {
        WRAP, PROTEIN, VEGGIES, CHEESE, SAUCE
    }
}
```

```java
// Taco 类
@Data
public class Taco {
    private String name;
    private List<String> ingredients;
}
```

```java
// 订单类
@Data
public class Order {
    private String name;
    private String street;
    private String city;
    private String state;
    private String zip;
    private String ccNumber;
    private String ccExpiration;
    private String ccCVV;
}
```

> **Lombok**：通过 `@Data` 自动生成 getter/setter、`toString()`、`equals()`、`hashCode()` 等方法，`@RequiredArgsConstructor` 生成包含 final 字段的构造器。需添加为项目依赖。
> 

### 2.1.2 创建控制器类

控制器的主要职责：

- 处理 HTTP 请求
- 将请求传递给视图以渲染 HTML，或直接将数据写入响应体（RESTful）

**核心注解：**

- `@Controller`：标识该类为控制器，纳入 Spring 组件扫描
- `@RequestMapping`：类级别，声明该控制器处理的请求类型
- `@GetMapping`：处理 HTTP GET 请求（对应还有 `@PostMapping`、`@PutMapping`、`@DeleteMapping`、`@PatchMapping`）

```java
@Slf4j
@Controller
@RequestMapping("/design")
@SessionAttributes("order")
public class DesignTacoController {

    @ModelAttribute
    public void addIngredientsToModel(Model model) {
        List<Ingredient> ingredients = Arrays.asList(
            new Ingredient("FLTO", "Flour Tortilla", Type.WRAP),
            new Ingredient("COTO", "Corn Tortilla", Type.WRAP),
            // ...其他配料
        );
        Type[] types = Ingredient.Type.values();
        for (Type type : types) {
            model.addAttribute(type.toString().toLowerCase(),
                filterByType(ingredients, type));
        }
    }

    @GetMapping
    public String showDesignForm(Model model) {
        model.addAttribute("design", new Taco());
        return "design"; // 返回视图名称
    }
}
```

<aside>
💡

**Model 对象**：充当控制器和视图之间传输数据的容器。放到 Model 属性中的数据会复制到 Servlet Response 的属性中，视图可以在其中找到并使用它们。

</aside>

### 2.1.3 设计视图

- Spring 支持多种视图模板方案：Thymeleaf、FreeMarker、Mustache、JSP 等
- 视图模板库作为依赖添加后，Spring Boot 会**自动配置**对应的视图解析器
- 模板放在 `/src/main/resources/templates/` 目录下

**Thymeleaf 示例：**

- `th:each`：遍历集合
- `th:value`：设置表单元素值
- `th:text`：输出文本
- `th:object`：绑定表单对象
- `th:field`：绑定字段（自动处理 id、name、value）

```html
<div th:each="ingredient : ${wrap}">
    <input name="ingredients" type="checkbox"
           th:value="${ingredient.id}" />
    <span th:text="${ingredient.name}">INGREDIENT</span>
</div>
```

<aside>
📌

Thymeleaf 模板就是增加了一些额外属性的 HTML。在没有经过 Spring 处理时也能作为静态 HTML 在浏览器中正常显示（**自然模板**），利于设计时预览。

</aside>

---

## 2.2 处理表单提交

- 视图中的 `<form>` 提交后，浏览器会收集所有表单数据，以 HTTP POST 请求发送到服务器
- 使用 `@PostMapping` 注解处理 POST 请求
- 控制器方法的参数对象会**自动绑定**表单提交的字段（名称匹配）

```java
@PostMapping
public String processDesign(Taco design) {
    // 保存 taco 设计...
    log.info("Processing design: " + design);
    return "redirect:/orders/current";  // 重定向到订单页
}
```

```java
@Controller
@RequestMapping("/orders")
public class OrderController {

    @GetMapping("/current")
    public String orderForm(Model model) {
        model.addAttribute("order", new Order());
        return "orderForm";
    }

    @PostMapping
    public String processOrder(Order order) {
        log.info("Order submitted: " + order);
        return "redirect:/";
    }
}
```

<aside>
💡

**`redirect:` 前缀**：表示这是一个重定向视图，而非直接渲染。浏览器会收到重定向响应后发起新的 GET 请求，避免表单重复提交。

</aside>

---

## 2.3 校验表单输入

使用 **Java Bean Validation API**（`javax.validation`），通过在领域类字段上添加校验注解来声明规则。Spring Boot 的 Validation starter 自动引入 Hibernate Validator。

### 2.3.1 声明校验规则

```java
@Data
public class Taco {
    @NotNull
    @Size(min = 5, message = "Name must be at least 5 characters long")
    private String name;

    @Size(min = 1, message = "You must choose at least 1 ingredient")
    private List<String> ingredients;
}
```

```java
@Data
public class Order {
    @NotBlank(message = "Name is required")
    private String name;

    @NotBlank(message = "Street is required")
    private String street;

    @NotBlank(message = "City is required")
    private String city;

    @NotBlank(message = "State is required")
    private String state;

    @NotBlank(message = "Zip code is required")
    private String zip;

    @CreditCardNumber(message = "Not a valid credit card number")
    private String ccNumber;

    @Pattern(regexp = "^(0[1-9]|1[0-2])([\\/])([1-9][0-9])$",
             message = "Must be formatted MM/YY")
    private String ccExpiration;

    @Digits(integer = 3, fraction = 0, message = "Invalid CVV")
    private String ccCVV;
}
```

**常用校验注解：**

| **注解** | **说明** |
| --- | --- |
| `@NotNull` | 值不能为 null |
| `@NotBlank` | 不为 null 且去除首尾空格后长度 > 0 |
| `@Size(min, max)` | 集合/字符串大小在 min 和 max 之间 |
| `@CreditCardNumber` | 通过 Luhn 算法检查（Hibernate Validator 提供） |
| `@Pattern(regexp)` | 匹配正则表达式 |
| `@Digits(integer, fraction)` | 数值位数校验 |

### 2.3.2 在表单绑定的时候执行校验

- 在控制器方法参数上添加 `@Valid` 注解触发校验
- 校验结果通过 `Errors` 对象获取

```java
@PostMapping
public String processDesign(@Valid Taco design, Errors errors) {
    if (errors.hasErrors()) {
        return "design";  // 有错误则返回表单页
    }
    log.info("Processing design: " + design);
    return "redirect:/orders/current";
}

@PostMapping
public String processOrder(@Valid Order order, Errors errors) {
    if (errors.hasErrors()) {
        return "orderForm";
    }
    log.info("Order submitted: " + order);
    return "redirect:/";
}
```

### 2.3.3 展现校验错误

Thymeleaf 通过 `fields` 工具对象和 `th:errors` 属性展示校验错误：

```html
<span class="validationError"
      th:if="${#fields.hasErrors('ccNumber')}"
      th:errors="*{ccNumber}">CC Num Error</span>
```

---

## 2.4 使用视图控制器

- 对于不需要模型数据或处理输入、只返回视图的简单控制器，可以使用**视图控制器**替代
- 实现 `WebMvcConfigurer` 接口来声明视图控制器

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("home");
    }
}
```

<aside>
💡

视图控制器适用于不需要任何业务逻辑、仅做 URL → 视图映射的场景，可以减少不必要的控制器类。

</aside>

---

## 2.5 选择视图模板库

Spring Boot 支持的主要模板引擎：

| **模板引擎** | **Spring Boot starter 依赖** |
| --- | --- |
| Thymeleaf | `spring-boot-starter-thymeleaf` |
| FreeMarker | `spring-boot-starter-freemarker` |
| Mustache | `spring-boot-starter-mustache` |
| Groovy Templates | `spring-boot-starter-groovy-templates` |
| JSP（不推荐） | 无需 starter，但不能与 JAR 打包一起使用 |
- 只需添加对应的 starter 依赖，Spring Boot 即可**自动配置**模板引擎所需的组件
- 模板文件统一放在 `/src/main/resources/templates/` 下
- **JSP 不推荐**：需要部署到 Servlet 容器中，不支持 JAR 打包方式

### 缓存模板

- 默认情况下，模板只在第一次使用时解析一次，解析结果会被**缓存**以提升性能
- **开发时**建议禁用缓存（例如 `spring.thymeleaf.cache=false`），以便修改模板后立即看到效果
- **生产环境**中应保持缓存开启
- 使用 **DevTools** 时会自动禁用模板缓存

---

## 2.6 小结

- Spring MVC 是一个强大的 Web 框架，基于**注解驱动**的编程模型
- 控制器使用 `@Controller`、`@RequestMapping`、`@GetMapping`、`@PostMapping` 等注解来处理请求
- **Model** 对象用于在控制器和视图之间传递数据
- 表单提交通过 `@PostMapping` 处理，表单字段自动绑定到领域对象
- 使用 **Java Bean Validation API** 校验表单输入，搭配 `@Valid` 和 `Errors` 对象
- **视图控制器**适用于只做 URL → 视图映射的简单场景
- Spring Boot 通过 starter 依赖自动配置 Thymeleaf 等多种视图模板库