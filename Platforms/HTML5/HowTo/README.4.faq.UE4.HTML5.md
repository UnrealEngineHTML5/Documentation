# F.A.Q. - Frequently Asked Questions

in this page, you can find information on:

- Emscripten
	- [links](#links)
	- [custom edits to emsripten for Unreal Engine](#custom-edits-to-emsripten-for-unreal-engine)
- Troubleshooting UE4 HTML5 builds
	- [attempting to package for HTML5 instead brings up the help page](#attempting-to-package-for-html5-instead-brings-up-the-help-page)
	- [trying to get latest Unreal Engine working with HTML5 platform extensions](#trying-to-get-latest-unreal-engine-working-with-html5-platform-extensions)
	- [XXX bit packing brokeness]()
- Unreal Engine HTML5 Files

* * *
* * *
## Emsripten

to learn more about what powers Unreal Engine for the web browsers, please see:

### Links
- https://github.com/emscripten-core/emsdk
- https://github.com/emscripten-core/emscripten
- https://emscripten.org/docs/building_from_source/toolchain_what_is_needed.html


### custom edits to emscripten for Unreal Engine

TODO: FINISH ME...


* * *
* * *
## Troubleshooting UE4 HTML5 builds

the following are tips and suggestions to try out when your HTML5 builds goes wrong.


* * *
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
* * *
### i already have UnrealEngine, how do i get the HTML5 platform extension

```bash
get fetch https://github.com/UnrealEngineHTML5/UnrealEngine 4.24-html5
git checkout 4.24-html5
```


* * *
* * *
### trying to get latest Unreal Engine working with HTML5 platform extensions

##### i.e. putting HTML5 platform files back in with latest Unreal Engine.

##### WARNING: this is not for the faint of heart.

> NOTE: Unreal Engine as of 4.24, is the last `ES2` supported rendering feature level

> this means, HTML5 rendering code and shader compiler will need to be changed to **only** support `ES3` (WebGL2) and above


#### Engine/Platforms/HTML5/Setup.sh

rerun `Engine/Platforms/HTML5/Setup.sh` build script (see [HERE](README.1.emscripten.UE4.HTML5.md#the-html5-setupsh-build-script)
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
### tobool63.i = icmp slt i176

TODO FINISH ME...

```bash
Engine/Binaries/HTML5$ llvm-dis-6.0 UE4Game.bc
Engine/Binaries/HTML5/w$ egrep -B 1000 -A 20 "tobool63.i = icmp slt i176" ../UE4Game.ll > z1.txt
Engine/Source/Runtime/Engine/Classes/Engine$ cat Scene.h.save | perl -0p -e "s/uint\d+\s+(.+)\s?:\s?1;/bool \1;/g" > Scene.h
```


* * * 
* * * 
## Unreal Engine HTML5 Files


#### Custom HTML TemplatesFiles
you can change HTML and CSS template files to make them stick every re-packaing
- for all projects:
	- `.../Engine/Platforms/HTML5/Build/TemplateFiles/*`
- however, we recommend developers (i.e. game makers) putting their custom template file changes in the project's own folder
	- copy `TempateFiles/*` (above) to `.../<Project>/Build/HTML5/<HERE>`


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
- this **_used_** to setup my shell environment when I need to _rebuild the thirdparty libraries_ for HTML5 (**from the OSX or Linux commandline**)
	- currently, I now use `source .../emsdk/emsdk_set_env.sh`
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

