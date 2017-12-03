# linux_xwindows
`linux_xwindows` is a plugin (well, arguably two plugins) for the Volatility Framework. It extracts various metadata from the window objects registered to X. It [placed joint fourth at the Volatility Plugin Contest 2017](https://volatility-labs.blogspot.co.uk/2017/11/results-from-5th-annual-2017-volatility.html).

# Motivation
At the time of writing, there are no plugins which perform any kind of analysis of the X Windows environment used by mainstream Linux distributions. `linux_xwindows` has been written to provide the forensic analyst with a full listing of every window from every display known to each X server.

The plugin works with the X server process and so is independent of the distribution or window manager - it requires there be only an X server process.

The plugin lists metadata about each window such as x and y co-ordinates, width and height dimensions, parent window objects and window ID.

During the development of the plugin it became apparent that it would be extremely beneficial to also interpret the properties of each window. The properties are stored as key-value pairs where the key is an atom - the atoms are stored on a per-X-server basis. Consequently, it became necessary to extract the atoms from the X server process as well.

The properties can store an extraordinary amount of data - this is discussed below.

It might be the case that an analyst is only interested in the atoms and so `linux_xatoms` exists independently of `linux_xwindows`.

# 2 plugins, 1 file

The source code ("the plugin"), `linux_xwindows.py`, provides both plugins: `linux_xwindows` and `linux_xatoms`.

# Invoking `linux_xwindows`
The plugin does not require any parameters, but the PID of the `X` or `Xorg` process(es) can be supplied for targeted analysis.

The plugin will identify the `X` or `Xorg` process by name (or from the `--pid` parameter).

If more than one X server process is found (or provided by `--pid`), each will be processed.

The user can also provide the `--atoms` parameter which will cause the plugin to output the atoms from the X process as well as the window data. This atoms data is the same output as produced by the `linux_xatoms` plugin.

## Expected Output from `linux_xwindows`
A string representation of each window object is output showing the offset of the object in memory and properties such as id, x, y, width, height, and parent. Below each window object will be each of the properties.

For each property, a string representation is output showing the offset of the object in memory and properties such as format, size, type and offset to data. It's name, type and possibly value are shown in human-readable format. Properties can be of many types, so a parser for each type must be implemented in order to make the value human-readable. The plugin implements lookups for the most common/useful types such as strings and integers.

Windows from all workspaces are output.

The output is modelled on the output of the linux `xwininfo` command.

The progress of the plugin is output to debug.info.

The output can be very verbose so users are encouraged to redirect the output to file, leaving only the progress (debug.info) visible on the console.

## Example
```
$ vol.py --plugins linux_windows -f LinuxMint-17.3-Mate-x64-61951b91.vmem --profile LinuxMint173x64 linux_xwindows --atoms --output-file=linux_xwindows.txt
Volatility Foundation Volatility Framework 2.6
Outputting to: linux_xwindows.txt
INFO    : volatility.debug    : Working with 'Xorg' (pid=6319).
INFO    : volatility.debug    : Seeking screenInfo. (This can take a while!)
INFO    : volatility.debug    : Anonymous section (BSS): rw- 0x7f3951117000 0x7f3951127000
INFO    : volatility.debug    : <ScreenInfo(offset=0x7f3951124e80, numScreens=1, x=0, y=0, width=1280, height=800)>
INFO    : volatility.debug    : Seeking atomRoot.
INFO    : volatility.debug    : Heap: rw- 0x7f39527eb000 0x7f3955b5a000
INFO    : volatility.debug    : <AtomNode(offset=0x7f39527ef150, a=0x1, string=PRIMARY)>
INFO    : volatility.debug    : Found 488 atom(s).
INFO    : volatility.debug    : Parsing the 1 ScreenPtr structure(s).
INFO    : volatility.debug    : <ScreenPtr(offset=0x7f39528224d0, myNum=0, id=0, x=0, y=0, width=1280, height=800, root=0x7f3952868d50)>
INFO    : volatility.debug    : Parsing the windows.
INFO    : volatility.debug    : Found 139 window(s).
INFO    : volatility.debug    : Working with 'Xorg' (pid=6819).
INFO    : volatility.debug    : Seeking screenInfo. (This can take a while!)
INFO    : volatility.debug    : Anonymous section (BSS): rw- 0x7f9f31bf2000 0x7f9f31c02000
INFO    : volatility.debug    : <ScreenInfo(offset=0x7f9f31bffe80, numScreens=1, x=0, y=0, width=800, height=600)>
INFO    : volatility.debug    : Seeking atomRoot.
INFO    : volatility.debug    : Heap: rw- 0x7f9f32071000 0x7f9f3404e000
INFO    : volatility.debug    : <AtomNode(offset=0x7f9f32075150, a=0x1, string=PRIMARY)>
INFO    : volatility.debug    : Found 485 atom(s).
INFO    : volatility.debug    : Parsing the 1 ScreenPtr structure(s).
INFO    : volatility.debug    : <ScreenPtr(offset=0x7f9f320a84d0, myNum=0, id=0, x=0, y=0, width=800, height=600, root=0x7f9f320eed50)>
INFO    : volatility.debug    : Parsing the windows.
INFO    : volatility.debug    : Found 148 window(s).
```
## Example output in 'linux_xwindows.txt'.
```
**********************************************************************
Working with 'Xorg' (pid=6319).
<AtomNode(offset=0x7f39527ef150, a=0x1, string=PRIMARY)>
<AtomNode(offset=0x7f39527ef180, a=0x2, string=SECONDARY)>
<AtomNode(offset=0x7f39527ef1b0, a=0x3, string=ARC)>
<AtomNode(offset=0x7f39527ef1e0, a=0x4, string=ATOM)>
<AtomNode(offset=0x7f39527ef210, a=0x5, string=BITMAP)>
<AtomNode(offset=0x7f39527ef240, a=0x6, string=CARDINAL)>
<AtomNode(offset=0x7f39527ef270, a=0x7, string=COLORMAP)>
<AtomNode(offset=0x7f39527ef2a0, a=0x8, string=CURSOR)>
...SNIP...
<AtomNode(offset=0x7f39545af620, a=0x1e7, string=image/bmp)>
<AtomNode(offset=0x7f39546ae320, a=0x1e8, string=sb_h_double_arrow)>
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
============================================================
<WindowPtr(offset=0x7f3952868d50, id=0x191, x=0, y=0, width=1280, height=800, parent=0x0, firstChild=0x7f3952ac4a00, lastChild=0x7f3952a9fde0, nextSib=0x0, prevSib=0x0)>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
_NET_ACTIVE_WINDOW(WINDOW): <Property(offset=0x7f3952b9ab00, propertyName=0x11a, type=0x21, format=32, size=0x1, data=0x7f39545b8cc0)>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
ESETROOT_PMAP_ID(PIXMAP): <Property(offset=0x7f3952bfcb10, propertyName=0x1d7, type=0x14, format=32, size=0x1, data=0x7f3952c771b0)>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
_XROOTPMAP_ID(PIXMAP): <Property(offset=0x7f3952b18550, propertyName=0x197, type=0x14, format=32, size=0x1, data=0x7f3952b65730)>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
XKLAVIER_STATE(INTEGER): <Property(offset=0x7f3952c6dd40, propertyName=0x1a7, type=0x13, format=32, size=0x2, data=0x7f3952c6dd70)>
0,1
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
CAJA_DESKTOP_WINDOW_ID(WINDOW): <Property(offset=0x7f3952bac9a0, propertyName=0x1ca, type=0x21, format=32, size=0x1, data=0x7f3952c58b40)>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
PULSE_COOKIE(STRING): <Property(offset=0x7f3952bb4210, propertyName=0x1a1, type=0x1f, format=8, size=0x200, data=0x7f3952bb4240)>
7ac04e3b1a8612b0eb72162a2dcb433d09548163bcb5038a8aca9a363388a5c6b81d4c52577a441ee5cc865ba34edecafad5e911cfc2c6666ee2e65d16000451aa7ed38af7da37ddb49183ec2d234fb2cafbb503f30c51007b6c4d11dbe617d4ec01edfcf4efaf376d41d72d4e56080e60fccb5378311b86f614ee5b188cd2d73c59806e45cd0db317fe64fed5dce7fe38c621e58390fa74c52dd587daf09b84a14ab0ff90de15cbb5c5d3a375081d2d7e9720c3ea3223cedadc7caf733f99bb4686e5e27a842f911a43f7c3d171511cf94be52f845504f1e8dbc3284bdcad0d43db44741d2fb8830df2a8470d7361740b6cfa55cccfcabeb56138f0d5d35594
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
PULSE_SERVER(STRING): <Property(offset=0x7f3952bb4190, propertyName=0x19e, type=0x1f, format=8, size=0x42, data=0x7f3952baf0d0)>
{641bf95da88a032691c184a258c3e848}unix:/run/user/1000/pulse/native
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
PULSE_SESSION_ID(STRING): <Property(offset=0x7f3952bb4140, propertyName=0x1c9, type=0x1f, format=8, size=0x2, data=0x7f3952bb4170)>
c1
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
PULSE_ID(STRING): <Property(offset=0x7f3952bb4080, propertyName=0x1c8, type=0x1f, format=8, size=0x2a, data=0x7f3952bb40b0)>
1000@641bf95da88a032691c184a258c3e848/2086
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
...SNIP...
```

## Notable Data in the Properties
Following are three examples of data revealed by parsing the properties of the window object. They can also be seen in 'linux_xwindows.txt' above

1) Firefox Window Title
```
WM_ICON_NAME(STRING): <Property(offset=0x7f9f3265f920, propertyName=0x25, type=0x1f, format=8, size=0x18, data=0x7f9f32e546f0)>
Google - Mozilla Firefox
```

2) LibreOffice Writer Title
```
WM_NAME(STRING): <Property(offset=0x7f39545793d0, propertyName=0x27, type=0x1f, format=8, size=0x25, data=0x7f39546adc40)>
Secret Plans.odt - LibreOffice Writer
```

3) "root" window defines colour schemes
```
RESOURCE_MANAGER(STRING): <Property(offset=0x7f3952a8ebc0, propertyName=0x17, type=0x1f, format=8, size=0x18c6, data=0x7f3952b1c1e0)>
*Box.background:	#d6d6d6
*Box.foreground:	#212121
*Button.activeBackground:	#ffffff
*Button.activeForeground:	#212121
*Button.background:	#d6d6d6
*Button.foreground:	#212121
*Button.highlightBackground:	#d6d6d6
*Button.highlightColor:	#212121
*Canvas.activeBackground:	#f7f7f7
*Canvas.activeForeground:	#212121
*Canvas.background:	#f7f7f7
*Canvas.foreground:	#212121
*Canvas.highlightBackground:	#f7f7f7
*Canvas.highlightColor:	#212121
*Canvas.selectbackground:	#9ab87c
*Canvas.selectforeground:	#f5f5f5
*Checkbu... <and 0x16c6 more bytes>
```

# Invoking `linux_xatoms`
The plugin does not require any parameters, but the PID of the `X` or `Xorg` process(es) can be supplied for targeted analysis.

The plugin will identify the `X` or `Xorg` process by name (or from the `--pid` parameter).

If more than one X server process is found (or provided by `--pid`), each will be processed.

## Expected Output from `linux_xatoms`
A string representation of each atom is output showing the offset of the atom in memory, the atom id, the atom name, and the atom type.

Again, the output can be very verbose so users are encouraged to redirect the output to file, leaving only the progress (debug.info) visible on the console.

An example of the output when saving to file follows:
```
$ vol.py --plugins linux_windows -f LinuxMint-17.3-Mate-x64-61951b91.vmem --profile LinuxMint173x64 linux_xatoms --output-file=linux_xatoms.txt
Volatility Foundation Volatility Framework 2.6
Outputting to: linux_xatoms.txt
INFO    : volatility.debug    : Working with 'Xorg' (pid=6319).
INFO    : volatility.debug    : Seeking atomRoot.
INFO    : volatility.debug    : Heap: rw- 0x7f39527eb000 0x7f3955b5a000
INFO    : volatility.debug    : <AtomNode(offset=0x7f39527ef150, a=0x1, string=PRIMARY)>
INFO    : volatility.debug    : Found 488 atom(s).
INFO    : volatility.debug    : Working with 'Xorg' (pid=6819).
INFO    : volatility.debug    : Seeking atomRoot.
INFO    : volatility.debug    : Heap: rw- 0x7f9f32071000 0x7f9f3404e000
INFO    : volatility.debug    : <AtomNode(offset=0x7f9f32075150, a=0x1, string=PRIMARY)>
INFO    : volatility.debug    : Found 485 atom(s).
```

## Example output in 'linux_xatoms.txt'.
```
<AtomNode(offset=0x7f9f326dd180, a=0x1d6, string=SAVE_TARGETS)>
<AtomNode(offset=0x7f9f326de4b0, a=0x1d7, string=ESETROOT_PMAP_ID)>
<AtomNode(offset=0x7f9f32576ae0, a=0x1d8, string=bottom_right_corner)>
<AtomNode(offset=0x7f9f3234ea20, a=0x1d9, string=_GTK_THEME_VARIANT)>
<AtomNode(offset=0x7f9f3234ea50, a=0x1da, string=_GTK_HIDE_TITLEBAR_WHEN_MAXIMIZED)>
<AtomNode(offset=0x7f9f326463c0, a=0x1db, string=top_side)>
<AtomNode(offset=0x7f9f3264c120, a=0x1dc, string=top_right_corner)>
<AtomNode(offset=0x7f9f32d0f7a0, a=0x1dd, string=right_side)>
<AtomNode(offset=0x7f9f32fad4b0, a=0x1de, string=bottom_side)>
<AtomNode(offset=0x7f9f3268aab0, a=0x1df, string=_MOZILLA_VERSION)>
<AtomNode(offset=0x7f9f32f9a550, a=0x1e0, string=_MOZILLA_LOCK)>
<AtomNode(offset=0x7f9f32f9a5a0, a=0x1e1, string=_MOZILLA_RESPONSE)>
<AtomNode(offset=0x7f9f3268c9d0, a=0x1e2, string=_MOZILLA_USER)>
<AtomNode(offset=0x7f9f32fb82b0, a=0x1e3, string=_MOZILLA_PROFILE)>
<AtomNode(offset=0x7f9f3268a340, a=0x1e4, string=_MOZILLA_PROGRAM)>
<AtomNode(offset=0x7f9f3268a370, a=0x1e5, string=_MOZILLA_COMMANDLINE)>
```

# How does it work?
The X process has a global named `screenInfo`. `screeninfo` contains an array of up to 16 `screens`. Each screen has it's own id, x, y, width, height, etc, but also a pointer to the root window struct (`root`). Each window struct points to its next and previous siblings and its first and last children. Consequently, having identified the root window, it is possible to walk all the other windows.

Each window struct contains a pointer to a `WindowOpt` struct, one of the properties of which is a pointer to the root `property`. Each property contains a pointer to the next property and so from the root property, all properties can be walked.

Similarly, the X process has a global named `atomRoot`. Each atom contains pointers to it's left and right atoms and so having found the root atom node, the list of all atoms can be walked.

The open-source nature of X means there are many different versions and so it is not practical to maintain a list of known offsets for the `screenInfo` or `atomRoot` globals.

Research identified that the `screenInfo` global can always be found in the anonymous memory range which immediately follows the 'rw' range of the mapped X/Xorg binary (think '/proc/<pid>/map') and by checking every address in the (reasonably small) range for sane values the global can reliably be identified. In testing, this process has never returned a false positive.

Similarly, research conducted by the author showed that the `atomRoot` global is stored on the heap and so the heap can be searched for sane values. The first (root) atom has an id of 0x1 and a value of "PRIMARY". Consequently it can be reliably signatured and located. In testing, this process has never returned a false positive.

# Making it "Gaslight Proof"
Taking into consideration the work demonstrated by the [Gaslight](http://www.sciencedirect.com/science/article/pii/S1742287617301986) framework, consideration was given to trying to prevent ungraceful exits.

Two kinds of checks are conducted on memory addresses throughout:

1) `is_valid_address` on all dereferences
Whenever an object is created by dereferencing a previously parsed memory address, `is_valid_address` is first called to ensure the address is valid.

2) A `set` is maintained of the memory addresses already parsed to an object. There is one set per X server for the atoms and one for the windows. If an attempt is made to parse the same memory address twice, it is skipped and reported via debug.info.

And, when outputting a property's data, output is capped at 512 bytes; the user is informed how many more bytes are stored in the property's data. (As can be seen in 'Notable Data in the Properties', example 3 above.)

# Testing
Both plugins have been tested and demonstrated as working on the following configurations:
1. CentOS 7 64-bit, KDE.
1. Linux Mint 17.3 64-bit, MATE.
1. Linux Mint 18.1 64-bit, Cinnamon.
1. Linux Mint 18.1 64-bit, MATE.
1. openSUSE  Leap 42.3 64-bit, KDE Plasma.
1. Ubuntu 14.04.5 64-bit, Unity.

# Limitations
Although all windows from all workspaces are output, it is not currently possible to distinguish which windows are on which workspace. (This might be a function of the desktop environment rather than X.)

The plugin only works on 64-bit Linux. I would imagine that with trivial changes to the vtypes, the plugin could be made to work with 32-bit Linux, but when trying to create 32-bit profiles I suffered the same issue as already reported to the Volatility github (https://github.com/volatilityfoundation/volatility/issues/416) which I could not overcome. (The plugin does check the profile and gracefully exit if the selected profile isn't Linux 64-bit.)
