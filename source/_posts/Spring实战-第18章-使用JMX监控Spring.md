---
title: 第18章 使用JMX监控Spring
date: 2026-02-17 15:10:54
categories:
- [编程技术, Spring]
tags:
- Spring实战
---
## 本章概览

Java管理扩展（**JMX**）是监控和管理Java应用程序的标准方式。通过暴露称为 **MBean**（managed bean）的受管组件，外部JMX客户端可以通过调用操作、检查属性和监听事件来管理应用程序。

本章涵盖三个核心主题：

- 使用 Actuator 端点 MBean
- 将 Spring Bean 暴露为 MBean
- 发送 JMX 通知

---

## 18.1 使用 Actuator MBean

### 默认行为

- 所有 Actuator 端点**默认**都会作为 MBean 暴露
- 但从 Spring Boot 2.2 开始，**JMX 本身默认是关闭的**，需要手动开启：

```
spring.jmx.enabled=true
```

- MBean 位于 `org.springframework.boot` 域下，可以通过 **JConsole** 等 JMX 客户端查看

### 端点暴露控制

- 通过 `management.endpoints.jmx.exposure.include` 和 `management.endpoints.jmx.exposure.exclude` 控制暴露哪些端点：

```
# 只暴露 health, info, bean 端点
management.endpoints.jmx.exposure.include=health,info,bean

# 排除某些端点
management.endpoints.jmx.exposure.exclude=env,metrics
```

- 也可以使用 `*` 通配符暴露所有端点：

```
management.endpoints.jmx.exposure.include=*
```

### 使用 JConsole 访问 Actuator MBean

1. 启动应用并确保 JMX 已开启
2. 在终端运行 `jconsole`
3. 选择本地进程连接
4. 在 **MBeans** 选项卡中找到 `org.springframework.boot` 域
5. 展开 Endpoint 节点即可看到各个 Actuator 端点对应的 MBean

---

## 18.2 创建自定义 MBean

### 使用 Spring 注解暴露 Bean

Spring 提供了一组注解，可以将任意 Spring Bean 暴露为 JMX MBean：

- `@ManagedResource` — 标记类为 MBean
- `@ManagedAttribute` — 暴露属性（getter/setter）
- `@ManagedOperation` — 暴露方法为可调用操作

### 示例

```java
@Service
@ManagedResource
public class TacoCounter extends AbstractMBean {

    private AtomicLong counter;

    public TacoCounter(MBeanExporter exporter) {
        counter = new AtomicLong(0);
    }

    @ManagedAttribute
    public long getTacoCount() {
        return counter.get();
    }

    @ManagedOperation
    public long increment(long amount) {
        return counter.addAndGet(amount);
    }
}
```

> **要点：** 只需添加这些注解，Spring Boot 就会自动将 Bean 注册到 MBeanServer，无需额外的 XML 配置。
> 

### MBean 命名

- 默认情况下，MBean 的名称基于 Bean 的全限定类名
- 可以通过 `@ManagedResource` 的 `objectName` 属性自定义名称：

```java
@ManagedResource(objectName = "tacos:name=TacoCounter")
```

---

## 18.3 发送通知（Notification）

### 什么是 JMX 通知

MBean 不仅可以被动地接受查询，还可以**主动推送通知**给 JMX 客户端。Spring 通过 `NotificationPublisher` 接口支持此功能。

### 实现步骤

1. 实现 `NotificationPublisherAware` 接口
2. 通过 `setNotificationPublisher()` 获取 `NotificationPublisher` 实例
3. 在合适的时机调用 `sendNotification()` 发布通知

### 示例

```java
@Service
@ManagedResource
public class TacoCounter
        implements NotificationPublisherAware {

    private AtomicLong counter;
    private NotificationPublisher np;

    @Override
    public void setNotificationPublisher(
            NotificationPublisher notificationPublisher) {
        this.np = notificationPublisher;
    }

    @ManagedOperation
    public long increment(long amount) {
        long before = counter.get();
        long after = counter.addAndGet(amount);

        // 每生产100个 taco 发送一次通知
        if ((after / 100) > (before / 100)) {
            Notification notification = new Notification(
                "taco.count", this,
                before, after + "th taco created!");
            np.sendNotification(notification);
        }
        return after;
    }
}
```

> **要点：** 通知机制允许 MBean 在特定事件发生时主动向 JMX 客户端推送信息，这比客户端轮询更加高效。
> 

---

## 本章小结

<aside>
📌

- **Actuator 端点自动暴露为 MBean**，可通过 JConsole 等工具管理，但 JMX 需要手动开启
- 使用 `@ManagedResource`、`@ManagedAttribute`、`@ManagedOperation` 注解可以将**任意 Spring Bean 暴露为自定义 MBean**
- 通过实现 `NotificationPublisherAware` 接口，MBean 可以**主动发送 JMX 通知**
</aside>