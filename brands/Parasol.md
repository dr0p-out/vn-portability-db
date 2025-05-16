Parasol
=======

[VNDB page](https://vndb.org/p1080)

## 晴のちきっと菜の花びより

[VNDB page](https://vndb.org/v14886)

compatible runtimes: [吉里吉里SDL2](../engines/吉里吉里SDL2.md) (experimental)

### Troubleshooting

#### `正常に起動できませんでした`

the game expects `haruno.cf` file to be present in its directory, creating a plain text file with these contents should do the trick:

```plaintext
hkcontroller=""
hkeditor=""
hkwatch=""
hkconsole=""
```

if it still happens, make sure the engine program is in same directory with this file.

#### `Cannot find storage sys_source/Init.tjs`

this game has a different resource structure, so you probably want some extra imports in `startup.tjs`:

```javascript
Storages.addAutoPath("data.xp3>sys_source/");
Storages.addAutoPath("data.xp3>macro/");
```

#### `Cannot find storage userFix.ks`

see above

#### `Not a function or invalid method/property type`

this may be related to the fact this game has some auto-loaded plugins, which only works depending on the engine variant you're using, try to load them manually in `startup.tjs`:

```javascript
// change the file names to what you actually
//  have under your `plugin` folder.
Plugins.link("wuvorbis.so");
Plugins.link("fstat.so");
```

#### `Cannot open storage file://./FullPathOfGameDir/frm_meswin.png`

this game has more than one .xp3 file, and you likely have some of those files missing
