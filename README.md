# TIL

Inspired by the likes of https://github.com/jbranchaud/til

---

###Categories
* [COM Interop](#com-interop)
* [Log4net](#log4net)
* [Visual Studio Shortcuts](#visual-studio-shortcuts)
 
---

###COM Interop
When creating a COM class in C# ensure that you stop .NET creating autogenerated interfaces by using the ClassInterface.None attribute
```C#
    [Guid("519BE8F6-6E28-47EC-96E5-9FF1D7FC1522")]
    [ClassInterface(ClassInterfaceType.None)]
    public class SaraClass : ISaraClass
    {
        public string GetGreeting()
        {
            return "Hello";
        }
    }
```

You'll also need to tell the interface which COM interface it should inherit from
```C#
    [Guid("846D44C9-230D-4D65-BF1E-0DD5CFD552D4")]
    [InterfaceType(ComInterfaceType.InterfaceIsIUnknown)]
    public interface ISaraClass
    {
        string GetGreeting();
    }
```

###Log4net
To setup log4net you need a logger and a configuration file. This example shows setting up a logger in a dll (not something you'd usually do, set it up in the exe and pass around the reference if you can) which is why the additional "find the configuration file it's by the dll" code is necessary.
```C#
using log4net;
using log4net.Config;

namespace SaraExamples
{
    public class SaraLogger
    {
        private readonly ILog logger;

        public SaraLogger()
        {
            logger = LogManager.GetLogger(typeof(SaraLogger));
            var assemblyLocation=GetType().Assembly.Location;
            XmlConfigurator.Configure(new FileInfo(Path.Combine(Path.GetDirectoryName(assemblyLocation),"loggingConfig.xml")));
        }
    }
}
```
Here are the entire contents of the log4net configuration file.
```XML
<log4net>
    <appender name="RollingFile" type="log4net.Appender.RollingFileAppender">
        <file value="C:\Logs\sara.log" />
        <appendToFile value="true" />
        <maximumFileSize value="100KB" />
        <maxSizeRollBackups value="2" />

        <layout type="log4net.Layout.PatternLayout">
            <conversionPattern value="%date [%thread] %-5level %logger - %message%newline" />
        </layout>
    </appender>
    
    <root>
        <level value="DEBUG" />
        <appender-ref ref="RollingFile" />
    </root>
</log4net>
```

###Visual Studio Shortcuts
- Ctrl+D -> Duplicate line
- Ctrl+K Ctrl+D -> Format document
- Ctrl+R r -> Rename method
- Alt+Up/Alt+Down -> Move line up/down
- Alt+Shift -> Block select
- Shift+F10 -> Right click
