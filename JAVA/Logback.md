# Logback

>[中文文档](http://www.logback.cn/)
>[官方文档](http://logback.qos.ch/manual/index.html)

## 简介

- Logback 继承自 log4j
- 由 `logback-core`, `logback-classic`, `logback-access` 组成
-  `logback-core` 是核心模块
-  `logback-classic` 是 log4j 的优化版，天然支持 SLF4J
-  `logback-access` 提供 Http 访问日志的功能
- logback 默认使用 `ConsoleAppender` 作为 Appender
- `StatusManager` 可以获取在 logback 生命周期中 logback 的内部状态
- `Appender` 类被看作日志输入的目的地。包括 `console, files, Syslog, TCP Sockets, JMS(Java Message Server)`

## 架构

### Logger, Appender 和 Layouts

- Logger 属于 logback-classic 的一部分
- Appender 和 Layouts 是接口，作为 logback-core 的一部分。logback-core 是一个通用的模块，所以没有 Logger 的概念

### Logger 上下文

- 在 logback-classic 中，每个 logger 都依附在 `LoggerContext` 上，它负责生产 logger，通过一个 `树状` 的层级结构来进行管理
- root logger 作为 logger 层次结构的最高层。它是一个特殊的 logger，它是每个层次结构的一部分，每个 logger 都可以通过它的名字去获取
```java
Logger anyLogger = LoggerFactory.getLogger(org.slf4j.Logger.ROOT_LOGGER_NAME);
```

### Logger 等级

 - Logger 被分为不同的等级（TRACE，DEBUG，INFO，WARN，ERROR）定义在 `ch.qos.logback.classic.Level` 类中
 - Logger 层级的定义：`一个 Logger 的有效层级为，从它自身开始（自身也包含）一直回溯到 root logger，直到找到第一个不为空的层级作为自己的层级`

### 日志打印级别

- 日志的打印级别为 p，Logger 实例的级别为 q，如果 p >= q，则该条日志可以打印出来
- TRACE < DEBUG < INFO < WARN < ERROR
- 总结：`Logger 实例只能打印，日志级别大于等于实例级别的日志。比如，Logger 的等级为 INFO，那么只能打印 info(), warn(), error()`

### 获取 Logger

- 通过 `LoggerFactory.getLogger()` 可以获取到具体的 logger 实例，名字相同则返回的 logger 实例也相同
```java
Logger x = LoggerFactory.getLogger("matt");
Logger y = LoggerFactory.getLogger("matt");
```
- logback 环境的配置会在应用初始化的时候完成。最优的方式是通过读取配置文件
- `根据类的全限定名来对 logger 进行命名是最好的方式，没有之一`

### Appender 和 Layout

- 通过设置 `additivity = false` 使 appender 不再具有叠加性
- logger L 的日志输出语句会遍历 L 和它的子级中所有的 appender（叠加性）
- 如果 L 的子集为 P，且 P 设置了 `additivity=false`，那么 L 的日志会在 L 所有的 appender 包括 P 本身的 appender 中输出，但是不会再 P 的子级 appender 中输出。logger 默认的 `additivity=true`
- PatternLayout 能够根据用户指定的格式来格式化日志，类似于 C 语言的 printf 函数
```java
// 无论是否真正执行写日志的操作，都会拼接 msg
logger.debug("The new entry is " + entry + ".");
// 如果禁止日志打印，那么不会执行 msg 中的转换，性能更好
logger.debug("The new entry is {}", entry);
```

### 实现步骤

![[logback_flow.gif]]

### 初始化步骤

- 类路径下寻找名为 logback-test.xml 的文件
- 如果没有找到，会继续寻找名为 logback.groovy 的文件
- 如果没有找到，会继续寻找名为 logback.xml 的文件
- 如果没有找到，将会通过 JDK 提供的 ServiceLoader 工具在类路径下寻找文件 META-INFO/services/ch.qos.logback.classic.spi.Configuator，该文件的内容实现了 Configurator 接口的实现类的全限定类名
- 如果以上都没有成功，logback 会通过 BasicConfigurator 为自己进行配置，日志将会全部打印在控制台


