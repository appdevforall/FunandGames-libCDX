# FunandGames (libCDX)

Three small Android game templates for [Code On The Go](https://github.com/appdevforall/CodeOnTheGo),
built with [libGDX](https://libgdx.com/). Each one creates a complete, playable game project that builds
and runs on the device. They are ports of the jMonkeyEngine templates in
[FunandGames-jme](https://github.com/appdevforall/FunandGames-jme).

| Template | Language | Game |
|---|---|---|
| **Demo (libCDX)** | Java | 3D Earth you drag to spin, an orbiting rocket with an exhaust trail and bloom glow, a starfield, tap for points |
| **Tetris (libCDX)** | Kotlin | Falling-block puzzle: drag to move, drag down to drop faster, tap to rotate, long-press to hard-drop |
| **Bubble Wand (libCDX)** | Kotlin | First-person shooter: catch floating balloon animals in bubbles using an on-screen D-pad and FIRE button |

All three use libGDX 1.13.1.

## Install in Code On The Go

1. Download [`FunandGames-libCDX.cgt`](FunandGames-libCDX.cgt) to the device's /sdcard/Download folder
2. Using the Add-ons manager in Preferences, install the templates
3. **Create a new project** → pick **Demo (libCDX)**, **Tetris (libCDX)** or **Bubble Wand (libCDX)** →
   name it → **Create**.
4. Wait for "Project initialized", then tap **Run**.

The first build downloads libGDX from Maven Central, so it needs an internet connection; later builds use
Gradle's cache.

## Controls

| Game | Controls |
|---|---|
| Demo | Drag to rotate the Earth · flick to spin with inertia · tap for +10 points |
| Tetris | Drag left/right to move · drag down to soft-drop · tap to rotate · long-press to hard-drop |
| Bubble Wand | Left half: drag to walk · right half: drag to look, tap to shoot · or use the D-pad and **FIRE** · **HIDE UI** toggles the buttons |

A hardware keyboard also works (arrow keys, space, WASD).

## Repository layout

```
demo/  tetris/  bubblewand/     one Code On The Go template each
  template/template.json        name, description and parameters shown in the IDE
  app/build.gradle.peb          app build script (natives unpacking, AndroidX pins)
  app/src/main/java/PACKAGE_NAME/*.peb   game source
  gradle/libs.versions.toml.peb          dependency versions
cgt/                            the three templates plus templates.json, as packaged
FunandGames-libCDX.cgt          zip of cgt/ — the file you install
```

Files ending in `.peb` are [Pebble](https://pebbletemplates.io/) templates. Code On The Go fills in
`${{PACKAGE_NAME}}`, `${{APP_NAME}}`, `${{AGP_VERSION}}`, `${{KOTLIN_VERSION}}`, `${{COMPILE_SDK}}` and the
other tokens listed in each `template.json` when it creates a project.

## How the build works

- **Native libraries:** libGDX ships `libgdx.so` inside per-ABI `gdx-platform` classifier jars. A
  `copyAndroidNatives` task in `app/build.gradle` unpacks them into `app/libs/<abi>/` (gitignored) before
  AGP merges native libraries. Builds include `armeabi-v7a` and `arm64-v8a` only.
- **AndroidX pins:** libGDX 1.13.x needs `androidx.core` at runtime. Code On The Go resolves `androidx.*`
  only from its offline Maven repository, which lacks some of what `androidx.core` 1.13.1 pulls in. The
  templates therefore force versions the repository has:

  | Module | Forced version |
  |---|---|
  | `androidx.core:core` | 1.9.0 |
  | `androidx.collection:collection` | 1.4.2 |
  | `androidx.annotation:annotation` | 1.8.1 |
  | `androidx.lifecycle:lifecycle-runtime`, `lifecycle-common` | 2.5.1 |

  These match the versions Code On The Go's own AndroidX templates use.

## Notes for template authors

- **AGP and Kotlin versions come from the IDE** (`${{AGP_VERSION}}`, `${{KOTLIN_VERSION}}`). Tested with
  AGP 9.3.1, Kotlin 2.3.21 and Gradle 9.6.1.
- **AGP 9 built-in Kotlin:** the Kotlin templates don't apply `org.jetbrains.kotlin.android` in the app
  module (AGP 9 rejects it); `jvmTarget` is set through `kotlin { compilerOptions { } }`.
- **Pebble drops the newline right after `}}`.** A token at the end of a line needs a trailing space
  (`compileSdk = ${{COMPILE_SDK}} `), or the next line gets joined onto it.
- **Don't drop to libGDX 1.12.x.** It avoids AndroidX, but its `libgdx.so` isn't 16 KB-page aligned, and
  Google Play Protect blocked the builds as harmful. libGDX 1.13.0 also uses `androidx.core` at runtime
  without declaring it, so it crashes on launch without the pins above.

## Rebuilding the package

After changing a template, copy it into `cgt/` and re-zip:

```bash
(cd cgt && zip -qr -X ../FunandGames-libCDX.cgt .)
```

`cgt/templates.json` lists the template folders the package contains.

## Tested

All three templates were downloaded from this repo, installed, built, installed as apps and played on a
Samsung tablet (SM-X238U, Android 16) with Code On The Go 26.40. Google Play Protect scanned each
build as safe.
