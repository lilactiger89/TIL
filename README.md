# TIL

Inspired by the likes of https://github.com/jbranchaud/til

---

###Categories
* [COM Interop / Platform Invoke](#com-interop--platform-invoke)
* [Dotfuscator](#dotfuscator)
* [Git](#git)
* [Git-TFS] (#git-tfs)
* [Log4net](#log4net)
* [Markdown](#markdown)
* [SpecFlow](#specflow)
* [Visual Studio Shortcuts](#visual-studio-shortcuts)
* [Visual Studio Team Services](#visual-studio-team-services)
* [WiX](#wix)
* [Xamarin](Xamarin.md)
 
---

###COM Interop / Platform Invoke
#####Creating COM classes with .NET
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

#####Using Platform Invoke to directly access a dll
In order to access an unmanaged class directly without registering it in COM you can add it as an isolated reference. If however this doesn't work out (for example if an alternative version with the same signature is registered in COM and referenced by the solution) then you can use Platform Invoke to directly import it.

```C#
    public class Externals
    {
        [DllImport("Sara.dll", CharSet = CharSet.Unicode, ExactSpelling = true, PreserveSig = false)]
        [return: MarshalAs(UnmanagedType.Interface)]
        public static extern object DllGetClassObject([In, MarshalAs(UnmanagedType.LPStruct)]Guid rclsid, [In, MarshalAs(UnmanagedType.LPStruct)]Guid riid);

        public const string IClassFactoryGuid = "00000001-0000-0000-C000-000000000046";
    }
    [ComImport, ComVisible(false), InterfaceType(ComInterfaceType.InterfaceIsIUnknown), Guid(Externals.IClassFactoryGuid)]
    public interface IClassFactory
    {
        void CreateInstance([MarshalAs(UnmanagedType.IUnknown)] object pUnkOuter, [MarshalAs(UnmanagedType.LPStruct)] Guid riid, out IntPtr ppvObject);
        void LockServer([MarshalAs(UnmanagedType.Bool)] bool fLock);
    }
```

DllGetClassObject returns a Class Factory which can be used to create the object
```C#
        private static T GetObjectFromSaraDll<T>(Guid classGuid, Guid interfaceGuid) where T : class
        {
            var dllGetClassObject = Externals.DllGetClassObject(classGuid, new Guid(Externals.IClassFactoryGuid));
            var factory = dllGetClassObject as IClassFactory;

            IntPtr objectPtr;
            factory.CreateInstance(null, interfaceGuid, out objectPtr);

            var objectForIUnknown = Marshal.GetObjectForIUnknown(objectPtr);

            return objectForIUnknown as T;
        }
```

###Dotfuscator
Features:
* Control Flow
* Linking - Merging multiple assemblies into one
* Pruning - Removing code deemed "unused"
* Renaming - Changing the names of methods and objects
* String encryption - Specify certain files to encrypt strings in to stop hackers locating critical code by the inline strings
* Maps - stores a map of what the names were vs what they are now - this can be used to ensure consistent renaming or to allow public methods to be used between assemblies
* Watermarking

Config file can just contain fragments or config and be used with the command line - it doesn't have to contain all of the configuration

###Git
#####Merging Repositories and keeping history
To merge one repository into another and retain history is quite simple:
 1. Go to the branch of the repository you're keeping
 2. Add the repository that you want to bring in as a remote
 3. Pull from the repository that you're bringing in
 4. Resolve any conflicts
 5. Remove the remote
 6. Commit
 7. Done!

```Shell
cd path/to/project-b
git remote add project-a path/to/project-a
git fetch project-a
git merge project-a/master # or whichever branch you want to merge
git remote remove project-a
```

#####Undoing changes to a subset of files in a commit when Origin has been pushed
 1. Soft reset to the commit prior to the mistake commit that you are undoing
 2. Unstage the changes that you don't want to commit
 3. Commit the modified staging locally
 4. Create a local branch at this point (I called mine holdit)
 5. Hard reset the master local branch back to the mistake commit
 6. Reverse the mistake commit locally
 7. Merge back in the holdit branch 
 8. Delete the holdit branch
 9. Recommit to origin

#####I want to change branches but I don't want to commit my changes to this branch!
[Git stash](https://git-scm.com/docs/git-stash) is a wonderful thing. It allows you to make something similar to a temporary commit which you can then *pop* when you're on the branch you want to commit on
```Shell
git stash
git checkout other-branch
git stash pop
```

###Git-TFS
This is a tool that will transform a TFS repository into a Git repository
* `c:\GittedTfs>git tfs clone https://mytfslocation.com "$/Git-TFS tester/First group of things/WindowsFormsApplication1" --branches=all` creates a Git repository from the given project and all of its branches. History is retained. **NOTE** TFS puts branches in different folders
* `c:\GittedTfs>git tfs list-remote-branches https://mytfslocation.com` lists all of the branches in a repository. The trunks/masters have an asterix next to them. Clone one of these to get both the trunk and all of it's branches.

You can't ignore a branch - do the whole thing and then delete the branch on your local repository.


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

###Markdown
####Headings
Add dashes under a line to get

    Page headings
    ---

Add hashes before a line to get

\#\#\#Section Headings

####Formatting
**\*\*Bold\*\***  or  __\_\_Bold\_\___

*\*Italics\**  or  _\_Italics\__

```
* Bullet 1
* Bullet 2
    * Sub-bullet 1
    * Sub-bullet 2
```

* Bullet 1
* Bullet 2
    * Sub-bullet 1
    * Sub-bullet 2

```
1. This is a numbered point
    * Bullet within a point
    * Another bullet
2. And here is another one
3. Three numbered points?
```

1. This is a numbered point
    * Bullet within a point
    * Another bullet
2. And here is another one
3. Three numbered points?

####Links
`[Display text for link](http://www.google.co.uk)` gives [Display text for link](http://www.google.co.uk)

To create a link without changing the display text just surround it in angular braces - `<http://www.google.co.uk>` gives <http://www.google.co.uk>. **Beware**: You need to include the http for this method to work


####Codeblocks
```
    Codeblocks can be created by indenting text with 4 spaces
```
`````
\```
Or by using backticks above and below
\```
`````
Inline `code` can be made by using single `` `backticks` `` 

####Tables
Tables are created by using pipes and dashes.

```
Heading 1    | Heading 2    | Heading 3
------------ | ------------ | ------------
Row 1 Cell 1 | Row 1 Cell 2 | Row 1 Cell 3
Row 2 Cell 1 | Row 2 Cell 2 | Row 2 Cell 3
Row 3 Cell 1 | Row 3 Cell 2 | Row 3 Cell 3
Row 4 Cell 1 | Row 4 Cell 2 | Row 4 Cell 3
```

Heading 1    | Heading 2    | Heading 3
------------ | ------------ | ------------
Row 1 Cell 1 | Row 1 Cell 2 | Row 1 Cell 3
Row 2 Cell 1 | Row 2 Cell 2 | Row 2 Cell 3
Row 3 Cell 1 | Row 3 Cell 2 | Row 3 Cell 3
Row 4 Cell 1 | Row 4 Cell 2 | Row 4 Cell 3

By default cells are left aligned. You can use colons to change the alignment
```
Animal    | Number of legs   | Hairy? | Name of young 
----------|:-----------------|-------:|:------------: 
Dog       | Four             | Yes    | Puppy         
Kangaroo  | Two              | Yes    | Joey          
Centipede | 30 - 354         | No     | Unknown       
```

Animal    | Number of legs   | Hairy? | Name of young 
----------|:-----------------|-------:|:------------: 
Dog       | Four             | Yes    | Puppy         
Kangaroo  | Two              | Yes    | Joey          
Centipede | 30 - 354         | No     | Unknown       

If you don't want headings on the table then you can leave them blank. **Beware**: If you leave the headers blank then you will either have to put extra pipes in to denote the edges of the able or you will need to ensure that there are fewer than four spaces to the first pipe otherwise the compiler will create a code block
```
|          |                  |
|--------- | ---------------- |
|Thing 1   | Hat              |
|Thing 2   | Ball             |
|Thing 3   | Jacket           |

or

   |   
--------- | ---------------- 
Thing 1   | Hat              
Thing 2   | Ball             
Thing 3   | Jacket           

```

|          |                  |
|--------- | ---------------- |
|Thing 1   | Hat              |
|Thing 2   | Ball             |
|Thing 3   | Jacket           |

####Images
Image paths are relative to the file location `![alternative text](img/somepicture.png)`

###SpecFlow
In a feature file you can add tables to your steps - this allows you to easily specify multiple values and to logically group parameters

```
Scenario: Do not show messages in the unicorn category on the main page
  Given that I have a database containing the following messages
    | Id | Category    | Text                            |
    | 1  | Rainbows    | Rainbows have many colours      |
    | 2  | Unicorn     | Unicorns are secretly evil      |
    | 3  | Butterflies | Butterflies can fly to the moon |
  When I open the application
  Then The main page should display the following messages
    | Message                         |
    | Rainbows have many colours      |
    | Butterflies can fly to the moon |
```
The table is automatically reformated whenever you add a pipe so if it becomes misaligned you can just remove and readd a pipe to easily fix it


###Visual Studio Shortcuts
- Alt+Shift -> Block select
- Alt+Up/Alt+Down -> Move line up/down
- Ctrl+- -> Navigate backward
- Ctrl+Tab -> Switch tabs
- Ctrl+D -> Duplicate line
- Ctrl+E Ctrl+C -> Resharper code formatting
- Ctrl+K Ctrl+D -> Format document
- Ctrl+R r -> Rename method
- Ctrl+Alt+P -> Attach to process
- Ctrl+F12 -> Go to implementation
- Shift+F10 -> Right click
- Shift+F12 -> Resharper find usages
- Shift+Alt+L -> Scroll to active file in Solution Explorer
- Shift+Ctrl+Alt+S -> SpecFlow find step usage

###Visual Studio Team Services
Build versioning - [Colin's AlM Corner Build & Release Tools](https://marketplace.visualstudio.com/items?itemName=colinsalmcorner.colinsalmcorner-buildtasks) - this is an extension to VSTS that allows automatic custom versioning by the build process.

###WiX
#####Setting Registry edit permissions
This needs to be done at every level that the special permission is required. Although User="Everyone" is in english, WiX maps it to the corresponding SID which means that the permissions will be set correctly on non-English computers
```xml
<RegistryKey Root="HKLM" Key="Software\SaraSoft" ForceCreateOnInstall="yes">
  <Permission GenericAll="yes" User="Everyone"/>
  <RegistryKey Key="Properties" ForceCreateOnInstall="yes">
    <Permission GenericAll="yes" User="Everyone"/>
    <RegistryValue Name="Update server" Value="sara.com/updates" Type="string" />
    <RegistryValue Name="Licence key" Value="" Type="string" />
    <RegistryValue Name="Language" Value="Klingon" Type="string" />
  </RegistryKey>
</RegistryKey>
```
