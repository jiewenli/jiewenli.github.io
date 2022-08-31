# 动态日志级别


## 背景

&emsp;&emsp;随着业务的快速发展，业务复杂度不断增加，线上系统环境有任何细小波动，对整个业务都产生巨大的影响，甚至造成灾难性的雪崩效应，开发人员对此必须立即响应，快速解决问题。

&emsp;&emsp;如何提供问题排查效率呢?最有效的方法就是通过系统日志。但是如果保证系统日志全面就必须打印所有系统和业务日志。这就带来另一个问题，日志量暴涨，日志除了能帮助我们解决问题外，同时会造成系统性能下降，在这种背景下，日志级别动态调整组件可以在需要时实时调整线上日志输出级别。

## 实现


```java
@GetMapping(value = "/all-log-level")
public String getAllLogLevel() throws Exception{
        return LogLevelDynamicChangeService.getLoggerList();
}

@GetMapping(value = "/change-log-level")
public String changeLogLevel(@RequestParam(LogConstant.LOG_NAME_KEY) String name, @RequestParam(LogConstant.LOG_LEVEL_KEY) String level) {
    List<LoggerSetting> settings = new ArrayList<>();
    LoggerSetting loggerSetting = new LoggerSetting();
    loggerSetting.setName(name);
    loggerSetting.setLevel(level);
    settings.add(loggerSetting);
    final boolean ret = LogLevelDynamicChangeService.setLogLevel(settings);
    return ret ? level : "error";
}

```

```java
@Component
public class LogLevelDynamicChangeService { 
    
    private Map<String, Object> LoggerMap = new HashMap<>();
    private String logFrameworkType;

    /**
     * 1. 初始化：确定所使用的日志框架，获取配置文件中所有的 Logger 内存实例，
     * 并将它们的引用缓存到 Map 容器中
    */
    @PostConstruct
    public void init() {
        String type = StaticLoggerBinder.getSingleton().getLoggerFactoryClassStr();
        if (LogConstant.LOGBACK_LOGGER_FACTORY.equals(type)) {
            logFrameworkType = LogFrameworkType.LOGBACK;
            ch.qos.logback.classic.LoggerContext LoggerContext = (ch.qos.logback.classic.LoggerContext) LoggerFactory.getILoggerFactory();
            for (ch.qos.logback.classic.Logger Logger : LoggerContext.getLoggerList()) {
                if (Logger.getLevel() != null) {
                    LoggerMap.put(Logger.getName(), Logger);
                }
            }
            ch.qos.logback.classic.Logger rootLogger = (ch.qos.logback.classic.Logger)LoggerFactory.getLogger(Logger.ROOT_LOGGER_NAME);
            LoggerMap.put(rootLogger.getName(), rootLogger);
        } 
        // else if (LogConstant.LOG4J_LOGGER_FACTORY.equals(type)) {
        //     logFrameworkType = LogFrameworkType.LOG4J;
        //     Enumeration enumeration = org.apache.log4j.LogManager.getCurrentLoggers();
        //     while (enumeration.hasMoreElements()) {
        //         Logger Logger = (org.apache.log4j.Logger)enumeration.nextElement();
        //         if (Logger.getLevel() != null) {
        //             LoggerMap.put(Logger.getName(), Logger);
        //         }
        //     }
        //     org.apache.log4j.Logger rootLogger = org.apache.log4j.LogManager.getRootLogger();
        //     LoggerMap.put(rootLogger.getName(), rootLogger);
        // } else if (LogConstant.LOG4J2_LOGGER_FACTORY.equals(type)) {
        //     logFrameworkType = LogFrameworkType.LOG4J2;
        //     org.apache.logging.log4j.core.LoggerContext LoggerContext = (org.apache.logging.log4j.core.LoggerContext) org.apache.logging.log4j.LogManager.getContext(false);
        
        //     Map<String, org.apache.logging.log4j.core.config.LoggerConfig> map = LoggerContext.getConfiguration().getLoggers();
        //     for (org.apache.logging.log4j.core.config.LoggerConfig LoggerConfig : map.values()) {
        //         String key = LoggerConfig.getName();
        //         if (StringUtils.isEmpty(key)) {
        //             key = "root";
        //         }
        //         LoggerMap.put(key, LoggerConfig);
        //     }
        // }
         else {
            logFrameworkType = LogFrameworkType.UNKNOWN;
            log.error("Log 框架无法识别 : type={}", type);
        }
    }

    /**
     * 2. 获取 Logger 列表：从本地 Map 容器取出
     * @throws JSONException
     */
    public String getLoggerList() throws Exception {
        JSONObject result = new JSONObject();
        result.put(LogConstant.LOG_FRAMEWORK_TYPE_KEY, logFrameworkType);
        JSONArray LoggerList = new JSONArray();
        for (ConcurrentMap.Entry<String, Object> entry : LoggerMap.entrySet()) {
            JSONObject LoggerJSON = new JSONObject();
            LoggerJSON.put(LogConstant.LOG_NAME_KEY, entry.getKey());
            if (logFrameworkType == LogFrameworkType.LOGBACK) {
                ch.qos.logback.classic.Logger targetLogger = (ch.qos.logback.classic.Logger) entry.getValue();
                LoggerJSON.put(LogConstant.LOG_LEVEL_KEY, targetLogger.getLevel().toString());
            } 
            // else if (logFrameworkType == LogFrameworkType.LOG4J) {
            //     org.apache.log.Logger targetLogger = (org.apache.log4j.Logger) entry.getValue();
            //     LoggerJSON.put(LogConstant.LOG_LEVEL_NAME, targetLogger.getLevel().toString());
            // } else if (logFrameworkType == LogFrameworkType.LOG4J2) {
            //     org.apache.logging.log4j.core.config.LoggerConfig targetLogger = (org.apache.logging.log4j.core.config.LoggerConfig) entry.getValue();
            //     LoggerJSON.put(LogConstant.LOG_LEVEL_NAME, targetLogger.getLevel().toString());
            // } 
            else {
                LoggerJSON.put(LogConstant.LOG_LEVEL_KEY, "Logger 的类型未知 , 无法处理 !");
            }
            LoggerList.put(LoggerJSON);
        }
        result.put(LogConstant.LOG_LIST_KEY, LoggerList);
        log.info("getLoggerList: result={}", result.toString());
        return result.toString();
    }

    /**
     * 3. 修改 Logger 的级别
     * @throws JSONException
     */
    public boolean setLogLevel(List<LoggerSetting> settings) {
        log.info("setLogLevel: data={}", settings);
        List<LoggerSetting> LoggerList = settings;
        if (CollectionUtils.isEmpty(LoggerList)) {
            return false;
        }
        for (LoggerSetting Loggerbean : LoggerList) {
            Object Logger = LoggerMap.get(Loggerbean.getName());
            if (Logger == null) {
                throw new RuntimeException("需要修改日志级别的 Logger 不存在 ");
            }
            if (logFrameworkType == LogFrameworkType.LOGBACK) {
                ch.qos.logback.classic.Logger targetLogger = (ch.qos.logback.classic.Logger) Logger;
                ch.qos.logback.classic.Level targetLevel = ch.qos.logback.classic.Level.toLevel(Loggerbean.getLevel());
                targetLogger.setLevel(targetLevel);
            }
            // if (logFrameworkType == LogFrameworkType.LOG4J) {
            //     org.apache.log4j.Logger targetLogger = (org.apache.log4j.Logger) Logger;
            //     org.apache.log4j.Level targetLevel = org.apache.log4j.Level.toLevel(Loggerbean.getLevel());
            //     targetLogger.setLevel(targetLevel);
            // } else if (logFrameworkType == LogFrameworkType.LOGBACK) {
            //     ch.qos.logback.classic.Logger targetLogger = (ch.qos.logback.classic.Logger) Logger;
            //     ch.qos.logback.classic.Level targetLevel = ch.qos.logback.classic.Level.toLevel(Loggerbean.getLevel());
            //     targetLogger.setLevel(targetLevel);
            // } else if (logFrameworkType == LogFrameworkType.LOG4J2) {
            //     org.apache.logging.log4j.core.config.LoggerConfig LoggerConfig = (org.apache.logging.log4j.core.config.LoggerConfig) Logger;
            //     org.apache.logging.log4j.Level targetLevel = org.apache.logging.log4j.Level.toLevel(Loggerbean.getLevel());
            //     LoggerConfig.setLevel(targetLevel);
            //     org.apache.logging.log4j.core.LoggerContext ctx = (org.apache.logging.log4j.core.LoggerContext) org.apache.logging.log4j.LogManager.getContext(false);
            //     ctx.updateLoggers(); // This causes all Loggers to refetch information from their LoggerConfig.
            // } 
            
            else {
                throw new RuntimeException("Logger 的类型未知 , 无法处理 !");
            }
        }
        return true;
    }

    private List<LoggerSetting> parseJsonData(JSONArray data) throws JSONException {

        List<LoggerSetting> LoggerBeans = new ArrayList<>();
        for (int i = 0; i < data.length(); i++) {
            JSONObject jsonObject = data.getJSONObject(i);
            LoggerSetting LoggerBean = new LoggerSetting();
            LoggerBean.setName(jsonObject.getString(LogConstant.LOG_NAME_KEY));
            LoggerBean.setLevel(jsonObject.getString(LogConstant.LOG_LEVEL_KEY));
            LoggerBeans.add(LoggerBean);
        }
        return LoggerBeans;
    }
}

```



