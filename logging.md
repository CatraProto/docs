---
title: Logger Configuration
nav_order: 3
---
# Logger Configuration
CatraProto uses [Serilog](https://serilog.net) for logging. You can use the helper method `Logger.CreateDefaultLogger()` to create a default logger which prints to console. **If no parameter is provided, the method will create a logger with logging level set to `LogEventLevel.Information`**.

**You can provide a `LoggingLevelSwitch` to `Logger.CreateDefaultLogger()` if you want to change logging level.**

Example:
```cs
var logger = Logger.CreateDefaultLogger(new LoggingLevelSwitch(LogEventLevel.Verbose));
```

**You can also create a logger with your own settings** by following the instructions in Serilog's [documentation](https://github.com/serilog/serilog/wiki/Configuration-Basics).
