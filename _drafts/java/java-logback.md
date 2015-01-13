<?xml version="1.0" encoding="UTF-8"?>

<configuration debug="true" scan="true" scanPeriod="30 seconds"> 

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender"> 
    <!-- encoders are  by default assigned the type
         ch.qos.logback.classic.encoder.PatternLayoutEncoder -->
    <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss} [%level] - %m%n</pattern>
        
        <!-- 常用的Pattern变量,大家可打开该pattern进行输出观察 -->
        <!-- 
          <pattern>
              %d{yyyy-MM-dd HH:mm:ss} [%level] - %msg%n
              Logger: %logger
              Class: %class
              File: %file
              Caller: %caller
              Line: %line
              Message: %m
              Method: %M
              Relative: %relative
              Thread: %thread
              Exception: %ex
              xException: %xEx
              nopException: %nopex
              rException: %rEx
              Marker: %marker
              %n
              
          </pattern>
           -->
    </encoder>
  </appender>
  
  <!-- 按日期区分的滚动日志 -->
  <appender name="ERROR-OUT" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <file>logs/error.log</file>
      
    <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss} [%class:%line] - %m%n</pattern>
    </encoder>
      
      <filter class="ch.qos.logback.classic.filter.LevelFilter">
      <level>ERROR</level>
      <onMatch>ACCEPT</onMatch>
      <onMismatch>DENY</onMismatch>
    </filter>
      
      <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!-- daily rollover -->
      <fileNamePattern>error.%d{yyyy-MM-dd}.log.zip</fileNamePattern>

      <!-- keep 30 days' worth of history -->
      <maxHistory>30</maxHistory>
    </rollingPolicy>
  </appender>
  
  <!-- 按文件大小区分的滚动日志 -->
  <appender name="INFO-OUT" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <file>logs/info.log</file>
      
    <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss} [%class:%line] - %m%n</pattern>
    </encoder>
    
      <filter class="ch.qos.logback.classic.filter.LevelFilter">
      <level>INFO</level>
      <onMatch>ACCEPT</onMatch>
      <onMismatch>DENY</onMismatch>
    </filter>
      
      <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
      <fileNamePattern>info.%i.log</fileNamePattern>
      <minIndex>1</minIndex>
      <maxIndex>3</maxIndex>
    </rollingPolicy>
    
    <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
      <maxFileSize>5MB</maxFileSize>
    </triggeringPolicy>
    
  </appender>
  
  
  <!-- 按日期和大小区分的滚动日志 -->
  <appender name="DEBUG-OUT" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <file>logs/debug.log</file>

    <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss} [%class:%line] - %m%n</pattern>
    </encoder>
      
      <filter class="ch.qos.logback.classic.filter.LevelFilter">
      <level>DEBUG</level>
      <onMatch>ACCEPT</onMatch>
      <onMismatch>DENY</onMismatch>
    </filter>
      
      <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!-- rollover daily -->
      <fileNamePattern>debug-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
      
      <timeBasedFileNamingAndTriggeringPolicy
            class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
        <!-- or whenever the file size reaches 100MB -->
        <maxFileSize>100MB</maxFileSize>
      </timeBasedFileNamingAndTriggeringPolicy>
      
    </rollingPolicy>
    
  </appender>
  
  
   <!-- 级别阀值过滤 -->
  <appender name="SUM-OUT" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <file>logs/sum.log</file>

    <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss} [%class:%line] - %m%n</pattern>
    </encoder>
      
    <!-- deny all events with a level below INFO, that is TRACE and DEBUG -->
    <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
      <level>INFO</level>
    </filter>

      
      <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!-- rollover daily -->
      <fileNamePattern>debug-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
      
      <timeBasedFileNamingAndTriggeringPolicy
            class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
        <!-- or whenever the file size reaches 100MB -->
        <maxFileSize>100MB</maxFileSize>
      </timeBasedFileNamingAndTriggeringPolicy>
      
    </rollingPolicy>
    
  </appender>
  
  
  <root level="debug">
    <appender-ref ref="STDOUT" />
    <appender-ref ref="ERROR-OUT" />
    <appender-ref ref="INFO-OUT" />
    <appender-ref ref="DEBUG-OUT" />
    <appender-ref ref="SUM-OUT" />
  </root>
</configuration>






<?xml version="1.0" encoding="UTF-8"?>

<configuration debug="true" scan="false" scanPeriod="30 seconds"> 

  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender"> 
    <!-- encoders are  by default assigned the type
         ch.qos.logback.classic.encoder.PatternLayoutEncoder -->
    <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss} [%level] - %m%n</pattern>
    </encoder>
  </appender>
  
  <!-- 按日期区分的滚动日志 -->
  <appender name="APP" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <file>logs/app.log</file>
      
    <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss} [%class:%line] - %m%n</pattern>
    </encoder>
      
      <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
      <level>INFO</level>
    </filter>
      
      <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!-- daily rollover -->
      <fileNamePattern>app.%d{yyyy-MM-dd}.log</fileNamePattern>

      <!-- keep 30 days' worth of history -->
      <maxHistory>30</maxHistory>
    </rollingPolicy>
  </appender>
  
  
  <!-- 按日期和大小区分的滚动日志 -->
  <appender name="DEBUG-OUT" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <file>logs/debug.log</file>

    <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss} [%class:%line] - %m%n</pattern>
    </encoder>
      
      <filter class="ch.qos.logback.classic.filter.LevelFilter">
      <level>DEBUG</level>
      <onMatch>ACCEPT</onMatch>
      <onMismatch>DENY</onMismatch>
    </filter>
      
      <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
      <!-- rollover daily -->
      <fileNamePattern>debug-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
      
      <timeBasedFileNamingAndTriggeringPolicy
            class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
        <!-- or whenever the file size reaches 100MB -->
        <maxFileSize>30MB</maxFileSize>
        
      </timeBasedFileNamingAndTriggeringPolicy>
    </rollingPolicy>
    
  </appender>
  
  
  <root level="debug">
    <appender-ref ref="STDOUT" />
    <appender-ref ref="APP" />
    <appender-ref ref="DEBUG-OUT" />
  </root>
</configuration>



public class Slf4jTest {

    private static Logger Log = LoggerFactory.getLogger(Slf4jTest.class);
    
    @Test
    public void testLogBack(){
        
        Log.debug("Test the MessageFormat for {} to {} endTo {}", 1,2,3);
        Log.info("Test the MessageFormat for {} to {} endTo {}", 1,2,3);
        Log.error("Test the MessageFormat for {} to {} endTo {}", 1,2,3);
        
        try{
            throw new IllegalStateException("try to throw an Exception");
        }catch(Exception e){
            Log.error(e.getMessage(),e);
        }
    }
    
}
