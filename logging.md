---
title: Logger Configuration
nav_order: 3
---
# Logger Configuration
CatraProto uses [Serilog](https://serilog.net) for logging. You can use the helper method `Logger.CreateDefaultLogger()` to create a default logger which prints to console. **If no parameter is provided, the method will create a logger with logging level set to `LogEventLevel.Information`**.

You can provide a `LoggingLevelSwitch` to `Logger.CreateDefaultLogger()` if you want to change logging level.\
You can also specify a console theme using the `templateTheme` parameter.

Example:
```cs
var logger = Logger.CreateDefaultLogger(new LoggingLevelSwitch(LogEventLevel.Verbose), TemplateTheme.Literate);
```

**You can also create a logger with your own settings** by following the instructions in Serilog's [documentation](https://github.com/serilog/serilog/wiki/Configuration-Basics).

## Using your own sink
CatraProto provides a `GetExpressionTemplate`. This method returns the `ExpressionTemplate` used by CatraProto for logging to help keep the logging template consistent.

Example:

```cs
var myOwnLogger = new LoggerConfiguration()
    .WriteTo
    .File(Logger.GetExpressionTemplate(), "Log.txt", LogEventLevel.Verbose)
    .CreateLogger();
```
