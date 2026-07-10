# North Launcher (C++/Qt)

A third-party Minecraft: Java Edition launcher: real Microsoft sign-in, official
version list, downloads client/libraries/assets, and launches the game — plus a
Windows installer `.exe` built with Inno Setup.

## What's included
- `src/Auth/` — Microsoft OAuth2 device-code login → Xbox Live → XSTS → Minecraft
  Services token chain (the same flow every legitimate third-party launcher uses).
- `src/Launcher/VersionManager` — reads Mojang's public version manifest.
- `src/Launcher/Downloader` — downloads client jar, libraries (with OS-rule
  filtering), extracts natives, downloads assets. All downloads are verified by SHA-1.
- `src/Launcher/GameLauncher` — builds the JVM/game argument line and starts `java`.
- `src/UI/MainWindow` — Qt Widgets UI tying it all together.
- `installer/installer.iss` — Inno Setup script producing `NorthLauncherSetup.exe`.
- `.github/workflows/build.yml` — builds the installer automatically in the cloud
  on every push, if you don't want to set up a local Windows toolchain at all.

## 1. Get your own Azure Client ID (required, ~2 minutes)
Microsoft requires every third-party app to have its own registration — this is
normal and free, not a paid developer account.
1. https://portal.azure.com → **App registrations** → **New registration**
2. Name it anything. Supported account types: **Personal Microsoft accounts only**.
3. Leave Redirect URI blank.
4. Copy the **Application (client) ID** GUID into `src/Config.h`:
   ```cpp
   inline constexpr const char* kAzureClientId = "paste-your-guid-here";
   ```

## 2. Build locally on Windows
Prerequisites: [Visual Studio 2022](https://visualstudio.microsoft.com/) (Desktop
C++ workload), [CMake](https://cmake.org/), [vcpkg](https://github.com/microsoft/vcpkg),
[Inno Setup](https://jrsoftware.org/isinfo.php).

```powershell
git clone <your-repo>
cd NorthLauncher

# one-time: bootstrap vcpkg if you haven't already
git clone https://github.com/microsoft/vcpkg
.\vcpkg\bootstrap-vcpkg.bat

cmake -B build -S . -DCMAKE_TOOLCHAIN_FILE=.\vcpkg\scripts\buildsystems\vcpkg.cmake
cmake --build build --config Release
```
This produces `build/Release/NorthLauncher.exe` with all Qt DLLs copied next
to it automatically (via `windeployqt`, wired into `CMakeLists.txt`).

## 3. Build the installer .exe
```powershell
cd installer
iscc installer.iss
```
Output: `installer/output/NorthLauncherSetup.exe` — a standard Windows
installer with Start Menu shortcuts, optional desktop icon, and an uninstaller.

## 4. Or: build in the cloud (no local Windows needed)
Push this repo to GitHub and the included Actions workflow
(`.github/workflows/build.yml`) will build both the app and the installer on a
hosted Windows runner automatically — download `NorthLauncherSetup.exe` from
the workflow's **Artifacts**.

## How the game data is stored
Everything lives in `%APPDATA%\.mclauncher\`: downloaded versions, libraries,
assets, and a cached Microsoft refresh token (so the user isn't asked to log in
every launch). Deleting that folder fully resets the launcher.

## Notes & limitations to be aware of
- **Java runtime**: this launcher shells out to `java` on PATH. For a truly
  "one-click" experience you'll want to bundle a JRE (e.g. Eclipse Temurin) and
  point `LaunchOptions::javaPath` at it — not included here to keep the installer
  small, but straightforward to add.
- **Mod loaders (Forge/Fabric)**: `VersionManager::resolveInheritance` already
  handles the `inheritsFrom` merging their version JSONs use; you'd add a small
  UI step to install a loader profile on top of a vanilla version.
- This project only supports logging in with **real, purchased Microsoft
  accounts** — there's intentionally no offline/cracked mode, since that would
  violate Minecraft's EULA and Microsoft's terms of service.
- You're responsible for complying with [Mojang's usage guidelines for the
  Minecraft name/brand](https://www.minecraft.net/en-us/usage-guidelines) if you
  distribute this publicly (e.g., don't call it "Minecraft" itself or use
  Mojang's logo — "a launcher for Minecraft" is fine, implying official
  endorsement is not).
