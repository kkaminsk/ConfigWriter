# ConfigWriter

A Win32 demonstration application that showcases file write failures when legacy applications are packaged as MSIX without the Package Support Framework (PSF).

## Purpose

ConfigWriter writes configuration files, logs, and data files to its installation directory - a common pattern in legacy Win32 applications that violates the MSIX security model. It demonstrates the need for PSF's FileRedirectionFixup.

## Technology Stack

- **Language:** C++ (Win32) or C# (.NET Framework 4.8)
- **Architecture:** x64
- **IDE:** Visual Studio 2022 (v143 toolset)
- **Windows SDK:** 10.0.22621.0+
- **C++ Standard:** C++17
- **Character Set:** Unicode

## Project Structure

```
ConfigWriter/
├── ConfigWriter.exe          # Main executable
├── settings.ini              # User settings (runtime created/modified)
├── app.log                   # Application log (runtime created/modified)
├── data.dat                  # Binary state file (runtime created/modified)
├── defaults.ini              # Read-only default settings (bundled)
└── resources/
    ├── app.ico
    └── splash.png
```

## Key Files

### settings.ini
Standard Windows INI format with sections: `[User]`, `[Behavior]`, `[Window]`
- Uses `GetPrivateProfileString()` / `WritePrivateProfileString()` APIs

### app.log
Text log file with format: `[YYYY-MM-DD HH:MM:SS] [LEVEL] Message`
- Levels: INFO, WARNING, ERROR
- Append mode only

### data.dat
Binary file storing window state and recent files list
- Magic header: "CWDT" (0x54445743)
- Uses `CreateFile()` / `ReadFile()` / `WriteFile()` APIs

## MSIX Behavior

| Operation | Without MSIX | MSIX (No PSF) | MSIX (With PSF) |
|-----------|--------------|---------------|-----------------|
| Read files | Works | Works | Works |
| Write files | Works | **ACCESS DENIED** | Works (redirected) |

## PSF Configuration

When packaged with PSF, files matching `*.ini`, `*.log`, `*.dat` are redirected from the package directory to:
```
%LOCALAPPDATA%\Packages\Contoso.ConfigWriter_8wekyb3d8bbwe\LocalCache\Local\VFS\
```

## Build

```batch
msbuild ConfigWriter.sln /p:Configuration=Release /p:Platform=x64
```

## Package Identity

- **Package Name:** Contoso.ConfigWriter
- **Publisher:** CN=Contoso Software, O=Contoso, C=US
- **Application ID:** ConfigWriter

## Minimum Requirements

- Windows 10 1809 (Build 17763)
- x64 architecture
