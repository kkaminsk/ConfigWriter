# ConfigWriter Implementation Recommendations

## Overview

This document provides implementation recommendations for ConfigWriter, a demonstration application designed to **fail** when packaged as MSIX without the Package Support Framework (PSF). The goal is simplicity—the application should clearly demonstrate the file redirection problem without unnecessary complexity.

---

## 1. Implementation Language Recommendation

**Recommended: C++ Win32**

| Option | Pros | Cons |
|--------|------|------|
| C++ Win32 | Direct API calls, no runtime dependencies, clearer failure demonstration | More verbose code |
| C# .NET Framework 4.8 | Faster development, simpler UI code | Requires .NET Framework, abstracts some file operations |

C++ is preferred because:
- Uses raw Win32 APIs (`WritePrivateProfileString`, `CreateFile`) that produce clear error codes
- No runtime abstraction layers that might obscure the ACCESS_DENIED behavior
- Smaller deployment footprint for the demo

---

## 2. Critical Implementation Requirements

### 2.1 Path Resolution (Must Cause Failure)

The application **must** resolve file paths relative to the executable location:

```cpp
wchar_t exePath[MAX_PATH];
GetModuleFileNameW(NULL, exePath, MAX_PATH);
PathRemoveFileSpecW(exePath);
// All file operations use this directory
```

**Do NOT:**
- Use `%APPDATA%` or `%LOCALAPPDATA%` (these work in MSIX)
- Use `SHGetKnownFolderPath` for user directories
- Implement any fallback to writable locations

### 2.2 File Operations That Must Fail

Implement all three file types to demonstrate different Win32 APIs failing:

| File | API | Error in MSIX |
|------|-----|---------------|
| settings.ini | `WritePrivateProfileStringW()` | Returns FALSE, GetLastError() = 5 |
| app.log | `fopen()` or `_wfopen()` | Returns NULL, errno = 13 (EACCES) |
| data.dat | `CreateFileW(GENERIC_WRITE)` | Returns INVALID_HANDLE_VALUE, GetLastError() = 5 |

### 2.3 Visible Error Handling

When writes fail, the application must:
1. Display a clear error dialog showing the error code
2. Show the full path that failed
3. Continue running (don't crash)
4. Log the failure if possible

This visibility is essential for demonstrating the problem.

---

## 3. Simplified UI Recommendations

### 3.1 Minimal UI Approach

The specification describes a full settings manager UI. For a focused demo, simplify to:

```
┌─────────────────────────────────────────────────┐
│ ConfigWriter Demo                        [─][×] │
├─────────────────────────────────────────────────┤
│                                                 │
│  Username: [____________________________]       │
│                                                 │
│  Theme:  (●) Light  ( ) Dark                    │
│                                                 │
│  [Save Settings]                                │
│                                                 │
│  ┌─ Log ──────────────────────────────────────┐ │
│  │ [2026-01-31 10:30:00] [INFO] Started       │ │
│  │ [2026-01-31 10:30:01] [ERROR] Save failed  │ │
│  └────────────────────────────────────────────┘ │
│                                                 │
│  Status: Settings path: C:\Program Files\...   │
│                                                 │
└─────────────────────────────────────────────────┘
```

### 3.2 UI Components to Keep

| Component | Purpose |
|-----------|---------|
| Username text field | Simple string to save |
| Theme radio buttons | Demonstrates INI write |
| Save button | Triggers visible failure in MSIX |
| Log display | Shows error messages |
| Status bar | Shows the path being written to |

### 3.3 UI Components to Omit (Optional)

These can be skipped for a minimal demo:
- Auto-save checkbox (just always save on exit)
- Reset button
- Recent files list
- Window size/position persistence (keep data.dat simple)

---

## 4. File Implementation Details

### 4.1 settings.ini - Keep Simple

```ini
[User]
Username=DefaultUser
Theme=Light
```

Skip the `[Behavior]` and `[Window]` sections unless needed. Two values are sufficient to demonstrate the problem.

### 4.2 app.log - Essential for Demo

The log file is important because it:
- Demonstrates append operations failing
- Shows timestamps of each failure
- Provides a visible record in the UI

Minimum log entries:
```
[timestamp] [INFO] Application started
[timestamp] [INFO] Settings loaded from: <path>
[timestamp] [ERROR] Failed to save settings: Access denied (5)
```

### 4.3 data.dat - Optional Simplification

If implementing:
- Store only window position (X, Y, Width, Height)
- Skip the recent files list
- Binary format is fine but not required—could use simple text

If time-constrained, this file can be omitted entirely. The settings.ini and app.log are sufficient to demonstrate PSF need.

---

## 5. Error Dialog Recommendations

When a write fails, show a dialog like:

```
┌─────────────────────────────────────────┐
│ ⚠ Save Failed                          │
├─────────────────────────────────────────┤
│                                         │
│  Could not save settings.               │
│                                         │
│  Error Code: 5 (ACCESS_DENIED)          │
│  Path: C:\Program Files\WindowsApps\    │
│        Contoso.ConfigWriter_1.0.0.0...  │
│        \settings.ini                    │
│                                         │
│  This error occurs when the app is      │
│  packaged as MSIX without PSF.          │
│                                         │
│              [OK]                       │
└─────────────────────────────────────────┘
```

Key elements:
- Numeric error code (5)
- Error name (ACCESS_DENIED)
- Full file path
- Brief explanation

---

## 6. What NOT to Implement

To ensure the app fails properly in MSIX, avoid:

| Anti-Pattern | Why to Avoid |
|--------------|--------------|
| Fallback to AppData | Would make app work without PSF |
| Try/catch that silently ignores errors | Hides the failure |
| Checking if path is writable first | Could implement workaround |
| Using Windows.Storage APIs | MSIX-aware, would work |
| Embedded database (SQLite in AppData) | Would work without PSF |

---

## 7. Testing Checklist

### 7.1 Win32 (Unpackaged) - Should Pass

- [ ] Application starts and creates settings.ini if missing
- [ ] Modifying and saving settings works
- [ ] app.log is created and appended to
- [ ] data.dat stores window state (if implemented)
- [ ] No error dialogs appear

### 7.2 MSIX (No PSF) - Should Fail

- [ ] Application starts (reads work)
- [ ] Settings load from package
- [ ] Clicking Save shows ACCESS_DENIED error (code 5)
- [ ] Log file write fails (may be silent or logged to UI)
- [ ] Application does NOT crash
- [ ] Changes are lost on restart

### 7.3 MSIX (With PSF) - Should Pass

- [ ] Application starts normally
- [ ] Settings save without error
- [ ] Changes persist after restart
- [ ] Verify files exist in LocalCache\Local\VFS

---

## 8. Build and Package Recommendations

### 8.1 Project Configuration

```
Configuration: Release
Platform: x64
Runtime Library: /MT (static) - avoids VC++ Redist dependency
Subsystem: Windows
Character Set: Unicode
```

Static linking (/MT) simplifies deployment—no need to bundle VC++ runtime DLLs.

### 8.2 Package Structure (Without PSF)

```
Package/
├── AppxManifest.xml
├── ConfigWriter.exe
├── defaults.ini        # Optional: read-only defaults
└── Assets/
    ├── Square44x44Logo.png
    ├── Square150x150Logo.png
    └── StoreLogo.png
```

### 8.3 Package Structure (With PSF)

```
Package/
├── AppxManifest.xml    # Entry point: PsfLauncher64.exe
├── config.json         # PSF configuration
├── PsfLauncher64.exe
├── PsfRuntime64.dll
├── FileRedirectionFixup64.dll
├── ConfigWriter.exe
├── defaults.ini
└── Assets/
    └── ...
```

---

## 9. PSF Configuration Notes

The config.json in the specification is correct. Key points:

```json
{
  "patterns": [
    ".*\\.ini$",
    ".*\\.log$",
    ".*\\.dat$"
  ]
}
```

- Patterns use regex (not glob)
- Backslashes must be escaped: `\\.` matches literal `.`
- `base: ""` means package root directory
- Files redirect to `%LOCALAPPDATA%\Packages\<PFN>\LocalCache\Local\VFS\`

---

## 10. Development Priority

If time is limited, implement in this order:

1. **Core executable** - Creates main window, resolves paths
2. **settings.ini read/write** - Primary failure demonstration
3. **Save button with error dialog** - Visible failure
4. **app.log writing** - Secondary failure demonstration
5. **Log display in UI** - Shows what happened
6. **data.dat** - Optional, adds third API demonstration

The first four items are sufficient for a complete demo.

---

## Summary

| Requirement | Recommendation |
|-------------|----------------|
| Language | C++ Win32 |
| Path resolution | GetModuleFileName + PathRemoveFileSpec |
| Primary failure | WritePrivateProfileString to settings.ini |
| Secondary failure | fopen to app.log |
| Error visibility | MessageBox with error code and path |
| UI complexity | Minimal - username, theme, save button, log view |
| Fallbacks | None - must fail in MSIX |

The application succeeds when it fails clearly and visibly in MSIX without PSF, then works seamlessly when PSF is added.
