# Debugging UE4 HTML5

while most browsers have a pretty nice debugger for "web page" development -- it is essentially un-usable when building UE4 for the web browser.

- before, builds compiled to "javascript assembly" (ASMjs) -- and, there was no way to put in breakpoints when the JS was smashed into one line
	- yes, some have a code beautifier button -- but, the debuggers would choke when trying to keep the (back then) giant 100+MB of code in the scroll window
- and now, with "web assembly" (WASM) -- there doesn't seem to be any UE4 code visible (just the JS plumbing code for the WASM driver)

so, I mostly just print stuff to the console window for debugging.

> TIP: build for your Desktop platform and debug as much as you can there.
it will be so much easier than trying to do the same thing for HTML5.


* * *
## How To Dump The Stack And Print From Cpp

at the top of any UE4 cpp file -- put the following at the **end** of the `#includes`:
```cpp
#ifdef __EMSCRIPTEN__
#include <emscripten/emscripten.h>
#include <cwchar>
#else
#define EM_ASM(x)
#define EM_ASM_(...)
#define EM_ASM_ARGS(...)
#endif
```

and use the `EM_ASM*` where ever you see fit.  the `#else` allows you to just put the `EM_ASM*` calls in anywhere
without having to `#ifdef __EMSCRIPTEN__` the `EM_ASM*` calls (e.g. just in case you rebuild the editor).


so for example, in `Engine/Source/Runtime/Core/Private/Misc/AssertionMacros.cpp`,
put the	`#include` and`#define` stuff (*EM_ASM code*) up near the top of the
file -- and then, in the following function:

```cpp
void FDebug::LogAssertFailedMessageImpl(const ANSICHAR* Expr, const ANSICHAR* File, int32 Line, const TCHAR* Fmt, ...)
{
	EM_ASM({ console.log( jsStackTrace() ); });
	. . .
```

here, I added a call to the `jsStackTrace()` function to dump the call stack when ever UE4 crashes.


I sometimes put `jsStackTrace()` when ever I need to figure what's going on.

* * *
### My `EM_ASM*` snippets:

`EM_ASM({ console.log("NICKNICK: msg here"); });`
- no variables -- just output to console.log

`EM_ASM_({ console.log("NICKNICK: msg here:" + $0); }, bSucceeded);`
- simple variables

`EM_ASM_ARGS({ var str = UTF8ToString($0,$1); console.log("NICKNICK: msg here [" + str + "]"); }, TCHAR_TO_ANSI(*some_FString), some_FString.len());`
- string variables -- most of UE4's strings are the `FString` object.
these needs to be wrapped in the `TCHAR_TO_ANSI()` macro in order to make them printable in the console.log

these 3 `EM_ASM*` snippets above, are pretty much my most used ones -- including finding some goofy compilier bugs...

the following are not used as much, but you may encounter a need for them in UE4's code:

```
emscripten_log(EM_LOG_CONSOLE, TCHAR_TO_ANSI(some_TCHAR));
EM_ASM_ARGS({ var str = UTF8ToString($0,$1); console.log("NICKNICK: msg here [" + str + "]"); }, TCHAR_TO_ANSI(some_TCHAR), strlen(TCHAR_TO_ANSI(some_TCHAR)));
EM_ASM_ARGS({ var str = UTF16ToString($0,$1); console.log("NICKNICK: msg here [" + str + "]"); }, some_TCHAR, wcslen(some_TCHAR));
```
- using the TCHAR variables here

**Note**: UE4 has a logger (that prints to stdout -- which then gets printed to console.log)
- but, these `UE_LOG()` needs setup (typed via macros) and turned on (via configuration settings)
- so I just use the `EM_ASM*` functions to make this simple to **use** -- as well as simple to **find and remove**
- however, if you feel there is a valid reason for permanent logging in certain situation (that will be commited), please see:
	- https://wiki.unrealengine.com/Logs,_Printing_Messages_To_Yourself_During_Runtime
	- pay special attention to: **Log Category Macros** and **Config file**

* * *
## BugHunting GLSL

for graphic shaders, I would use a combination of **RenderDoc** and **Spector.js**.

### RenderDoc

RenderDoc has a nice way to view a mesh from the vertex shader before and after calculations.

i'm going to just defer to the following video on how to configure chrome and use renderdoc:
- https://www.youtube.com/watch?v=T7MPxvX5alA
	- here's a shortcut/batch file command i use with this:
		- `"C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --disable-gpu-sandbox --disable-gpu-watchdog --gpu-startup-dialog --user-data-dir=zzz --no-first-run http://localhost:8000`

sometimes, i have to look through (in the **Mesh Output** tab) the raw data via the
(right click anywhere in the **VS Input** window) "Export to Bytes" option to see/ensure what the data is and expected.

the only problem with RenderDoc (in the context of webgl) -- is that the VertexShader disassembly is only available
for the desktop's GPU driver (i.e. ms directx, amd gcn, etc.).  in other words, not in "WebGL shader" code.

### Spector.js

this brings us to the in-browser extension called: [**Spector.js**](http://spector.babylonjs.com).

but first, we need to get UE4 to dump the WebGL shaders during cooks. follow the instructions here:
- https://www.unrealengine.com/en-US/blog/debugging-the-shader-compiling-process
	- basically, after cooks, you'll be able to find the generated glsl files at (**for example**):
		- `.../Project/Saved/ShaderDebugInfo/GLSL_ES_WEBGL/.../0/Engine/Private/MobileBasePassVertexShader.usf.glsl`
	- and the respective UE4 shader file used to build that, is located at:
		- `.../Project/Saved/ShaderDebugInfo/GLSL_ES_WEBGL/.../0/MobileBasePassVertexShader.usf`

	- again, you need to repackage in order to recook the assets -- which will then dump the GLSL code.

now, use Spector.js to capture a session, click through the frames we are interested in,
and (finally) view the shader used for that draw.

we can scan the dump of WebGL shaders with a snippet of the shader code (from Spector.js) to find
the `*.usf` file (which contains the sources and inlined file paths) to hunt down the broken shader.
- i've found scanning for the block inside of the main() curly braces (of the GLSL code from Spector.js)
is all you'll need to find the respective `*.usf` file.
- here's the perl script (scan.pl) i use to do this:

```perl
#!/usr/bin/perl -w

use strict;
use warnings;

# the code we are interested in
my @data = ();
while(<DATA>) { # see BELOW for the the DATA block
	s/[\r|\n|\s]//g; # strip all whitespace and line ending chars
	push @data, qq($_);
}

# the <stdin> file we are scanning against...
my $index = 0;
my $limit = @data;
while (<>) {
	s/[\r|\n|\s]//g; # strip all whitespace and line ending chars
	$index++ if ( qq($data[$index]) eq qq($_));

	if ( $index >= $limit ) {
		print "FOUND";
		last;
	}
}

# the lines inside of the main() curly braces (of the GLSL code from Spector.js)
# (please note, this is a sample and has been truncated for this doc)
__DATA__
    v0.xyz = vu_h[10].xyz;
    vec4 v3;
    v5.xyz = clamp((cross(clamp((cross(in_ATTRIBUTE2.xyz, in_ATTRIBUTE1)*in_ATTRIBUTE2.www), vec3(0.000000e+00, 0.000000e+00, 0.000000e+00), vec3(1.000000e+00, 1.000000e+00, 1.000000e+00)), in_ATTRIBUTE2.xyz)*in_ATTRIBUTE2.www), vec3(0.000000e+00, 0.000000e+00, 0.000000e+00), vec3(1.000000e+00, 1.000000e+00, 1.000000e+00));
    vec3 v6;
    vec3 v7;
    v11.xyz = ((in_ATTRIBUTE9.zzz*vu_h[3].xyz)+((in_ATTRIBUTE9.yyy*vu_h[2].xyz)+(in_ATTRIBUTE9.xxx*vu_h[1].xyz)));
```

- and that script is called via this shell script (scan.sh):

```bash
#!/bin/sh
find /work/ue4-4.24.3-html5/ShooterGame/Saved/ShaderDebugInfo/GLSL_ES2_WEBGL -name "*.glsl" -print | while read i; do
	results=`cat "$i" | ./scan.pl`
	if [ "$results" == "FOUND" ]; then
		echo $i
	fi
done
```

- finally, to kick this all off, just run: `./scan.sh`
	 - i'm sure these perl and shell scripts could have been combined and written a little more optimally
	 (instead of having a script call another script) -- but, for me -- this was all **throw-away code**,
	 quick and dirty utilities...

* * *

Next, [UE4 HTML5 F.A.Q](README.4.faq.UE4.HTML5.md)

* * *

