# F.A.Q. - Frequently Asked Questions

in this page, you can find information on:

- Browsers
	- [multi-threading](#multi-threading)
	- [integer overflow](#integer-overflow--script-error)
	- [infinite spinning "Launching engine..."](#infinite-spinning-launching-engine)
- Unreal Engine and HTML5
	- [already have UnrealEngine, how to get HTML5 platform extension](#i-already-have-unrealengine-how-do-i-get-the-html5-platform-extension)
	- [trying to get latest Unreal Engine working with HTML5 platform extensions](#trying-to-get-latest-unreal-engine-working-with-html5-platform-extensions)
- Troubleshooting UE4 HTML5 builds
	- [attempting to package for HTML5 instead brings up the help page](#attempting-to-package-for-html5-instead-brings-up-the-help-page)
	- [bit packing brokeness](#bit-packing-brokeness)
	- [common compiler issues when upgrading emscripten toolchain](#common-compiler-issues-when-upgrading-emscripten-toolchain)
- Emscripten
	- [links](#links)
	- [custom edits to emsripten for Unreal Engine](#custom-edits-to-emsripten-for-unreal-engine)
	- [troubleshooting emsdk issues](#troubleshooting-emsdk-issues)
- [Unreal Engine HTML5 Files](#unreal-engine-html5-files)


* * *
* * *
## Browsers

### multi-threading

- if you are having problems seeing the game run (**especially on the first time running this**)
	- e.g. on firefox - `LinkError: shared memory is disabled`
	- e.g. on chrome - `Assertion failed: requested a shared WebAssembly.Memory but the returned buffer is not a SharedArrayBuffer, indicating that while the browser has SharedArrayBuffer it does not have WebAssembly threads support - you may need to set a flag`

- you will need to configure your browser to allow "multi-threading" support
	- for Chrome: set `chrome://flags/#enable-webassembly-threads` as `WebAssembly threads support`
	- for Firefox: in `about:config` set `javascript.options.shared_memory` preference to `true` to enable **SharedArrayBuffer**

- then try reloading the project in your browser
- you may need to restart your browser if you are still having problems


### Integer Overflow / Script error

open the `console.log` window for your browser (`Control+Shift+I` as in "eye")

- if you see an error:
	- e.g. on firefox - `error: RuntimeError: integer overflow`
	- e.g. on chome - `Script error.`
- or (on developement builds) you do **NOT** see `LogHTML5Launch: Display: Starting UE4 ...`

these are usually indications of the browser not having enough resources to load page.

- try shutting down "other" applications
	- tests have shown that running to many apps in the background have lead to memory fragmentation,
		which browsers will have a hard time getting the memory space needed to run
	- shut down things like visual studio, the Editor, EVERYTHING -- except your
		browser and the "web server" -- this will all help you get the game running
		in the browser
	- then, reload the project page on the browser


if however, you see a lot of logs in the `console.log` window, then BUGHUNT the
game code (logic/functions).  please see [Debugging UE4 HTML5](README.3.debugging.UE4.HTML5.md)
for more details.


### Infinite Spinning "Launching engine..."

chances are, the game has crashed.  open the `console.log` window for your
browser (`Control+Shift+I` as in "eye").  and if you see a pile of red errors,
this should show you the crash stack.  begin the BUGHUNT!


* * *
* * *
## Unreal Engine and HTML5

* * *
### i already have UnrealEngine, how do i get the HTML5 platform extension

```bash
git fetch https://github.com/UnrealEngineHTML5/UnrealEngine 4.24.3-html5-1.39.18
git checkout 4.24.3-html5-1.39.18
```


* * *
### trying to get latest Unreal Engine working with HTML5 platform extensions

##### i.e. putting HTML5 platform files back in with latest Unreal Engine.

##### WARNING: this is not for the faint of heart.

> NOTE: Unreal Engine as of 4.24, is the last `ES2` supported rendering feature level

> this will cause headaches, as the HTML5 rendering code and shader compiler will need
	to be either changed to **only** support `ES3` (WebGL2) and above -- or pull in a
	lot of `ES2` code into the HTML5 platform extension folder (work is currently trying
	to impliment the latter solution so that more "older" OS are supported)


#### Engine/Platforms/HTML5/HTML5Setup.sh

rerun `Engine/Platforms/HTML5/HTML5Setup.sh` build script (see [HERE](README.1.emscripten.UE4.HTML5.md#html5setupsh-build-script)
for a refresher).  this will:
- re-setup the emscripten toolchain
- re-build all ThirdParty libraries HTML5 uses (that Epic sometimes edits)
- re-inject the HTML5 platform into UnrealBuildTool


#### re-generate Project/Make files

this should **ALWAYS** be done when ever new code is `pull` down (i.e. your local repository is `fetch` with latest & `merge` in):


##### on Windows
	GenerateProjectFiles.bat

##### on Mac
	GenerateProjectFiles.command

##### on Linux
	GenerateProjectFiles.sh


#### UE4Editor

with new Unreal Engine code now ready, re-build the Editor:
- [Compiling Support Programs](README.0.building.UE4.Editor.md#compiling-support-programs)
- [Compiling UE4Editor](README.0.building.UE4.Editor.md#compiling-ue4editor)


#### Packaging

make sure your existing development platform isn't broken. try to:
[Package a Sample BluePrint Project](README.0.building.UE4.Editor.md#package-a-sample-blueprint-project) first.

and then finally, try to: [package a UE4 sample project for HTML5](README.1.emscripten.UE4.HTML5.md#package-a-sample-blueprint-project-for-html5)


* * *
* * *
## Troubleshooting UE4 HTML5 builds

the following are tips and suggestions to try out when your HTML5 builds goes wrong.


* * *
### attempting to package for HTML5 instead brings up the help page

if you are seeing the [XXX Developing HTML5 project](https://docs.unrealengine.com/en-us/Platforms/HTML5/GettingStarted)
help page after clicking on **Menu Bar -> File -> Package Project -> HTML5**
- chances are that there is a mismatched `emscripten toolchain version` value(s)
	- this is normally seen when developers try to upgrade their emscripten toolchain and forget to update those values in `HTML5SDKInfo.cs`
	- please see [Updating UE4 C# scripts](README.1.emscripten.UE4.HTML5.md#updating-ue4-c-scripts)
		for more details on what to check for and how to fix this

> note: do not forget to recompile all [ThirdParty libraries](README.1.emscripten.UE4.HTML5.md#fetch-emsdk-and-build-thirdpary-libraries-for-html5) with your changed toolchain versions.


* * *
### bit packing brokeness

##### tobool63.i = icmp slt i176

the crux of this issue is best explained from this stackoverflow post:
- [C++ bitfield packing with bools](https://stackoverflow.com/questions/308364/c-bitfield-packing-with-bools)

in UE4, there are some mixed bitfield types that seems to explode the data structure size that emscripten doesn't like
- this may be fixed in "upstream" (i.e. clang10)
- but, UE4 can only be build with "fastcomp" (i.e. clang6)
	- this was mentioned in [Emscripten and UE4 README](README.1.emscripten.UE4.HTML5.md#upgrading-emscripten-toolchain) -- Upgrading Emscripten Toolchain section

to hunt down the offending `struct`, you need to disassemble the bitcode (`.bc`) file and scan for the error the build spits out during link.
- for example, during HTML5 packaging, you might see `error: tobool63.i = icmp slt i176` in the build log
- to get an idea of what function this might be coming from -- we scan the disassembled file (`.ll`) for that error
	- using `grep`, we dump 1000 lines before (`-B 1000`) and a few lines after (`-A 20`)
	- inspect that smaller chunk of disassembled lines to try and narrow the file we need to look for

in this example, `Engine/Source/Runtime/Engine/Classes/Engine/Scene.h` contained a mix of `uint8 :1` and `uint32 :1` data types.
emscripten ("fastcomp") didn't like this -- but, changing them all to bool fixed this link time error.

```bash
Engine/Binaries/HTML5$ llvm-dis-6.0 UE4Game.bc
Engine/Binaries/HTML5$ egrep -B 1000 -A 20 "tobool63.i = icmp slt i176" UE4Game.ll > minidump.txt
. . .
Engine/Source/Runtime/Engine/Classes/Engine$ mv Scene.h Scene.h.save
Engine/Source/Runtime/Engine/Classes/Engine$ cat Scene.h.save | perl -0p -e "s/uint\d+\s+(.+)\s?:\s?1;/bool \1;/g" > Scene.h
```

##### PLATFORM_USE_SHOWFLAGS_ALWAYS_BITFIELD

in `Engine/Source/Runtime/Engine/Public/ShowFlags.h`, this also had a similar bit packing problem.
but, all of the data types were the same here.  and yet, something was causing the game to render a black screen if the data type wasn't set as just `bool` for some reason.

it could have been due to using a `char` size value in `memset()` -- when the data type was originally `uint32 :1` -- but,
setting everything to `bool` here, caused the playstation to render its screen black... so this was `#if` guarded on platforms
in a case by case basis (yes, only HTML5 was the only one that needed this "fix").


* * *
### Common Compiler Issues When Upgrading Emscripten Toolchain

TODO: FINISH ME...


* * *
* * *
## Emscripten

to learn more about what powers Unreal Engine for the web browsers, please see:

### Links
- https://github.com/emscripten-core/emsdk
- https://github.com/emscripten-core/emscripten
- https://emscripten.org/docs/building_from_source/toolchain_what_is_needed.html


### Custom Edits to Emscripten for Unreal Engine

the detailed are found in the [patching emscripten](README.1.emscripten.UE4.HTML5.md#patching-emscripten) section.


* * *
### Troubleshooting emsdk issues

this is run during `HTML5Setup.sh` - if you see errors like the following:


##### python SSL/TSL

```bash
Warning: Possibly SSL/TLS issue. Update or install Python SSL root certificates...
```

update your system's [Python](https://www.python.org/downloads/) version.


##### "tools cannot be activated since it is not installed"

```bash
Warning: The SDK/tool 'node-X.Y.Z-64bit' cannot be activated since it is not installed! Skipping this tool...
Warning: The SDK/tool 'python-X.Y.Z.a-64bit' cannot be activated since it is not installed! Skipping this tool...
Warning: The SDK/tool 'java-X.Y-64bit' cannot be activated since it is not installed! Skipping this tool...
...
```

you might have missied the first error when running the `HTML5Setup.sh` script the first time.

in other words, running the `HTML5Setup.sh` script (the first time) shows that
`./emsdk install <version>` has failed for some reason.

and now, when trying to run `HTML5Setup.sh` again - it tried to proceed to
`./emsdk activate <version>` and it is saying that these tools are missing.
- to show the first error again (for bug reporting purposes):
	- DELETE you local copy of `Engine/Platforms/HTML5/Build/emsdk/*`
	- rerun `HTML5Setup.sh` again
	- carefully look at the build output and see if this error has been reported before
		- be sure to also see if this has been reported in https://github.com/emscripten-core/emscripten/issues


* * * 
* * * 
## Unreal Engine HTML5 Files


#### Custom HTML TemplatesFiles
you can change HTML and CSS template files to make them stick every re-packaing
- for all projects:
	- `.../Engine/Platforms/HTML5/Build/TemplateFiles/*`
- however, we recommend developers putting their custom template file changes in the project's own folder
	- copy `TempateFiles/*`
		- i.e. the same one "for all projects" (just above)
	- to `.../<Project>/Build/HTML5/<HERE>`


#### Build Folders

before the final copy to the **archive** folder, they are staged at:

- for **blueprint projects**
		- `.../Engine/Binaries/HTML5/`
- for **C++ projects** (e.g. project is named `CPP_TP`)
	- `.../CPP_TP/Saved/StagedBuilds/WindowsNoEditor/`
	- `.../CPP_TP/Saved/StagedBuilds/MacNoEditor/`
	- `.../CPP_TP/Saved/StagedBuilds/LinuxNoEditor/`

do not edit anything in those folders, they will be stompped on after every packaging.
	- i.e. treat this folder as READ ONLY


#### Engine/Intermediate/Build/HTML5/EmscriptenCache
- every time you upgrade/change your toolchain version (or switch between them or make a local modification
	to `.../emsdk/emscripten/<version>/system/...` you will need to delete the respective *.bc file or the
	whole folder to ensure your changes are recompiled


#### emscripten ports
- while a lot of the UE4 Thirdparty libs (used with HTML5 builds) are also available as escripten ports:
	- there are a number of custom UE4 changes that are required to make the engine functional for HTML5
	- for now, we recommend using the UE4 versions until this is revisited in the future
- NOTE: when using ports, remember to "turn off" UE4's version
	- e.g. if you wish to use emscripten's version of SDL2 -- you will need to disable UE4's version:
		- edit: **.../Engine/Thirdparty/SDL2/SDL2.Build.cs**
		- delete/comment out the HTML5 section and save the file
		- this change will be automatically picked up at packaging time


#### Engine/Platforms/HTML5/Build/BatchFiles/Build_All_HTML5_libs.rc
- this **_used_** to setup my shell environment when I need to
	_rebuild the thirdparty libraries_ for HTML5 (**from the ~OSX or Linux~ commandline**)
	- currently, I now use: `source .../emsdk/emsdk-<version>/emsdk_set_env.sh`
- **_now_**, this file has notes reminding me of the changes needed to the emscripten toolchain for UE4 purposes
	- ~please see the file if you're curious (especially the "**upgrading emsdk - REMEMBER TO DO THE FOLLOWING**" section)~
		- no longer needed - these are now [patched](README.1.emscripten.UE4.HTML5.md#patching-emscripten) in automatically


#### Engine/Platforms/HTML5/Build/BatchFiles/Build_All_HTML5_libs.sh
- after pulling down a new version of emscripten, we **have to rebuild** the (UE4's) thirdparty libs used with HTML5
	- i.e. **new toolchain --> new binaries**
- this shell script spells out which libs need to be rebuilt for HTML5
- scan the additional scripts called from this file for more details
	- every now and then, the Makefiles, CMake files, or something needs to change
		(new compiler warnings, depricated features, compiler/toolchain support level, etc.)
	- I like to build the thirdparty libs **one at a time** and even **one optimization level** at a time
	- YMMV, play with the (sh) scripts -- it's spelled out like a normal cookbook set of CLI instructions
- **this(these) script(s) DOES NOT NEED to know about UE4's build environment**
- ~that said, **it is recommended to build the thirdparty libs from OSX or Linux** (see the companion
	file **Build_All_HTML5_libs.rc** -- just mentioned above -- under the **WINDOWS NOTES** section for details on why)~
	- no longer needed - everything is now powered by CMake

- when these errors happen, you will need to hunt them down
	- all of the CMake files used here are found in `Engine/Platforms/HTML5/Build/BatchFiles/ThirdParty/...`
	- editing `Engine/Platforms/HTML5/Build/BatchFiles/Build_All_HTML5_libs.sh` and
		commenting out all libs except for the troubled one will be of help

