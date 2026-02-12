---
title: Spring Aop 开发笔记
published: 2026-02-11T17:41:26.381Z
tags: [spring]
pinned: false
image: "./images/b4.jpg"
category: 开发笔记
draft: true
---

# Spring AOP 学习笔记

## 第一章：AOP 基础概念

### 什么是 AOP？

- **AOP**：面向切面编程（Aspect Oriented Programming），一种编程思想
- **核心理解**：面向特定方法进行编程，实现对某些方法的通用操作
- **通俗解释**：不需要修改原方法代码，就能为方法添加额外功能

### AOP 解决的问题

- **传统方式问题**：统计每个业务方法耗时，需要修改几百个方法，代码重复且繁琐
- **AOP 解决方案**：将重复逻辑（如记录耗时）抽取到独立模块中，统一管理

### AOP 的优势

1. **减少重复代码**：公共逻辑统一抽取
2. **代码无侵入**：不修改原始业务方法
3. **提高开发效率**：快速添加通用功能
4. **便于维护**：修改公共逻辑只需改一处

## 第二章：AOP 快速入门

### 开发步骤

```xml
<!-- 1. 引入依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

### 编写 AOP 程序

```java
@Component  // 交给 Spring 管理
@Aspect     // 声明为切面类
public class TimeAspect {
    
    // 环绕通知：统计方法执行耗时
    @Around("execution(* com.itheima.service.*.*(..))")
    public Object recordTime(ProceedingJoinPoint pjp) throws Throwable {
        // 1. 记录开始时间
        long start = System.currentTimeMillis();
        
        // 2. 执行原始方法
        Object result = pjp.proceed();
        
        // 3. 记录结束时间并计算耗时
        long end = System.currentTimeMillis();
        log.info("{} 执行耗时：{}ms", 
                pjp.getSignature(),  // 获取方法签名
                end - start);
        
        return result;
    }
}
```

### 关键注解说明

- `@Aspect`：声明当前类为切面类
- `@Around`：环绕通知注解
- `ProceedingJoinPoint`：连接点对象，用于执行原始方法

## 第三章：AOP 核心概念

### 五大核心概念

| 概念     | 英文       | 说明                                      |
| -------- | ---------- | ----------------------------------------- |
| 连接点   | Join Point | 可以被 AOP 控制的方法（如业务层所有方法） |
| 切入点   | Pointcut   | 实际被 AOP 控制的方法（通过表达式匹配）   |
| 通知     | Advice     | 抽取的共性功能（如记录耗时的方法）        |
| 切面     | Aspect     | 通知 + 切入点的对应关系                   |
| 目标对象 | Target     | 通知所应用的对象                          |

### 重要关系

- **切入点一定是连接点**，但连接点不一定是切入点
- **切面** = 通知 + 切入点
- **通知**分为多种类型，执行时机不同

## 第四章：AOP 通知类型

### 五种通知类型

| 通知类型   | 注解              | 执行时机           | 特点                           |
| ---------- | ----------------- | ------------------ | ------------------------------ |
| 前置通知   | `@Before`         | 目标方法执行前     | -                              |
| 后置通知   | `@After`          | 目标方法执行后     | 无论是否异常都执行             |
| 环绕通知   | `@Around`         | 目标方法执行前后   | 功能最强大，需手动调用原始方法 |
| 返回后通知 | `@AfterReturning` | 目标方法正常返回后 | 仅正常返回时执行               |
| 异常后通知 | `@AfterThrowing`  | 目标方法抛出异常后 | 仅异常时执行                   |

### 环绕通知注意事项

```java
@Around("pt()")
public Object aroundAdvice(ProceedingJoinPoint pjp) throws Throwable {
    // 1. 必须调用 proceed() 执行原始方法
    Object result = pjp.proceed();
    
    // 2. 返回值必须为 Object 类型
    return result;
}
```

### 抽取公共切入点表达式

```java
@Aspect
@Component
public class MyAspect {
    
    // 抽取公共切入点表达式
    @Pointcut("execution(* com.itheima.service.*.*(..))")
    public void pt() {}
    
    // 引用公共切入点
    @Before("pt()")
    public void beforeAdvice() {
        // ...
    }
}
```

## 第五章：切入点表达式

### 两种表达式写法

```java
// 1. execution：基于方法签名匹配（主流方式）
@Before("execution(public * com.itheima.service.*.*(..))")

// 2. @annotation：基于注解匹配
@Before("@annotation(com.itheima.anno.Log)")
```

### execution 表达式详解

```
execution(访问修饰符 返回值 包名.类名.方法名(参数) throws 异常)
         ----------   ----  ---------------- ----- --------
           可选       必选      可选          必选    可选
```

### 通配符说明

- `*`：单个独立的任意符号
  - 返回值位置：任意返回值类型
  - 包名位置：单级包名
  - 类名/方法名位置：任意类/方法
  - 参数位置：一个任意类型参数
- `..`：多个连续的任意符号
  - 包名位置：任意层级包
  - 参数位置：任意个任意类型参数

### 表达式示例

```java
// 匹配 service 包下所有类的所有方法
execution(* com.itheima.service.*.*(..))

// 匹配以 delete 开头的方法
execution(* com.itheima.service.*.delete*(..))

// 匹配指定包下所有方法（任意层级）
execution(* com.itheima..service.*.*(..))
```

### 表达式书写建议

1. 业务方法命名尽量规范，方便匹配
2. 优先基于接口描述，增强扩展性
3. 尽可能缩小匹配范围，提高性能

## 第六章：连接点信息获取

### 获取连接点信息

```java
@Before("pt()")
public void beforeAdvice(JoinPoint joinPoint) {
    // 1. 获取目标对象
    Object target = joinPoint.getTarget();
    
    // 2. 获取目标类名
    String className = joinPoint.getTarget().getClass().getName();
    
    // 3. 获取方法名
    String methodName = joinPoint.getSignature().getName();
    
    // 4. 获取方法参数
    Object[] args = joinPoint.getArgs();
    
    log.info("类：{}，方法：{}，参数：{}", 
             className, methodName, Arrays.toString(args));
}
```

### 注意事项

- **环绕通知**：使用 `ProceedingJoinPoint` 获取
- **其他通知**：使用 `JoinPoint` 获取
- `ProceedingJoinPoint` 继承自 `JoinPoint`，功能更强大

## 第七章：AOP 执行流程与原理

### 执行流程

1. Spring AOP 基于动态代理技术
2. 为目标对象生成代理对象
3. 代理对象与目标对象实现相同接口
4. 调用时实际执行代理对象的方法
5. 代理方法中包含通知逻辑

### 动态代理示例

```java
// 伪代码：生成的代理对象方法
public class DeptServiceProxy implements DeptService {
    public List<Dept> list() {
        // 1. 记录开始时间（通知逻辑）
        long start = System.currentTimeMillis();
        
        // 2. 调用目标对象原始方法
        List<Dept> result = target.list();
        
        // 3. 记录结束时间（通知逻辑）
        long end = System.currentTimeMillis();
        log.info("耗时：{}ms", end - start);
        
        return result;
    }
}
```

## 第八章：AOP 实际案例 - 操作日志记录

### 案例需求

记录系统增删改操作日志，包含：

- 操作人 ID、操作时间、操作类、操作方法
- 方法参数、返回值、执行耗时

### 实现方案

```java
@Aspect
@Component
@Slf4j
public class OperateLogAspect {
    
    @Autowired
    private OperateLogMapper operateLogMapper;
    
    // 基于注解匹配切入点
    @Around("@annotation(com.itheima.anno.Log)")
    public Object recordOperateLog(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        
        // 执行原始方法
        Object result = pjp.proceed();
        
        long end = System.currentTimeMillis();
        
        // 封装操作日志
        OperateLog operateLog = new OperateLog();
        operateLog.setOperatorId(UserContext.getCurrentUserId()); // 当前用户
        operateLog.setOperateTime(LocalDateTime.now());
        operateLog.setClassName(pjp.getTarget().getClass().getName());
        operateLog.setMethodName(pjp.getSignature().getName());
        operateLog.setMethodParams(Arrays.toString(pjp.getArgs()));
        operateLog.setReturnValue(result != null ? result.toString() : "void");
        operateLog.setCostTime(end - start);
        
        // 保存日志
        operateLogMapper.insert(operateLog);
        
        return result;
    }
}
```

### 自定义注解

```java
@Retention(RetentionPolicy.RUNTIME)  // 运行时生效
@Target(ElementType.METHOD)          // 只能加在方法上
public @interface Log {
    // 标识需要记录日志的方法
}
```

## 第九章：ThreadLocal 应用

### 问题背景

- 在 Filter 中解析 JWT 获取用户 ID
- 需要在 AOP 中获取该用户 ID
- 如何跨组件传递数据？

### ThreadLocal 解决方案

```java
// ThreadLocal 工具类
public class UserContext {
    private static final ThreadLocal<Integer> THREAD_LOCAL = new ThreadLocal<>();
    
    // 存储当前用户ID
    public static void setCurrentUserId(Integer userId) {
        THREAD_LOCAL.set(userId);
    }
    
    // 获取当前用户ID
    public static Integer getCurrentUserId() {
        return THREAD_LOCAL.get();
    }
    
    // 删除当前用户ID（防止内存泄漏）
    public static void removeCurrentUserId() {
        THREAD_LOCAL.remove();
    }
}
```

### 使用流程

1. **Filter 中存储**：解析 JWT → 获取用户 ID → 存入 ThreadLocal
2. **AOP 中获取**：从 ThreadLocal 取出用户 ID
3. **Filter 中清理**：请求完成后移除 ThreadLocal 数据

### ThreadLocal 特点

- 线程局部变量，每个线程独立存储空间
- 线程隔离，不同线程互不干扰
- 适用场景：同一线程/请求中的数据共享

## 第十章：AOP 应用场景总结

### 常见应用场景

1. **日志记录**：操作日志、执行耗时
2. **事务管理**：Spring 事务底层基于 AOP
3. **权限控制**：方法级权限校验
4. **性能监控**：方法执行时间统计
5. **缓存管理**：方法结果缓存
6. **异常处理**：统一异常捕获处理

### 开发规范建议

1. **切面类命名**：XxxAspect，如 TimeAspect、LogAspect
2. **通知方法命名**：体现功能，如 recordTime、recordLog
3. **切入点表达式**：尽量精确，避免过大范围影响性能
4. **线程安全**：注意通知方法中的线程安全问题

------

## 思维导图大纲

```
Spring AOP
├── 基础概念
│   ├── 什么是AOP：面向切面编程
│   ├── 解决的问题：代码重复、侵入性强
│   └── 核心优势：无侵入、易维护、高效率
├── 快速入门
│   ├── 引入依赖：spring-boot-starter-aop
│   ├── 编写切面：@Aspect + @Component
│   ├── 定义通知：@Around + ProceedingJoinPoint
│   └── 切入点表达式：execution()
├── 核心概念
│   ├── 连接点：JoinPoint，可被AOP控制的方法
│   ├── 切入点：Pointcut，实际被控制的方法
│   ├── 通知：Advice，抽取的共性功能
│   ├── 切面：Aspect，通知+切入点
│   └── 目标对象：Target，通知应用的对象
├── 通知类型
│   ├── @Before：前置通知
│   ├── @After：后置通知（始终执行）
│   ├── @Around：环绕通知（前后都执行）
│   ├── @AfterReturning：返回后通知（正常返回）
│   └── @AfterThrowing：异常后通知（异常时）
├── 切入点表达式
│   ├── execution：基于方法签名
│   │   ├── 语法：修饰符 返回值 包.类.方法(参数) throws 异常
│   │   ├── 通配符：*（单个任意）、..（多个任意）
│   │   └── 建议：精确匹配、基于接口
│   └── @annotation：基于注解
│       └── 自定义注解标记切入点方法
├── 连接点信息
│   ├── JoinPoint：获取方法执行信息
│   │   ├── getTarget()：目标对象
│   │   ├── getSignature()：方法签名
│   │   ├── getArgs()：方法参数
│   │   └── 环绕通知使用ProceedingJoinPoint
│   └── 应用：日志记录、参数校验
├── 执行原理
│   └── 动态代理：生成代理对象，增强目标方法
├── 实际案例
│   └── 操作日志记录
│       ├── 需求：记录增删改操作
│       ├── 实现：环绕通知+自定义注解
│       ├── 信息：操作人、时间、类、方法、参数、结果、耗时
│       └── 存储：数据库日志表
├── ThreadLocal应用
│   ├── 作用：线程局部变量，线程隔离
│   ├── 方法：set()、get()、remove()
│   ├── 场景：同一请求跨组件数据共享
│   └── 案例：Filter→AOP用户ID传递
└── 应用场景
    ├── 日志记录
    ├── 事务管理
    ├── 权限控制
    ├── 性能监控
    ├── 缓存管理
    └── 异常处理
```

------

## 关键总结

1. **AOP 本质**：一种编程思想，Spring 提供了具体实现
2. **核心价值**：解耦、代码复用、非侵入式编程
3. **开发重点**：合理使用通知类型、编写精确切入点表达式
4. **实际应用**：结合业务场景（如日志、事务）发挥最大价值
5. **注意事项**：性能影响、线程安全、合理使用 ThreadLocal





