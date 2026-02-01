# ConfigWriter Application Specification

**Version:** 1.0.0
**Last Updated:** 2026-01-31
**Status:** Draft
**PSF Scenario:** File Redirection

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Application Overview](#2-application-overview)
3. [Functional Requirements](#3-functional-requirements)
4. [Technical Specifications](#4-technical-specifications)
5. [User Interface Design](#5-user-interface-design)
6. [File System Interactions](#6-file-system-interactions)
7. [MSIX Packaging Considerations](#7-msix-packaging-considerations)
8. [PSF Configuration](#8-psf-configuration)
9. [Testing Strategy](#9-testing-strategy)
10. [Build Instructions](#10-build-instructions)

---

## 1. Executive Summary

### Purpose

ConfigWriter is a demonstration application designed to showcase file write failures that occur when traditional Win32 applications are packaged as MSIX without the Package Support Framework (PSF). The application writes configuration files, logs, and data files to its installation directory—a common pattern in legacy Win32 applications that violates the MSIX security model.

### PSF Fixup Required

**FileRedirectionFixup** - Redirects file system writes from the protected package installation directory to a writable per-user location while maintaining read access to the original package files.

### Key Behaviors

| Behavior | Without MSIX | MSIX (No PSF) | MSIX (With PSF) |
|----------|--------------|---------------|-----------------|
| Read settings.ini | Works | Works | Works |
| Write settings.ini | Works | **ACCESS DENIED** | Works (redirected) |
| Create app.log | Works | **ACCESS DENIED** | Works (redirected) |
| Update data.dat | Works | **ACCESS DENIED** | Works (redirected) |

### Design Philosophy

This application is intentionally simple. The goal is to clearly demonstrate MSIX file access failures, not to build a feature-rich application. Complexity should be avoided—the minimal implementation that fails visibly is the correct implementation.

---

## 2. Application Overview

### Identity

| Field | Value |
|-------|-------|
| **Application Name** | ConfigWriter |
| **Display Name** | ConfigWriter Settings Manager |
| **Version** | 1.0.0.0 |
| **Publisher** | Contoso Software |
| **Language** | C++ (Win32) |
| **Architecture** | x64 |
| **Subsystem** | Windows GUI |

### Package Identity (MSIX)

| Field | Value |
|-------|-------|
| **Package Name** | Contoso.ConfigWriter |
| **Package Family Name** | Contoso.ConfigWriter_8wekyb3d8bbwe |
| **Publisher** | CN=Contoso Software, O=Contoso, C=US |
| **Application ID** | ConfigWriter |

### Description

ConfigWriter is a minimal settings management application that demonstrates the common legacy pattern of storing configuration data alongside the application executable. This pattern creates compatibility issues when packaged as MSIX, making it an ideal candidate for PSF FileRedirectionFixup demonstration.

**Implementation Goal:** Keep the application as simple as possible while clearly demonstrating the ACCESS_DENIED failures that occur in MSIX without PSF.

---

## 3. Functional Requirements

### FR-1: Application Startup

**Priority:** Critical

| ID | Requirement |
|----|-------------|
| FR-1.1 | On launch, the application SHALL attempt to read `settings.ini` from the application's installation directory |
| FR-1.2 | If `settings.ini` does not exist, the application SHALL create it with default values |
| FR-1.3 | If `settings.ini` exists but is unreadable, the application SHALL display an error dialog and continue with defaults |
| FR-1.4 | The application SHALL display current settings values in the main window after loading |
| FR-1.5 | The application SHALL log the startup event to `app.log` |

**Default Settings Values:**
```ini
[User]
Username=DefaultUser
Theme=Light
```

> **Note:** Settings are intentionally minimal. Two values are sufficient to demonstrate the MSIX write failure.

### FR-2: Settings Management

**Priority:** Critical

| ID | Requirement |
|----|-------------|
| FR-2.1 | The application SHALL provide UI controls to modify the Username setting (string, max 50 characters) |
| FR-2.2 | The application SHALL provide UI controls to select Theme (Light or Dark, radio buttons) |
| FR-2.3 | When the user clicks "Save Settings", the application SHALL persist all settings to `settings.ini` |
| FR-2.4 | Settings SHALL be saved automatically when the application exits |

> **Simplified:** AutoSave checkbox, LastOpenedFile display, and Reset button are optional enhancements.

### FR-3: Logging

**Priority:** High

| ID | Requirement |
|----|-------------|
| FR-3.1 | The application SHALL write log entries to `app.log` in the installation directory |
| FR-3.2 | Log entries SHALL use the format: `[YYYY-MM-DD HH:MM:SS] [LEVEL] Message` |
| FR-3.3 | Log levels SHALL include: INFO, WARNING, ERROR |
| FR-3.4 | The application SHALL log: startup, settings load, settings save, and errors |
| FR-3.5 | The log file SHALL be appended to (not overwritten) on each application launch |
| FR-3.6 | The application SHALL display recent log entries in a read-only text area in the UI |

**Log Entry Examples:**
```
[2026-01-31 10:30:00] [INFO] Application started
[2026-01-31 10:30:00] [INFO] Loading settings from settings.ini
[2026-01-31 10:30:01] [INFO] Settings loaded successfully
[2026-01-31 10:35:22] [INFO] Settings saved by user
[2026-01-31 10:40:00] [ERROR] Failed to write settings: Access denied
```

### FR-4: Data Files (Optional)

**Priority:** Low

| ID | Requirement |
|----|-------------|
| FR-4.1 | The application MAY maintain a `data.dat` file for window state persistence |
| FR-4.2 | `data.dat` SHALL store window position (X, Y) and size (Width, Height) |
| FR-4.3 | Window position and size SHALL be restored on startup and saved on exit |

> **Note:** This feature is optional. The settings.ini and app.log files are sufficient to demonstrate PSF need. If implemented, skip the recent files list to keep it simple.

### FR-5: Error Handling

**Priority:** High

| ID | Requirement |
|----|-------------|
| FR-5.1 | When file write operations fail, the application SHALL display a user-friendly error message |
| FR-5.2 | Error messages SHALL include the operation that failed and the error code |
| FR-5.3 | The application SHALL NOT crash when file operations fail |
| FR-5.4 | Failed write operations SHALL be logged (if logging is functional) |
| FR-5.5 | The application SHALL provide a "Retry" option for failed save operations |

---

## 4. Technical Specifications

### 4.1 Development Environment

| Component | Specification |
|-----------|--------------|
| **IDE** | Visual Studio 2022 (17.8+) |
| **Platform Toolset** | v143 |
| **Windows SDK** | 10.0.22621.0 or later |
| **C++ Standard** | C++17 |
| **Runtime Library** | Static (/MT) - avoids VC++ Redist dependency |
| **Character Set** | Unicode |

### 4.2 System Requirements

| Requirement | Minimum | Recommended |
|-------------|---------|-------------|
| **OS** | Windows 10 1809 (Build 17763) | Windows 11 22H2 |
| **Architecture** | x64 | x64 |
| **RAM** | 256 MB | 512 MB |
| **Disk Space** | 10 MB | 20 MB |

### 4.3 File Structure

```
C:\Program Files\ConfigWriter\
├── ConfigWriter.exe          # Main executable (entry point)
├── settings.ini              # User settings (runtime created/modified)
├── app.log                   # Application log (runtime created/modified)
├── data.dat                  # Window state (optional, runtime created/modified)
└── resources\
    └── app.ico               # Application icon
```

> **Simplified:** defaults.ini and splash.png are optional. The core demo requires only the executable and the three runtime files.

### 4.4 Settings File Format (settings.ini)

The settings file uses standard Windows INI format with minimal sections:

```ini
; ConfigWriter Settings File

[User]
Username=DefaultUser
Theme=Light
```

> **Simplified:** Only two settings are required to demonstrate the MSIX write failure. Additional sections ([Behavior], [Window]) are optional.

### 4.5 Data File Format (data.dat) - Optional

Simple binary format for window state:

```
Offset  Size    Field           Description
0x00    4       Magic           "CWDT" (0x54445743)
0x04    4       Version         File format version (1)
0x08    4       WindowX         Window X position
0x0C    4       WindowY         Window Y position
0x10    4       WindowWidth     Window width
0x14    4       WindowHeight    Window height
```

> **Note:** This file is optional. If implemented, skip the recent files list. The settings.ini and app.log are sufficient to demonstrate PSF need.

### 4.6 Core Code Implementation

#### Path Resolution (Critical for MSIX Failure)

```cpp
#include <windows.h>
#include <shlwapi.h>
#include <pathcch.h>

#pragma comment(lib, "shlwapi.lib")
#pragma comment(lib, "pathcch.lib")

class PathResolver {
public:
    static std::wstring GetExecutableDirectory() {
        wchar_t exePath[MAX_PATH];
        GetModuleFileNameW(NULL, exePath, MAX_PATH);
        PathRemoveFileSpecW(exePath);
        return std::wstring(exePath);
    }

    static std::wstring GetSettingsPath() {
        return GetExecutableDirectory() + L"\\settings.ini";
    }

    static std::wstring GetLogPath() {
        return GetExecutableDirectory() + L"\\app.log";
    }

    static std::wstring GetDataPath() {
        return GetExecutableDirectory() + L"\\data.dat";
    }
};
```

#### Settings Read/Write (Will Fail in MSIX)

```cpp
class SettingsManager {
private:
    std::wstring m_settingsPath;

public:
    SettingsManager() {
        m_settingsPath = PathResolver::GetSettingsPath();
    }

    // Read setting - Works in MSIX (reads from package)
    std::wstring ReadString(const wchar_t* section, const wchar_t* key,
                            const wchar_t* defaultValue) {
        wchar_t buffer[256];
        GetPrivateProfileStringW(section, key, defaultValue,
                                 buffer, 256, m_settingsPath.c_str());
        return std::wstring(buffer);
    }

    // Write setting - FAILS IN MSIX WITHOUT PSF
    bool WriteString(const wchar_t* section, const wchar_t* key,
                     const wchar_t* value) {
        BOOL result = WritePrivateProfileStringW(section, key, value,
                                                  m_settingsPath.c_str());
        if (!result) {
            DWORD error = GetLastError();
            // Error 5 = ERROR_ACCESS_DENIED (common in MSIX)
            LogError(L"Failed to write setting", error);
            return false;
        }
        return true;
    }
};
```

#### Logging Implementation (Will Fail in MSIX)

```cpp
class Logger {
private:
    std::wstring m_logPath;

public:
    Logger() {
        m_logPath = PathResolver::GetLogPath();
    }

    // FAILS IN MSIX WITHOUT PSF
    bool Log(LogLevel level, const wchar_t* message) {
        FILE* logFile = _wfopen(m_logPath.c_str(), L"a");
        if (logFile == nullptr) {
            // errno will be EACCES (13) in MSIX
            return false;
        }

        // Get current timestamp
        SYSTEMTIME st;
        GetLocalTime(&st);

        const wchar_t* levelStr = GetLevelString(level);

        fwprintf(logFile, L"[%04d-%02d-%02d %02d:%02d:%02d] [%s] %s\n",
                 st.wYear, st.wMonth, st.wDay,
                 st.wHour, st.wMinute, st.wSecond,
                 levelStr, message);

        fclose(logFile);
        return true;
    }

private:
    const wchar_t* GetLevelString(LogLevel level) {
        switch (level) {
            case LogLevel::Info:    return L"INFO";
            case LogLevel::Warning: return L"WARNING";
            case LogLevel::Error:   return L"ERROR";
            default:                return L"UNKNOWN";
        }
    }
};
```

#### Data File Operations (Optional - Will Fail in MSIX)

```cpp
// Optional: Only implement if time permits
struct DataHeader {
    DWORD magic;        // "CWDT"
    DWORD version;
    int windowX;
    int windowY;
    int windowWidth;
    int windowHeight;
};

class DataManager {
private:
    std::wstring m_dataPath;

public:
    DataManager() {
        m_dataPath = PathResolver::GetDataPath();
    }

    // FAILS IN MSIX WITHOUT PSF
    bool SaveWindowState(int x, int y, int width, int height) {
        HANDLE hFile = CreateFileW(
            m_dataPath.c_str(),
            GENERIC_WRITE,
            0,
            NULL,
            CREATE_ALWAYS,
            FILE_ATTRIBUTE_NORMAL,
            NULL
        );

        if (hFile == INVALID_HANDLE_VALUE) {
            DWORD error = GetLastError();
            // Error 5 = ERROR_ACCESS_DENIED in MSIX
            return false;
        }

        DataHeader header = {
            0x54445743,  // "CWDT"
            1,           // Version
            x, y, width, height
        };

        DWORD bytesWritten;
        WriteFile(hFile, &header, sizeof(header), &bytesWritten, NULL);
        CloseHandle(hFile);

        return true;
    }
};
```

---

## 5. User Interface Design

### 5.1 Main Window Layout (Simplified)

```
┌─────────────────────────────────────────────────┐
│ ConfigWriter                             [─][□][×]│
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌─ Settings ─────────────────────────────────┐ │
│  │                                             │ │
│  │  Username:  [__________________________]    │ │
│  │                                             │ │
│  │  Theme:     (●) Light    ( ) Dark           │ │
│  │                                             │ │
│  └─────────────────────────────────────────────┘ │
│                                                 │
│  ┌─ Log ──────────────────────────────────────┐ │
│  │ [2026-01-31 10:30:00] [INFO] Started       │ │
│  │ [2026-01-31 10:30:01] [INFO] Settings loaded│ │
│  │ [2026-01-31 10:35:00] [ERROR] Save failed  │ │
│  └─────────────────────────────────────────────┘ │
│                                                 │
│  Status: C:\Program Files\ConfigWriter\...      │
│                                                 │
│                [Save Settings]                  │
│                                                 │
└─────────────────────────────────────────────────┘
```

> **Note:** This minimal UI is sufficient for demonstrating the MSIX failure. The Auto-save checkbox, Recent Files panel, and Reset button are optional enhancements.

### 5.2 UI Components

| Component | Type | Properties |
|-----------|------|------------|
| Username | Edit Control | Max length: 50, Single line |
| Theme Light | Radio Button | Group: ThemeGroup |
| Theme Dark | Radio Button | Group: ThemeGroup |
| Log Output | Edit Control | Read-only, multiline, scroll, monospace font |
| Status | Static Text | Shows current settings path |
| Save Settings | Button | Triggers settings persistence |

> **Optional Components:** Auto-save checkbox, Last Opened display, Reset button

### 5.3 Window Properties

| Property | Value |
|----------|-------|
| **Default Width** | 600 pixels |
| **Default Height** | 550 pixels |
| **Minimum Width** | 500 pixels |
| **Minimum Height** | 400 pixels |
| **Resizable** | Yes |
| **Background Color** | System window color |
| **Font** | Segoe UI, 9pt |

### 5.4 Error Dialog

When a save operation fails:

```
┌─────────────────────────────────────────┐
│ ⚠ ConfigWriter - Save Error            │
├─────────────────────────────────────────┤
│                                         │
│  Failed to save settings.               │
│                                         │
│  Error: Access is denied (0x00000005)   │
│  File: C:\Program Files\ConfigWriter\   │
│        settings.ini                     │
│                                         │
│  This may occur when running as MSIX    │
│  without Package Support Framework.     │
│                                         │
│         [Retry]    [Cancel]             │
│                                         │
└─────────────────────────────────────────┘
```

---

## 6. File System Interactions

### 6.1 File Operation Summary

| File | Read | Write | Create | Delete | Required |
|------|------|-------|--------|--------|----------|
| settings.ini | ✓ Startup | ✓ Save | ✓ First run | ✗ | **Yes** |
| app.log | ✓ Display | ✓ Append | ✓ First run | ✗ | **Yes** |
| data.dat | ✓ Startup | ✓ Exit | ✓ First run | ✗ | Optional |

### 6.2 File Access Patterns

#### settings.ini
- **Read:** `GetPrivateProfileString()` at startup and on demand
- **Write:** `WritePrivateProfileString()` on Save button or auto-save
- **Error Handling:** Fall back to defaults, display warning

#### app.log
- **Write:** `fopen()` with append mode (`"a"`)
- **Pattern:** Append-only, never truncate
- **Error Handling:** Silently skip logging if unavailable

#### data.dat (Optional)
- **Read:** `CreateFile()` + `ReadFile()` at startup
- **Write:** `CreateFile()` + `WriteFile()` at shutdown
- **Error Handling:** Use default window position/size
- **Note:** This file is optional. Implement only if demonstrating CreateFile API failure is needed.

### 6.3 Path Determination

All file paths are derived from the executable location:

```cpp
wchar_t exePath[MAX_PATH];
GetModuleFileName(NULL, exePath, MAX_PATH);
PathRemoveFileSpec(exePath);
// exePath now contains: C:\Program Files\ConfigWriter

wchar_t settingsPath[MAX_PATH];
PathCombine(settingsPath, exePath, L"settings.ini");
// settingsPath: C:\Program Files\ConfigWriter\settings.ini
```

This pattern is the **root cause** of MSIX failures—the paths resolve to the protected package installation directory.

---

## 6.4 Implementation Anti-Patterns (Must Avoid)

To ensure the application fails properly in MSIX without PSF, the following patterns **must be avoided**:

| Anti-Pattern | Why It Must Be Avoided |
|--------------|------------------------|
| Fallback to %APPDATA% or %LOCALAPPDATA% | Would make the app work without PSF |
| Using SHGetKnownFolderPath for user directories | Same as above - avoids the failure |
| Try/catch that silently ignores write errors | Hides the failure from demonstration |
| Checking if path is writable before writing | Could implement workarounds |
| Using Windows.Storage APIs | MSIX-aware APIs that would work |
| Embedded database in AppData (SQLite, etc.) | Would work without PSF |

**The application MUST:**
- Always resolve paths relative to the executable location
- Always attempt writes to the installation directory
- Always show clear error dialogs when writes fail
- Never implement fallback paths for write operations

---

## 7. MSIX Packaging Considerations

### 7.1 Why It Fails Without PSF

When packaged as MSIX:

1. **Installation Directory is Read-Only:** The package is installed to `C:\Program Files\WindowsApps\Contoso.ConfigWriter_1.0.0.0_x64__8wekyb3d8bbwe\` which is a protected, read-only location.

2. **Write Operations Blocked:** Any attempt to create or modify files in this directory results in `ERROR_ACCESS_DENIED` (0x00000005).

3. **No VFS Redirection for Writes:** While MSIX provides Virtual File System (VFS) for reads, write operations to the package directory are explicitly blocked.

### 7.2 Expected Failures in MSIX (No PSF)

| Operation | Win32 Result | API Return | Error Code |
|-----------|--------------|------------|------------|
| `WritePrivateProfileString()` | Fails | FALSE | 5 (ACCESS_DENIED) |
| `fopen("app.log", "a")` | Fails | NULL | errno=13 (EACCES) |
| `CreateFile(GENERIC_WRITE)` | Fails | INVALID_HANDLE_VALUE | 5 |

### 7.3 Application Manifest (AppxManifest.xml)

```xml
<?xml version="1.0" encoding="utf-8"?>
<Package xmlns="http://schemas.microsoft.com/appx/manifest/foundation/windows10"
         xmlns:uap="http://schemas.microsoft.com/appx/manifest/uap/windows10"
         xmlns:rescap="http://schemas.microsoft.com/appx/manifest/foundation/windows10/restrictedcapabilities"
         IgnorableNamespaces="rescap">

  <Identity Name="Contoso.ConfigWriter"
            Publisher="CN=Contoso Software, O=Contoso, C=US"
            Version="1.0.0.0"
            ProcessorArchitecture="x64" />

  <Properties>
    <DisplayName>ConfigWriter</DisplayName>
    <PublisherDisplayName>Contoso Software</PublisherDisplayName>
    <Description>Settings management demonstration application</Description>
    <Logo>Assets\StoreLogo.png</Logo>
  </Properties>

  <Dependencies>
    <TargetDeviceFamily Name="Windows.Desktop"
                        MinVersion="10.0.17763.0"
                        MaxVersionTested="10.0.22621.0" />
  </Dependencies>

  <Resources>
    <Resource Language="en-us" />
  </Resources>

  <Applications>
    <Application Id="ConfigWriter"
                 Executable="ConfigWriter.exe"
                 EntryPoint="Windows.FullTrustApplication">
      <uap:VisualElements DisplayName="ConfigWriter"
                          Description="Settings management application"
                          BackgroundColor="transparent"
                          Square150x150Logo="Assets\Square150x150Logo.png"
                          Square44x44Logo="Assets\Square44x44Logo.png">
        <uap:DefaultTile Wide310x150Logo="Assets\Wide310x150Logo.png" />
      </uap:VisualElements>
    </Application>
  </Applications>

  <Capabilities>
    <rescap:Capability Name="runFullTrust" />
  </Capabilities>

</Package>
```

---

## 8. PSF Configuration

### 8.1 Required Files

Add these files to the package root:

| File | Purpose |
|------|---------|
| PsfLauncher64.exe | PSF launcher (replaces direct exe launch) |
| PsfRuntime64.dll | PSF runtime core |
| FileRedirectionFixup64.dll | File redirection fixup DLL |
| config.json | PSF configuration |

### 8.2 PSF config.json

```json
{
  "$schema": "https://raw.githubusercontent.com/microsoft/MSIX-PackageSupportFramework/main/schemas/config-schema.json",
  "applications": [
    {
      "id": "ConfigWriter",
      "executable": "ConfigWriter.exe"
    }
  ],
  "processes": [
    {
      "executable": "ConfigWriter",
      "fixups": [
        {
          "dll": "FileRedirectionFixup64.dll",
          "config": {
            "redirectedPaths": {
              "packageRelative": [
                {
                  "base": "",
                  "patterns": [
                    ".*\\.ini$",
                    ".*\\.log$",
                    ".*\\.dat$"
                  ]
                }
              ]
            }
          }
        }
      ]
    }
  ]
}
```

### 8.3 Configuration Explanation

| Property | Value | Description |
|----------|-------|-------------|
| `id` | "ConfigWriter" | Matches Application Id in AppxManifest.xml |
| `executable` | "ConfigWriter.exe" | Actual application executable |
| `dll` | "FileRedirectionFixup64.dll" | 64-bit file redirection fixup |
| `base` | "" | Redirect from package root |
| `patterns` | Array | Regex patterns for files to redirect |

### 8.4 Pattern Matching Details

| Pattern | Matches | Examples |
|---------|---------|----------|
| `.*\\.ini$` | All .ini files | settings.ini, defaults.ini |
| `.*\\.log$` | All .log files | app.log, debug.log |
| `.*\\.dat$` | All .dat files | data.dat, cache.dat |

### 8.5 Redirection Target

Files are redirected to:
```
%LOCALAPPDATA%\Packages\Contoso.ConfigWriter_8wekyb3d8bbwe\LocalCache\Local\VFS\
```

The application sees the original path, but writes go to the writable location.

### 8.6 Updated AppxManifest.xml

Update the Application element to use PsfLauncher:

```xml
<Application Id="ConfigWriter"
             Executable="PsfLauncher64.exe"
             EntryPoint="Windows.FullTrustApplication">
```

### 8.7 Package Directory Structure (With PSF)

```
Contoso.ConfigWriter_1.0.0.0_x64/
├── AppxManifest.xml
├── config.json                    # PSF configuration
├── PsfLauncher64.exe             # New entry point
├── PsfRuntime64.dll              # PSF runtime
├── FileRedirectionFixup64.dll    # File redirection DLL
├── ConfigWriter.exe              # Original application
├── defaults.ini                  # Read-only defaults
├── Assets/
│   ├── Square44x44Logo.png
│   ├── Square150x150Logo.png
│   └── StoreLogo.png
└── resources/
    └── app.ico
```

---

## 9. Testing Strategy

### 9.1 Test Phases

| Phase | Environment | Expected Result |
|-------|-------------|-----------------|
| 1 | Win32 (unpackaged) | All operations succeed |
| 2 | MSIX (no PSF) | Write operations fail |
| 3 | MSIX (with PSF) | All operations succeed (via redirection) |

### 9.2 Test Cases

#### TC-1: Settings Persistence (Win32 Baseline)

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Launch application | Settings loaded (or defaults created) |
| 2 | Modify username to "TestUser" | UI updates |
| 3 | Click "Save Settings" | Save succeeds, no error |
| 4 | Close and reopen application | Username shows "TestUser" |

#### TC-2: Settings Persistence (MSIX No PSF)

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Launch application | Settings loaded from package |
| 2 | Modify username to "TestUser" | UI updates |
| 3 | Click "Save Settings" | **Error dialog: Access Denied** |
| 4 | Close and reopen application | Username shows "DefaultUser" (unchanged) |

#### TC-3: Settings Persistence (MSIX With PSF)

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Launch application | Settings loaded |
| 2 | Modify username to "TestUser" | UI updates |
| 3 | Click "Save Settings" | Save succeeds, no error |
| 4 | Close and reopen application | Username shows "TestUser" |
| 5 | Verify redirect location | File exists in LocalCache |

#### TC-4: Log File Creation

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Launch application | Log file created (or appended) |
| 2 | Verify log content | Contains startup entry |
| 3 | Perform settings save | Log contains save entry |

#### TC-5: Window State Persistence

| Step | Action | Expected Result |
|------|--------|-----------------|
| 1 | Launch application | Window appears at default position |
| 2 | Move and resize window | Window moved |
| 3 | Close application | data.dat updated |
| 4 | Reopen application | Window at previous position |

### 9.3 Verification Commands

Verify redirected files exist:

```powershell
# Get package family name
$package = Get-AppxPackage -Name "Contoso.ConfigWriter"
$localCache = "$env:LOCALAPPDATA\Packages\$($package.PackageFamilyName)\LocalCache"

# Check for redirected files
Get-ChildItem -Path "$localCache\Local\VFS" -Recurse
```

### 9.4 Troubleshooting

| Symptom | Possible Cause | Resolution |
|---------|----------------|------------|
| PSF not loading | Missing PsfRuntime64.dll | Verify DLL in package |
| Fixup not applied | Regex pattern mismatch | Check pattern escaping |
| Files not redirecting | Wrong `base` path | Verify package-relative path |
| Access denied persists | Fixup DLL mismatch (32/64) | Use correct architecture |

---

## 10. Build Instructions

### 10.1 Visual Studio Project Setup

1. Create new **Windows Desktop Application** project
2. Configure project properties:
   - **Configuration:** Release
   - **Platform:** x64
   - **Character Set:** Unicode
   - **Runtime Library:** /MD (for VC++ Redist) or /MT (static)

### 10.2 Build Commands

```batch
:: Build from command line
msbuild ConfigWriter.sln /p:Configuration=Release /p:Platform=x64

:: Output location
dir x64\Release\ConfigWriter.exe
```

### 10.3 MSIX Packaging (Without PSF)

Using MSIX Packaging Tool or manual packaging:

```powershell
# Create package layout
$layout = "C:\Packaging\ConfigWriter_NoFsf"
New-Item -ItemType Directory -Path $layout -Force

# Copy application files
Copy-Item "x64\Release\ConfigWriter.exe" $layout
Copy-Item "defaults.ini" $layout
Copy-Item "Assets\*" "$layout\Assets\" -Recurse

# Copy manifest
Copy-Item "AppxManifest.xml" $layout

# Create package
MakeAppx.exe pack /d $layout /p "ConfigWriter_NoFsf.msix"

# Sign package (with test certificate)
SignTool.exe sign /fd SHA256 /a /f "TestCert.pfx" /p "password" "ConfigWriter_NoFsf.msix"
```

### 10.4 MSIX Packaging (With PSF)

```powershell
# Create package layout with PSF
$layout = "C:\Packaging\ConfigWriter_WithFsf"
New-Item -ItemType Directory -Path $layout -Force

# Copy application files
Copy-Item "x64\Release\ConfigWriter.exe" $layout
Copy-Item "defaults.ini" $layout
Copy-Item "Assets\*" "$layout\Assets\" -Recurse

# Copy PSF files (from PSF release)
$psfPath = "C:\PSF\bin\x64"
Copy-Item "$psfPath\PsfLauncher64.exe" $layout
Copy-Item "$psfPath\PsfRuntime64.dll" $layout
Copy-Item "$psfPath\FileRedirectionFixup64.dll" $layout

# Copy PSF config
Copy-Item "config.json" $layout

# Copy modified manifest (with PsfLauncher as entry point)
Copy-Item "AppxManifest_PSF.xml" "$layout\AppxManifest.xml"

# Create package
MakeAppx.exe pack /d $layout /p "ConfigWriter_WithFsf.msix"

# Sign package
SignTool.exe sign /fd SHA256 /a /f "TestCert.pfx" /p "password" "ConfigWriter_WithFsf.msix"
```

### 10.5 Quick Test Installation

```powershell
# Install test package
Add-AppxPackage -Path "ConfigWriter_WithFsf.msix"

# Launch application
Start-Process "shell:AppsFolder\Contoso.ConfigWriter_8wekyb3d8bbwe!ConfigWriter"

# Uninstall when done
Get-AppxPackage "Contoso.ConfigWriter" | Remove-AppxPackage
```

---

## 10.6 Implementation Priority

If development time is limited, implement features in this order:

| Priority | Feature | Purpose |
|----------|---------|---------|
| 1 | Core executable with main window | Foundation |
| 2 | Path resolution using GetModuleFileName | Root cause of MSIX failure |
| 3 | settings.ini read/write | Primary failure demonstration |
| 4 | Save button with error dialog | Visible failure in MSIX |
| 5 | app.log writing | Secondary failure demonstration |
| 6 | Log display in UI | Shows what happened |
| 7 | data.dat (optional) | Third API demonstration |

**Minimum viable demo:** Priorities 1-4 are sufficient to demonstrate the PSF need.

---

## Appendix A: Complete config.json Reference

```json
{
  "$schema": "https://raw.githubusercontent.com/microsoft/MSIX-PackageSupportFramework/main/schemas/config-schema.json",

  "applications": [
    {
      "id": "ConfigWriter",
      "executable": "ConfigWriter.exe",
      "workingDirectory": ""
    }
  ],

  "processes": [
    {
      "executable": "ConfigWriter",
      "fixups": [
        {
          "dll": "FileRedirectionFixup64.dll",
          "config": {
            "redirectedPaths": {
              "packageRelative": [
                {
                  "base": "",
                  "patterns": [
                    ".*\\.ini$",
                    ".*\\.log$",
                    ".*\\.dat$"
                  ]
                }
              ],
              "packageDriveRelative": [],
              "knownFolders": []
            }
          }
        }
      ]
    }
  ]
}
```

---

## Appendix B: Troubleshooting Guide

### B.1 Common Issues

| Issue | Symptom | Solution |
|-------|---------|----------|
| PSF not loading | App crashes on start | Check PsfLauncher is entry point |
| Wrong architecture | DLL load failure | Match 64-bit app with *64.dll files |
| Pattern not matching | Files still blocked | Escape backslashes: `\\\\` |
| Config syntax error | PSF ignored | Validate JSON syntax |

### B.2 Diagnostic Tools

```powershell
# Enable PSF trace logging
# Add to config.json processes section:
{
  "executable": "ConfigWriter",
  "fixups": [
    {
      "dll": "TraceFixup64.dll",
      "config": {
        "traceMethod": "outputDebugString"
      }
    },
    {
      "dll": "FileRedirectionFixup64.dll",
      "config": { ... }
    }
  ]
}

# View trace output
# Use DebugView or attach debugger
```

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0.0 | 2026-01-31 | Generated | Initial specification |

---

*End of Document*
