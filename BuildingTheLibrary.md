

# Adding the Google APIs Objective-C Client Library to a Project #

The Google APIs Objective-C Client Library's core classes are provided as a built framework, suitable for inclusion in a Mac application bundle's Frameworks folder, and as a static library for iOS applications.

Alternatively, all of the library's sources may also be compiled directly into a Mac or an iOS application.

Service-specific classes, such as for Calendar and Drive, are provided in separate folders. They are not compiled into the library or framework with the core classes, so the needed service classes should should be dragged into the application project.

**Use one of the following three approaches** to add the library to your project. Do not attempt to build and link the framework target for iOS applications.

## Linking to the iOS Static Library ##

The library project includes a target for building a static library for iOS apps. The static library target should be dragged into your application project's Build Phases "Link Binary with Libraries" list.

The static library target also creates a folder with the library's headers to drag into your target's sources. The headers folder is created in the build products directory. To find build products directory, in Xcode 4's Locations preferences pane, click the arrow for Derived Data.

**_Tip:_** Use an Xcode 4 workspace for your application to keep library project sources available.

Next, add the [ObjC link option](http://developer.apple.com/library/mac/#qa/qa1490/_index.html) to the application target's build settings, along with the all\_load flag:

> Other Linker Flags: `-ObjC -all_load`

Also drag the folder with the class files for the services needed by your application, such as Calendar or Books, directly into your project. From the generated files folder, include in your project either the individual class .m files for the service **or** the service's `_Sources.m` file, but _not both_.

Application classes using the library should include the header for each specific service, such as

`#import "GTLCalendar.h"`

**_Note:_** To support the OAuth 2 sign-in classes in the library, your application will need to link to Security.framework and SystemConfiguration.framework, as well as include the OAuth 2 controller's `xib`.

## Linking to the Mac OS X Framework ##

To add the framework to an Xcode project, drag GTL.framework to the project's Linked Frameworks source group, then drag the GTL framework from the  Linked Frameworks group folder to the Link Binary With Library phase inside of the application target.

Drag the folder with the class files for the services needed by your application, such as Calendar or Books, directly into your project.

For both debug and release builds of your application, add this define to your project file:

-DGTL\_BUILT\_AS\_FRAMEWORK=1

Application classes using the library should include the header for each specific service, such as

`#import "GTLCalendar.h"`


To facilitate debugging, you may choose to include the GTL.xcodeproj project file either in an Xcode 4 workspace for your application or directly in your application project as a cross-project reference.  The example applications show how to include a reference to the GTL framework project file in an Xcode project.

**_Tip_**: if Xcode's debugger is ignoring breakpoints set in the framework, turn off the ["Load symbols lazily" option](http://developer.apple.com/mac/library/documentation/DeveloperTools/Conceptual/XcodeDebugging/150-Debugging_Preferences/debugging_preferences.html) in Xcode's Debugging preferences.

**_Note:_** To support the OAuth 2 sign-in classes in the library, your application will need to link to Security.framework and SystemConfiguration.framework, as well as include the OAuth 2 controller's `xib`.

## Compiling the Source Files Directly into a Mac or iOS Application ##

Rather than link to the GTL framework, you can compile the GTL library sources directly into your own project. To do this, find the library's `GTLCommon_Sources.m` and `GTLCommon_Networking.m` files, and drag the files into your project's window.

Then add the library's source folders to the Header Search Paths entry of your project's build settings: `Source`, `Source/Objects`, `Source/Utilities`, `Source/HTTPFetcher`, `Source/OAuth2`, `Source/OAuth2/Touch` (or `Source/OAuth2/Mac`).

Also add to your project the sources files for the services needed by your application, such as Calendar or Books. From the generated files folder, drag into your project the service's `_Sources.m` file, such as `GTLCalendar_Sources.m`. Also add the service's source folder, such as `Source/Services/Calendar/Generated`, to the Header Search Path entry of you project's build settings.

Application classes using the library should include the header for each specific service, such as

`#import "GTLCalendar.h"`

For just the Debug configuration of your project, add this compiler definition to ensure that the library's debug-only code is included:

> Other C Flags: `-DDEBUG=1`

Or, if the Other C Flags setting is not available in your target's build options, set the equivalent user-defined setting:

> `OTHER_CFLAGS=-DDEBUG=1`

**_Note:_** To support the OAuth 2 sign-in classes in the library, your application will need to link to Security.framework and SystemConfiguration.framework, as well as include the OAuth 2 controller's `xib`.


### ARC Compatibility ###

When the library source files are compiled directly into a project that has ARC enabled, then ARC must be disabled specifically for the library source files.

**To disable ARC for source files in Xcode 5**, select the project and the target in Xcode. Under the target "Build Phases" tab, expand the Compile Sources build phase, select the library source files in the left column (Name), then press Enter to open an edit field, and enter `-fno-objc-arc` into the right column (Compiler Flags).