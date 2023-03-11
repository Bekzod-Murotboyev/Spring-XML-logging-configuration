## _In this short tutorial, we're going to explore the main logging options available in Spring Boot._

### Zero Configuration Logging
_Spring Boot is a very helpful framework. It allows us to forget about the majority of the configuration settings, many of which it opinionatedly auto-tunes. In the case of logging, the only mandatory dependency is Apache Commons Logging. We need to import it only when using Spring 4.x (Spring Boot 1.x) since it's provided by Spring Framework’s spring-jcl module in Spring 5 (Spring Boot 2.x). We shouldn't worry about importing spring-jcl at all if we're using a Spring Boot Starter (which we almost always are). That's because every starter, like our spring-boot-starter-web, depends on spring-boot-starter-logging, which already pulls in spring-jcl for us._


_Even though the default configuration is useful (for example, to get started in zero time during POCs or quick experiments), it's most likely not enough for our daily needs. Let's see how to include a Logback configuration with a different color and logging pattern, with separate specifications for console and file output, and with a decent rolling policy to avoid generating huge log files. First, we should find a solution that allows for handling our logging settings alone instead of polluting application.properties, which is commonly used for many other application settings._

### Logback Configuration Logging
#### When a file in the classpath has one of the following names, Spring Boot will automatically load it over the default configuration:
* logback-spring.xml
* logback.xml
* logback-spring.groovy
* logback.groovy

Let's write a simple logback-spring.xml:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <property name="LOGS" value="./logs" />

    <appender name="Console"
        class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <Pattern>
                %black(%d{ISO8601}) %highlight(%-5level) [%blue(%t)] %yellow(%C{1.}): %msg%n%throwable
            </Pattern>
        </layout>
    </appender>

    <appender name="RollingFile"
        class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOGS}/spring-boot-logger.log</file>
        <encoder
            class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <Pattern>%d %p %C{1.} [%t] %m%n</Pattern>
        </encoder>

        <rollingPolicy
            class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- rollover daily and when the file reaches 10 MegaBytes -->
            <fileNamePattern>${LOGS}/archived/spring-boot-logger-%d{yyyy-MM-dd}.%i.log
            </fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy
                class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
    </appender>
    
    <!-- LOG everything at INFO level -->
    <root level="info">
        <appender-ref ref="RollingFile" />
        <appender-ref ref="Console" />
    </root>

    <!-- LOG "com.baeldung*" at TRACE level -->
    <logger name="com.baeldung" level="trace" additivity="false">
        <appender-ref ref="RollingFile" />
        <appender-ref ref="Console" />
    </logger>

</configuration>

```

## Log4j2 Configuration Logging
_While Apache Commons Logging is at the core, and Logback is the reference implementation provided, all the routings to the other logging libraries are already included to make it easy to switch to them._

#### In order to use any logging library other than Logback, though, we need to exclude it from our dependencies:

```groovy
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")

    implementation("org.springframework.boot:spring-boot-starter-log4j2")
}

configurations.all {
    exclude(group = "org.springframework.boot", module = "spring-boot-starter-logging")
}
```

#### At this point, we need to place in the classpath a file named one of the following:
* log4j2-spring.xml
* log4j2.xml

Let's write a simple log4j2-spring.xml:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration>
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout
                    disableAnsi="false"
                    pattern="%style{%d}{bright,white} %highlight{%-5level }[%style{%t}{bright,blue}] %style{%C{1.}}{bright,green}: %msg%n%throwable"/>
        </Console>

        <RollingFile name="RollingFile"
                     fileName="./logs/logger.txt"
                     filePattern="./logs/$${date:yyyy-MM}/logger-%d{dd-MMMM-yyyy}-%i.log.txt">
            <PatternLayout>
                <pattern>%d %p %C{1.} [%t] %m%n</pattern>
            </PatternLayout>
            <Policies>
                <!-- rollover on startup, daily and when the file reaches
                    10 MegaBytes -->
                <OnStartupTriggeringPolicy/>
                <SizeBasedTriggeringPolicy
                        size="10 MB"/>
                <TimeBasedTriggeringPolicy/>
            </Policies>
        </RollingFile>
    </Appenders>

    <Loggers>
        <!-- LOG everything at INFO level -->
        <Root level="info">
            <AppenderRef ref="Console"/>
            <AppenderRef ref="RollingFile"/>
        </Root>
    </Loggers>

</Configuration>
```

