吉里吉里SDL2
============

[project page (Japanese)](https://krkrsdl2.github.io/krkrsdl2/)

- games that can be launched with this engine stores its media resources and scenario texts inside a file called `data.xp3` usually found inside its directory, some games use multiple categorized packages instead of one, you will likely need all of these files in order to launch

- this engine also support starting directly from an unpackaged folder (**disclaimers**: DO NOT try to extract a game's .xp3 file which might otherwise put you at risks for copyright infringements), which by definition has the same content of what a `data.xp3` would have, in this case, significantly, it looks for a file called `startup.tjs` which is the start of all game logics (TJS is its small in-house JavaScript-inspired scripting language, used for controlling the logics for presenting the game)

- the program takes an argument for specifying the location of an .xp3 file, or a directory that contains `startup.tjs`. if no argument is passed, it looks for `data.xp3` at its own location or `startup.tjs` at the [current directory](https://en.wikipedia.org/wiki/Working_directory). this page will also mention some other options (starting with a dash) that can be passed in launch arguments

- some features needed by games are provided by plugins, the engine looks for [shared library files](https://en.wikipedia.org/wiki/Shared_library) around it and also in a directory called `plugin` at its location. while compiling it's also possible to have some built-in plugins to be included with the engine itself, built-in plugins are automatically loaded at engine start

## Portability

* there should be quite some commercial games that is playable by just launching its .xp3 files using a vanilla version of this engine, given that if it didn't use any proprietary plugin that don't yet have a free reimplementation available.

* at the moment, some plugins were heavily coupled with Windows semantics and couldn't be easily ported to this version of engine.

## Installation

- compile or download the engine program from https://github.com/krkrsdl2/krkrsdl2-env-example, this already bundled many frequently used plugins.

- if you prefer to have the plugins living separately, use https://github.com/krkrsdl2/krkrsdl2 and grab the plugins based on your needs:

  * [KAGParser](https://github.com/krkrsdl2/KAGParser): commonly used scenario script parser

  * [wuvorbis](https://github.com/krkrsdl2/wuvorbis): .ogg audio playback support

  * [fstat](https://github.com/krkrsdl2/fstat): some extensions to file management functions

- in theory, this version of engine will also work on Android devices, if you compile it yourself

- even though it's not necessary, this engine can also work if launched under [Wine](https://www.winehq.org/) (this is an option if you absolutely can't use a native version, for example, if you need to use a Windows-only plugin you compiled yourself using [a MinGW cross-compiler](https://github.com/mstorsjo/llvm-mingw))

### Troubleshooting

#### `passing argument 1 of ‘pw_node_enum_params’ from incompatible pointer type`

this is likely due to the pipewire audio library installed on your system is too new but the SDL2 source code inside the engine is outdated, you can either apply [a patch](https://github.com/libsdl-org/SDL/commit/6be87ce) or disable pipewire support by passing `-DSDL_PIPEWIRE=OFF` to CMake when compiling

#### `‘ptrdiff_t’ does not name a type`

the plugins can fail to compile under archlinux for some reasons, if this is what you're trying to do, try to either add the missing `#include` lines according to what the compiler says, or building everything with a different compiler and standard library somehow works (for example `export CC=clang CXX=clang++ CXXFLAGS=' -stdlib=libc++'` before building the engine and plugins)

#### `segmentation fault (core dumped)`

if you compiled the engine yourself and got this under Linux, it's *probably* okay to ignore it if it happens only when you exit the game. if this prevent you from launching the game, try compiling it with a different compiler or standard library, see above

#### `CPU family 4 or lesser is not supported`

if you're trying to run on a device with ARM processor, this was a bug, try upgrading your engine program to latest and try again

## Pitfalls

* it's sometimes necessary to have the engine program living in same directory with other game files, this is because some games depend on relative paths to find its files.

* by default this engine saves user data of every game you launch at a single global location, you should launch games with `-datapath=file://.ReplaceMeWithPathToYourGameDir/savedata` (tweak the path accordingly yourself; must be a full a.k.a. absolute path not [relative](https://en.wikipedia.org/wiki/Path_(computing))) in order to store these separately, which is likely necessary to get the intended behavior.

* if you have created a `startup.tjs` file and added something in it but the game is not picking it up, this is because it prioritize `data.xp3` as entrypoint over your script file, you can overcome this by launching with `-nosel ReplaceMeWithPathToYourGameDir` (again, tweak the path accordingly yourself)

* if you didn't place the engine program in same directory with the `plugin` folder, it won't be able to see plugins in there. but it could if you launch with `-krkrsdl2_pluginsearchpath=ReplaceMeWithPathToYourGameDir/plugin` (and again tweak the path accordingly yourself)

* if the original game has some plugins ending with .tpm extension, it's an auto-loaded plugin, this behavoir is not yet supported by this variant of engine and you must load these manually in `startup.tjs` using `Plugins.link`

* sometimes games might ship more plugins than it actually use, so don't be fooled by it. the only way to know what plugins it needs is to actually launch the game and experiment

* some games has its .xp3 files encrypted, the best you can do is to leave it and find another game with unencrypted resources

## Known workarounds

some section in this part requires you adding lines into `startup.tjs`, in most cases (unless you have an unpackaged folder with game resources), you need to create this file yourself, and then it usually contains the following:

```javascript
// import all resources from game's package.
// you may need to add more of these
//  depending on the game you run.
// only data.xp3 is needed here!
Storages.addAutoPath("data.xp3>video/");
Storages.addAutoPath("data.xp3>others/");
Storages.addAutoPath("data.xp3>rule/");
Storages.addAutoPath("data.xp3>sound/");
Storages.addAutoPath("data.xp3>bgm/");
Storages.addAutoPath("data.xp3>fgimage/");
Storages.addAutoPath("data.xp3>bgimage/");
Storages.addAutoPath("data.xp3>scenario/");
Storages.addAutoPath("data.xp3>image/");
Storages.addAutoPath("data.xp3>system/");

// chain-loads the game's original start script.
//
// always make sure this line is at the end,
// i.e. DO NOT add anything after this,
//  add everything above!!
Scripts.execStorage("data.xp3>startup.tjs");
```

the idea is to force the game to begin with our startup logics, so we can apply any desired fixes, and when that is done transfer the execution back to the game and let it do what it was originally designed to.

### `Cannot convert given narrow string to wide string`

likely the game has a different text encoding, try launching again with `-readencoding=Shift_JIS`. another equivalent way is adding `Scripts.textEncoding = "Shift_JIS";` in `startup.tjs`

### `Member "MenuItem" does not exist`

menu bar is not yet supported by this engine. for now it's likely possible to launch without it, try adding these into `startup.tjs`:

```javascript
// https://github.com/krkrsdl2/kag3/blob/krkrsdl2/data/system_polyfill/MenuItem_stub.tjs
class MenuItem
{
  var caption, checked, enabled, group, radio, shortcut, visible, parent, children, root, window, index;
  function MenuItem() { children = []; }
  function finalize() {}
  function add() {}
  function insert() {}
  function remove() {}
  function popup() {}
  function onClick() {}
  function fireClick() {}
}

Window.menu = new MenuItem();
```

### `Member "innerSunken" does not exist`

obsolete features and were removed from engine, it's likely possible to launch the game after adding these into `startup.tjs`:

```javascript
// https://github.com/krkrsdl2/kag3/blob/krkrsdl2/data/system_polyfill/PolyfillInitialize.tjs
property _dummyProp { getter {} setter (v) {} }
with(Window)
{
  &.innerSunken = &_dummyProp;
  &.showScrollBars = &_dummyProp;
}
```

### `Member "showScrollBars" does not exist`

see above

### `Font 'Noto Sans CJK JP$s' cannot be used`

the engine doesn't yet seems to support discovering system-widely installed fonts, try to prepare a font file yourself under the game's directory (for example download [this one](https://github.com/krkrsdl2/kag3/raw/refs/heads/krkrsdl2/data/system_polyfill/font.ttf)), and add these into `startup.tjs`:

```javascript
// change font.ttf to the actual file name of your font
System.setArgument("-deffont", Font.addFont("font.ttf")[0]);
```

### `Member "PassThroughDrawDevice" does not exist`

again, obsolete features and were removed from engine, try launching again after adding these into `startup.tjs`:

```javascript
Window.PassThroughDrawDevice = %[ recreate:function{}, dtNone:0, dtDrawDib:1, dtDBGDI:2, dtDBDD:3, dtDBD3D:4 ];
```

### `Member "KAGParser" does not exist`

you should instruct the game to load this plugin manually, add these into `startup.tjs`:

```javascript
// change the file name to what you actually
//  have under your `plugin` folder.
Plugins.link("KAGParser.so");
```

### `Member "fstat" does not exist`

see above, change the file name accordingly

### `Cannot load Plugin wuvorbis.dll`

you should double check if that given plugin exists in your `plugin` folder (the file extensions doesn't matter, for example, having a `wuvorbis.so` should do too), if yes, make sure you have this folder in same directory with the engine program. at last sort, try to use a version of the engine program which has this plugin built-in

### `Cannot find storage abc/xyz`

this probably means you need another line of `Storages.addAutoPath("data.xp3>abc/");` in your `startup.tjs` file

### `Member "registerExEvent" does not exist`

this was supposed to be provided by a not yet supported plugin, for now you can try to add some empty implementations in `startup.tjs`:

```javascript
Window.registerExEvent = function() {};
Window.setMessageHook = function() { return 0; };
```

### `Member "setMessageHook" does not exist`

see above

### `Member "pileRect" does not exist`

a few functions were refactored in engine, to make the game work, add these in `startup.tjs`:

```javascript
// https://github.com/krkrz/documents/blob/master/TJS2/deleted.md#layer-%E3%82%AF%E3%83%A9%E3%82%B9%E3%81%AE-obsolete-%E3%83%A1%E3%82%BD%E3%83%83%E3%83%89
Layer.affineBlend = function(src, sleft, stop, swidth, sheight, affine, A, B, C, D, E, F, opa=255, type=stNearest) {
  this.operateAffine(src, sleft, stop, swidth, sheight, affine, A, B, C, D, E, F, omOpaque, opa, type);
};
Layer.affinePile = function(src, sleft, stop, swidth, sheight, affine, A, B, C, D, E, F, opa=255, type=stNearest) {
  this.operateAffine(src, sleft, stop, swidth, sheight, affine, A, B, C, D, E, F, omAuto, opa, type);
};
Layer.blendRect = function(dleft, dtop, src, sleft, stop, swidth, sheight, opa=255) {
  this.operateRect(dleft, dtop, src, sleft, stop, swidth, sheight, omOpaque, opa);
};
Layer.pileRect = function(dleft, dtop, src, sleft, stop, swidth, sheight, opa=255) {
  this.operateRect(dleft, dtop, src, sleft, stop, swidth, sheight, omAuto, opa);
};
Layer.stretchBlend = function(dleft, dtop, dwidth, dheight, src, sleft, stop, swidth, sheight, opa=255, type=stNearest) {
  this.operateStretch(dleft, dtop, dwidth, dheight, src, sleft, stop, swidth, sheight, omOpaque, opa, type);
};
Layer.stretchPile = function(dleft, dtop, dwidth, dheight, src, sleft, stop, swidth, sheight, opa=255, type=stNearest) {
  this.operateStretch(dleft, dtop, dwidth, dheight, src, sleft, stop, swidth, sheight, omAuto, opa, type);
};
```

### `Unknown wave format xxx.ogg`

you likely need a plguin (in this example, could be `wuvorbis`) for these audio to play

## Known issues

* Video playback is not yet supported by engine, a black screen will be displayed if the game attempts to play one, in most games, you can "skip" the video by pressing ENTER key or by clicking the mouse

* Playing back audio crashes when launched under Wine on a 64-bit ARM machine, this bug needs to be fixed by recompiling Wine with the following patches:

```diff
--- wine-10.0/dlls/ntdll/unix/system.c.orig	2025-05-17 00:45:23.087029072 +0900
+++ wine-10.0/dlls/ntdll/unix/system.c	2025-05-17 00:45:42.068872705 +0900
@@ -620,6 +620,7 @@
                 if (has_feature(value, "crc32"))      features |= CPU_FEATURE_ARM_V8_CRC32;
                 if (has_feature(value, "aes"))        features |= CPU_FEATURE_ARM_V8_CRYPTO;
                 if (has_feature(value, "atomics"))    features |= CPU_FEATURE_ARM_V81_ATOMIC;
+                if (has_feature(value, "asimd"))      features |= CPU_FEATURE_ARM_NEON;
                 if (has_feature(value, "asimddp"))    features |= CPU_FEATURE_ARM_V82_DP;
                 if (has_feature(value, "jscvt"))      features |= CPU_FEATURE_ARM_V83_JSCVT;
                 if (has_feature(value, "lrcpc"))      features |= CPU_FEATURE_ARM_V83_LRCPC;
--- wine-10.0/programs/wineboot/wineboot.c.orig	2025-05-17 02:40:27.018443788 +0900
+++ wine-10.0/programs/wineboot/wineboot.c	2025-05-17 02:40:57.142124187 +0900
@@ -468,6 +468,7 @@
         features[PF_ARM_V8_CRC32_INSTRUCTIONS_AVAILABLE]  = !!(sci.ProcessorFeatureBits & CPU_FEATURE_ARM_V8_CRC32);
         features[PF_ARM_V8_CRYPTO_INSTRUCTIONS_AVAILABLE] = !!(sci.ProcessorFeatureBits & CPU_FEATURE_ARM_V8_CRYPTO);
         features[PF_ARM_V81_ATOMIC_INSTRUCTIONS_AVAILABLE]= !!(sci.ProcessorFeatureBits & CPU_FEATURE_ARM_V81_ATOMIC);
+        features[PF_ARM_NEON_INSTRUCTIONS_AVAILABLE]      = !!(sci.ProcessorFeatureBits & CPU_FEATURE_ARM_NEON);
         features[PF_ARM_V82_DP_INSTRUCTIONS_AVAILABLE]    = !!(sci.ProcessorFeatureBits & CPU_FEATURE_ARM_V82_DP);
         features[PF_ARM_V83_JSCVT_INSTRUCTIONS_AVAILABLE] = !!(sci.ProcessorFeatureBits & CPU_FEATURE_ARM_V83_JSCVT);
         features[PF_ARM_V83_LRCPC_INSTRUCTIONS_AVAILABLE] = !!(sci.ProcessorFeatureBits & CPU_FEATURE_ARM_V83_LRCPC);
```
