# TIL

Inspired by the likes of https://github.com/jbranchaud/til

---

###Categories
* [COM Interop](#com-interop)
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

###Visual Studio Shortcuts
- Ctrl+D -> Duplicate line
- Ctrl+K Ctrl+D -> Format document
- Alt+Up, Alt+Down -> Move line up/down
- Alt+Shift -> Block select
