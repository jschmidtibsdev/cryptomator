# Cryptomator (Full Feature Fork)

Fork of [cryptomator/cryptomator](https://github.com/cryptomator/cryptomator) with all features unlocked (dark mode, automatic theme, no donate banner). Licensed under GPL-3.0.

## What Changed

The license check in `src/main/java/org/cryptomator/common/LicenseHolder.java` was modified so that `isValidLicense()` always returns `true` and `validLicenseProperty()` always binds to `true`. This unlocks:

- Dark mode and automatic theme switching
- Hides the donate/supporter certificate banner

No functionality was removed — the supporter certificate UI still works if you want to enter a key.

## Prerequisites (Windows)

You need four things installed. Open PowerShell **as Administrator** and run:

### 1. JDK 25 (Azul Zulu, recommended by upstream)

```powershell
winget install Azul.Zulu.25.JDK
```

After install, set `JAVA_HOME` if not set automatically:

```powershell
# Find where it installed (adjust version as needed)
$javaPath = (Get-ChildItem "C:\Program Files\Zulu" -Filter "zulu-25*" | Select-Object -First 1).FullName
[Environment]::SetEnvironmentVariable("JAVA_HOME", $javaPath, "User")
```

### 2. Maven (manual install — not available in winget)

1. Download the latest binary zip from https://maven.apache.org/download.cgi (e.g. `apache-maven-3.9.9-bin.zip`)
2. Extract to `C:\Program Files\Maven`
3. Add to PATH:

```powershell
# Run as Administrator
[Environment]::SetEnvironmentVariable("PATH", $env:PATH + ";C:\Program Files\Maven\apache-maven-3.9.9\bin", "Machine")
```

### 3. PowerShell Core (the build script requires `pwsh`)

```powershell
winget install Microsoft.PowerShell
```

### 4. .NET SDK (needed for WiX toolset)

```powershell
winget install Microsoft.DotNet.SDK.8
```

### 5. WiX Toolset 6 (creates the .msi/.exe installers)

```powershell
dotnet tool install --global wix --version 6.0.2
wix extension add --global WixToolset.UI.wixext/6.0.2
wix extension add --global WixToolset.Util.wixext/6.0.2
wix extension add --global WixToolset.BootstrapperApplications.wixext/6.0.2
```

### Verify everything

**Restart your terminal**, then:

```powershell
java -version      # should show 25.x
mvn -version       # should show 3.9+
wix --version      # should show 6.0.2
```

## Building

### Quick: just compile

```powershell
mvn package -DskipTests
```

### Full: build the .msi and .exe installer

From the repo root:

```powershell
cd dist\win
.\build.bat
```

This will:
1. Compile the Java project with Maven
2. Create a custom JRE with `jlink` (no Java needed on target machine)
3. Package an app image with `jpackage`
4. Build a `.msi` installer with WiX
5. Bundle the `.msi` + WinFsp into a standalone `.exe` installer

Output:
- `dist\win\installer\Cryptomator-*.msi` — standalone MSI
- `dist\win\installer\Cryptomator-Installer.exe` — EXE bundle (includes WinFsp)

### Run without packaging

```powershell
mvn javafx:run
```

## Merging Upstream Changes

Add the upstream remote (one-time setup):

```powershell
git remote add upstream https://github.com/cryptomator/cryptomator.git
```

Pull in recent changes:

```powershell
git fetch upstream
git merge upstream/develop
```

The only file likely to conflict is `LicenseHolder.java`. When resolving, keep these two changes:

1. In the constructor: `this.validLicenseProperty = Bindings.createBooleanBinding(() -> true);`
2. In `isValidLicense()`: `return true;`
