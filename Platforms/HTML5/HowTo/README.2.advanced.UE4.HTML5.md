# Advanced Example of Building a UE4 HTML5 Project

this page will show you how to:

- [obtain a sample UE4 game project](#download-shootergame-sample-project) via the Epic Games Launcher
- [configure multiplayer project to use websockets](#configuring-ue4-projects-with-websocket-networking)
- [play a multiplayer game session with desktop server, desktop client and HTML5 client](#test-html5-with-desktop-shootergame-server)


* * *
## Epic Games Launcher

normally, game developers (who do not have much or any programming experience)
can download the **Epic Games Launcher** to get the Unreal Engine Editor and to
start making games with it.

you can also download a number of sample projects from the "Learn" that's
available via the **Launcher**.

we will use one of Epic Games sample projects from the Launcher's **Learning**
center to continue with this advanced demo.


### Downloading the **Epic Games Launcher**

to obtain the **Launcher**, download it from:

- https://www.epicgames.com/store/en-US/download
	- Windows and Mac
- https://lutris.net/games/epic-games-store/
	- Linux (requires Wine)

after opening up the **Launcher**, you will be required to **Sign In** in order
to use it.  use **your** Epic Games account credentials you've used when linking
your GitHub account (we saw how to do this [HERE](../../../README.md)).


### Download ShooterGame sample project

to download the **ShooterGame** project:
- open **Epic Games Launcher** application
- goto the **Learn** tab
- scroll down to the **Games** section
- click on **ShooterGame** to view the project

![](Images/launcher_shootergame.png)

- click on Free
- click on Create Project
- pick the path to download the project to
- select the engine version: 4.24
- click on Create

![](Images/launcher_location_create.png)


* * *
## ShooterGame for Desktop

just like [Starting with UE4 Editor](README.0.building.UE4.Editor.md), we will
first build for the Desktop to ensure the project (still) "works" before trying
other platforms.
- it is much easier to debug or trying to diagnose any problems on the "Native Platform" first (i.e. the desktop)

ShooterGame is a fully fleshed out project that can host network games.

- usually, a server is used to host these game sessions
	- Unreal Engine make these and are known as "UE4 Dedicated Servers"
	- an example of this was done in the previous HowTo: [UE4 Multi-player Testing With HTML5](README.1.emscripten.UE4.HTML5.md#ue4-multi-player-testing-with-html5)

- another way to host game sessions is to have the running application open a
	listening port and perform the same functionality as a Dedicated Server
	- we will be doing an example of this here

finally, the HTML5 client will need to connect to a **listening host**.
> NOTE: UE4 HTML5 will NEVER be able to "host" a game (it can join them, but never host).

for the purpose of this demo, we are going to use the "desktop game client" to
open a listening port to host a session.
- the HTML5 client will connect to this to play a networked game


### Configuring UE4 Projects with WebSocket Networking

in the 
example which basically demonstrated networking gameplay working out-of-the-box.

it was also mentioned there that a clone of a special branch based on Release-4.24
was made with the HTML5 platform files already populated.  this includes **config.ini**
files.

in this HowTo, we will go through the steps to make UE4 networking projects work
on the HTML5 platform in detail.

normally, all desktop and most console platforms will be using what is known as
the Unreal Engine **IpNetwork** NetDriver for network communications.

- this needs to be change to use the **WebSocketNetwok NetDriver** ([1](#netdriverdefinitions-1))
- add the **WebSocketNetworking.WebSocketNetDriver** ([2](#add-websocketnetdriver-settings-2))
settings to the Engine
- and finally, **disable Packet Handler Components not supported with websocket** ([3](#disable-packethandlercomponents-3))
	- one such example is the **SteamAuthComponentModuleInterface**


we will now go over each of these (1-3) steps one by one.


#### NetDriverDefinitions (1)

in the following files:
- Engine/Config/BaseEngine.ini
- ShooterGame/Config/Windows/WindowsEngine.ini
- ShooterGame/Config/Mac/MacEngine.ini
- ShooterGame/Config/Linux/LinuxEngine.ini

**disable** all existing NetDriverDefinitions
- by placing a `;` (semicolon) at the start of each line
	- this is known as **commenting out** a line (which "disables" or "hides" them from being used)

in **BaseEngine.ini**, it would look like this:

```ini
;NetDriverDefinitions=(DefName="GameNetDriver",DriverClassName="/Script/OnlineSubsystemUtils.IpNetDriver",DriverClassNameFallback="/Script/OnlineSubsystemUtils.IpNetDriver")
;+NetDriverDefinitions=(DefName="DemoNetDriver",DriverClassName="/Script/Engine.DemoNetDriver",DriverClassNameFallback="/Script/Engine.DemoNetDriver")
```

next, **add** the WebSocket NetDriver - like this:

```ini
;NetDriverDefinitions=(DefName="GameNetDriver",DriverClassName="/Script/OnlineSubsystemUtils.IpNetDriver",DriverClassNameFallback="/Script/OnlineSubsystemUtils.IpNetDriver")
;+NetDriverDefinitions=(DefName="DemoNetDriver",DriverClassName="/Script/Engine.DemoNetDriver",DriverClassNameFallback="/Script/Engine.DemoNetDriver")
NetDriverDefinitions=(DefName="GameNetDriver",DriverClassName="/Script/WebSocketNetworking.WebSocketNetDriver",DriverClassNameFallback="/Script/WebSocketNetworking.WebSocketNetDriver")
```

**remember** to also do this in WindowsEngine.ini, MacEngine.ini or LinuxEngine.ini
(depending on the system you are using - or do it on all of them if you are not sure).

> note: **BaseEngine.ini** has been done for you (if you're using the special branch)

> repeat: **ShooterGame/Config/{Windows,Mac,Linux}/{Windows,Mac,Linux}Engine.ini**
will **NEED TO BE EDITED** with these changes mentioned above.


#### Add WebSocketNetDriver Settings (2)

at the end of **BaseEngine.ini** (and only this file), add the
**WebSocketNetworking.WebSocketNetDriver** settings at the end of the file:

```ini
[/Script/WebSocketNetworking.WebSocketNetDriver]
AllowPeerConnections=False
AllowPeerVoice=False
ConnectionTimeout=60.0
InitialConnectTimeout=120.0
RecentlyDisconnectedTrackingTime=180
AckTimeout=10.0
KeepAliveTime=20.2
MaxClientRate=15000
MaxInternetClientRate=10000
RelevantTimeout=5.0
SpawnPrioritySeconds=1.0
ServerTravelPause=4.0
NetServerMaxTickRate=30
LanServerMaxTickRate=35
WebSocketPort=8889
NetConnectionClassName="/Script/WebSocketNetworking.WebSocketConnection"
MaxPortCountToTry=512
```

> again, this has been done for you if you're using the special branch


#### Disable PacketHandlerComponents (3)

in the following files:
- Engine/Config/BaseEngine.ini
- ShooterGame/Config/Windows/WindowsEngine.ini
- ShooterGame/Config/Mac/MacEngine.ini
- ShooterGame/Config/Linux/LinuxEngine.ini

find any **PacketHandlerComponents** sections, and disabling them by commenting them out.

for example, in **ShooterGame/Config/Windows/WindowsEngine.ini** -- this should look like this:

```ini
;[PacketHandlerComponents]
;+Components=OnlineSubsystemSteam.SteamAuthComponentModuleInterface
```

> note the `;` (semicolon) at the start of the line indicating this line is disabled

> UPDATE: as of this writting, this is **not** seen in BaseEngine.ini -- but,
it is mentioned here just in case you see this in the future...


* * *
### Package ShooterGame for Desktop

using the same steps outlined in [Build Sample Project](README.0.building.UE4.Editor.md#build-sample-project)

set the build type:
- Menu bar -> File -> **Package Project** -> Build Configuration
	- select `Development` (i.e. keep the build type the same for all platforms)

finally, package for your desktop
- Menu bar -> File -> Package Project -> **Windows/Mac/Linux** (etc.)
	- select the folder where the final files will be **archived** to


### Package ShooterGame for HTML5

using the same steps outlined in [Build Sample Project](README.0.building.UE4.Editor.md#build-sample-project)

set the build type:
- Menu bar -> File -> **Package Project** -> Build Configuration
	- select `Development` (i.e. keep the build type the same for all platforms)

finally, package for HTML4
- Menu bar -> File -> Package Project -> **HTML5**
	- select the folder where the final files will be **archived** to


* * *
### Test Desktop ShooterGame Client

here, we will be using the more advanced way to run the project from the command line.

to run multiple instances of the project at once (this networking project is a
perfect example), this time we will launch the game in `windowed` mode from the
command line.

- run the game in a windowed size screen (i.e. not full screen)
	- using `-windowed -resx=800 -resy=600` parameter to do this

- again, i like to see the `stdout` prints -- to know that the game is still running
	- using `-log` parameter to view this


#### on Windows via CommandPrompt

- open `CommandPrompt` to the location where files were **archived** to
	- e.g. `cd ...\BP_TP\WindowsNoEditor`
- run executable
	- `BP_TP.exe -windowed -resx=800 -resy=600 -log`

#### on Mac via Terminal

- open `terminal` to the location where files were **archived** to
	- e.g. `cd .../BP_TP/MacNoEditor`
- run executable
	- `open ./BP_TP.app --args -windowed -resx=800 -resy=600 -log`

#### on Linux via Terminal

- open `terminal` to the location where files were **archived** to
	- e.g. `cd .../BP_TP/LinuxNoEditor`
- run executable
	- `./BP_TP.sh -windowed -resx=800 -resy=600 -log`


> TIP: put the command in a shortcut (or alias, script, etc.) for future use!


### Another Desktop Client

for the purpose of this demo, open another client in windowed mode.
- in the first game window, select 

TODO: FINISH ME


* * *
### Test HTML5 with Desktop ShooterGame Server

LEFT OFF HERE

LEFT OFF HERE

LEFT OFF HERE

* * *
* * *

Next, [Debugging UE4 HTML5](README.3.debugging.UE4.HTML5.md)

* * *

