## logback(-test).xml 
```java 
<configuration debug="true" scan="true" scanPeriod="30 seconds" > //scan-dynamically scan if logback config file is changed.scanPeriod-set scan period
  <property name="LOG_ARCHIVE" value="${LOG_PATH}/archive"/> //set property
  <timestamp key="timestamp-by-second" datePattern="yyyyMMdd'T'HHmmss"/> //timestamp pattern property
  
  <appender name="Console-Appender" class="ch.qos.Logback.core.ConsoleAppender">
    <layout>
        <pattern>%msg%n</pattern>
    </layout>
</appender>
<appender name="File-Appender" class="ch.qos.Logback.core.FileAppender">
    <file>${LOG_PATH}/logfile-${timestamp-by-second}.log</file>
    <encoder>
        <pattern>%msg%n</pattern>
        <outputPatternAsHeader>true</outputPatternAsHeader>
    </encoder>
</appender>
<appender name="RollingFile-Appender" class="ch.qos.Logback.core.rolling.RollingFileAppender">
    <file>${LOG_PATH}/rollingfile.log</file>//log file path
    <rollingPolicy class="ch.qos.Logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>${LOG_ARCHIVE}/rollingfile.%d{yyyy-MM-dd}.log</fileNamePattern> //rolling file path
        <maxHistory>30</maxHistory>//max number of day to remain rolling file,before deleting older files asynchronously.
        <totalSizeCap>1MB</totalSizeCap>//max size of all rolling file,before deleting older files asynchronously.
    </rollingPolicy>
    <encoder>
        <pattern>%msg%n</pattern>
    </encoder>
</appender>
<appender name="Async-Appender" class="ch.qos.logback.classic.AsyncAppender">
        <appender-ref ref="RollingFile-Appender" />
 </appender>
 
 <logger name="guru.springframework.blog.logbackxml"  level="info" additivity="false">//additivity=false-exlude log from root logger
        <appender-ref ref="Console-Appender" />
        <appender-ref ref="File-Appender" />
        <appender-ref ref="Async-Appender" />
 </logger>

<root>//root logger
    <appender-ref ref="Console-Appender"/>
</root>
</configuration>
```
