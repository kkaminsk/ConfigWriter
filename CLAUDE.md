# ConfigWriter

A Win32 demonstration application designed to showcase file write failures when legacy applications are packaged as MSIX without the Package Support Framework (PSF).

## Status

**Specification complete, implementation pending.** This project uses an OpenSpec artifact-driven workflow. No source code exists yet—see Implementation Phases below.

## Purpose

ConfigWriter writes configuration files, logs, and data files to its installation directory—a common pattern in legacy Win32 applications that violates the MSIX security model. It demonstrates the need for PSF's FileRedirectionFixup by **failing visibly** when packaged as MSIX without PSF.

## Documentation

- **1-ConfigWriter.md** - Complete specification (1000+ lines)
- **recommendations.md** - Implementation guidance and priorities

## Technology Stack

- **Language:** C++ Win32 (recommended for clearer failure demonstration)
- **Architecture:** x64
- **IDE:** Visual Studio 2022 (v143 toolset)
- **Windows SDK:** 10.0.22621.0+
- **C++ Standard:** C++17
- **Character Set:** Unicode
- **Runtime Library:** /MT (static linking)

## Project Structure

```
ConfigWriter/
├── CLAUDE.md                 # This file
├── README.md                 # Project intro
├── LICENSE                   # MIT License
├── 1-ConfigWriter.md         # Full specification
├── recommendations.md        # Implementation guidance
├── openspec/                 # Artifact-driven workflow
│   ├── config.yaml
│   └── changes/              # Implementation phases (a-g)
└── .claude/                  # Claude Code configuration
    ├── settings.local.json
    ├── commands/opsx/        # Workflow command docs
    └── skills/               # OpenSpec skills
```

## Implementation Phases

Use `/opsx:apply` to implement each phase in order:

| Phase | Directory | Purpose |
|-------|-----------|---------|
| 1 | `a-core-window` | Main window with UI controls |
| 2 | `b-path-resolution` | Path resolution via GetModuleFileName |
| 3 | `c-settings-ini` | Settings file read/write |
| 4 | `d-save-with-error-dialog` | Save button with error dialog |
| 5 | `e-app-log` | Logging system |
| 6 | `f-log-display-ui` | Log display in main window |
| 7 | `g-data-dat-optional` | Optional binary data file |

## Critical Requirements

### Path Resolution (Must Cause Failure)

```cpp
wchar_t exePath[MAX_PATH];
GetModuleFileNameW(NULL, exePath, MAX_PATH);
PathRemoveFileSpecW(exePath);
// All file operations use this directory (read-only in MSIX)
```

### Do NOT Implement

- Fallback to `%APPDATA%` or `%LOCALAPPDATA%`
- Silent error suppression
- Writable path checks
- Windows.Storage APIs (MSIX-aware)

## Runtime Files (Created by App)

| File | Format | API |
|------|--------|-----|
| settings.ini | INI (`[User]` section) | `WritePrivateProfileStringW()` |
| app.log | Text log | `fopen()` / `CreateFileW()` |
| data.dat | Binary (magic: "CWDT") | `CreateFileW()` / `WriteFile()` |

## MSIX Behavior

| Operation | Win32 | MSIX (No PSF) | MSIX (With PSF) |
|-----------|-------|---------------|-----------------|
| Read files | Works | Works | Works |
| Write files | Works | **ACCESS_DENIED (5)** | Works (redirected) |

## PSF Configuration

Files matching `*.ini`, `*.log`, `*.dat` redirect to:
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

## Requirements

- Windows 10 1809 (Build 17763)+
- x64 architecture
