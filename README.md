# 1Tsu Uninstaller

1Tsu Uninstaller is a Windows desktop utility (WPF) by **KenpSoft** for viewing, managing, and removing installed applications. It reads the standard Windows uninstall registry keys and presents your installed programs in a clean, sortable UI with advanced removal options.

> **Warning**  
> The app is designed for advanced users. The **Force Remove** feature can permanently delete files and registry entries. Use it carefully and at your own risk.

---

## Features

- **Modern WPF interface** with banner, summary bar, and sortable grid
- **Full installed apps list** (from `HKLM` and `HKCU` uninstall registry keys)
- **Search and filter**
  - Search by **name** or **publisher** (live filtering)
  - Toggle between **All apps** and **Microsoft apps only**
- **Standard uninstall**
  - Runs the app’s normal uninstall command (`UninstallString` via `cmd.exe`)
- **Force Remove** (advanced)
  - Attempts to kill running processes belonging to the app’s install folder
  - Deletes the main install directory (with read-only attributes cleared)
  - Removes the app’s uninstall **registry entry**
  - Cleans leftover folders under `%APPDATA%` and `%LOCALAPPDATA%` that match the app name
- **Context menu tools** on apps list:
  - **Uninstall** / **Force Remove**
  - **Open Install Folder**
  - **Copy Install Location**
  - **Copy Uninstall String**
  - **Open Registry Key** in `regedit` (last key is set so it opens at the app’s entry)
  - **Search on Google** (e.g. "AppName uninstall")
  - **Program Website** (uses `URLInfoAbout` / `URLUpdateInfo` from registry)
  - **Properties** window with detailed info
- **Summary bar** showing:
  - Total apps
  - How many are currently visible after filtering
  - Estimated total size of visible apps (MB/GB)
- **Global error handling** with friendly message if the app crashes

---

## Requirements

- **Operating System:** Windows 10/11 (x64)
- **Runtime:** .NET is bundled when using the self-contained build (no separate .NET install required)
- **Privileges:**
  - Standard uninstall often works with normal user rights
  - **Force Remove** and registry operations may require **Run as Administrator**

> The application manifest requests **administrator privileges** (`requireAdministrator`), so you will see a UAC prompt on launch.

---

## Installation & Portable .EXE

1Tsu Uninstaller is built as a **self-contained**, **single-file** executable for Windows x64.

Typical publish command used during development:

```powershell
dotnet publish "OneTsuUninstaller.csproj" -c Release -r win-x64 --self-contained true `
  /p:PublishSingleFile=true `
  /p:IncludeNativeLibrariesForSelfExtract=true
```

The resulting portable binary is located at:

```text
bin\Release\net8.0-windows\win-x64\publish\OneTsuUninstaller.exe
```

You can copy this `.exe` to any compatible Windows machine and run it directly.

> If you see permission errors when removing apps or deleting folders, right-click the EXE and choose **Run as administrator**.

---

## Usage

### Main window

- **Apps list**: shows all detected installed applications with columns:
  - **Name** (`DisplayName`)
  - **Publisher**
  - **Date Installed**
  - **Size** (from `EstimatedSize` in KB → formatted to KB/MB)
- **Sorting**: click a column header to sort; Shift+Click allows multi-column sort. The last sort is remembered between refreshes.
- **Search box** (top right):
  - Type to filter by name or publisher
  - Placeholder text: `type to find a program`

### Menus

- **File → Exit**: closes the application
- **Actions → Uninstall**: runs the standard uninstall for the selected app
- **Actions → Force Remove**: uses the aggressive removal logic (see below)
- **View → All apps**: shows all entries
- **View → Microsoft apps only**: filters the list to items whose publisher contains "Microsoft"
- **Help → About**: shows version and copyright

### Context menu on an app

Right-click an app row to open the context menu:

- **Uninstall** – same as Actions → Uninstall
- **Force Remove** – same as Actions → Force Remove
- **Open Install Folder** – opens the app’s installation directory in Explorer
- **Copy Install Location** – copies the folder path to clipboard
- **Copy Uninstall String** – copies the raw uninstall command
- **Open Registry Key** – opens `regedit` and navigates to the uninstall entry
- **Search on Google** – opens a browser search like `"<AppName> uninstall"`
- **Program Website** – opens the product’s homepage/update URL (if present)
- **Properties** – opens a detail window with name, publisher, install location, uninstall string, date, and size

### Force Remove (how it works)

When you select **Force Remove**:

1. **Kill related processes**
   - Checks running processes and looks for executables located under the app’s install folder, then attempts to terminate them.
2. **Delete install folder**
   - Clears read-only attributes on files to avoid `UnauthorizedAccessException`.
   - Attempts to delete the entire folder recursively.
3. **Registry cleanup**
   - Deletes the uninstall registry key under either `HKEY_LOCAL_MACHINE` or `HKEY_CURRENT_USER`, depending on where the app was found.
4. **Leftover folders cleanup**
   - Looks in `%APPDATA%` and `%LOCALAPPDATA%` for top-level folders whose normalized name contains the app name and attempts to delete them.

Because of these operations, running as Administrator is often necessary for full cleanup.

---

## Technical Details

- **Project type:** WPF (`WinExe`)
- **Target framework:** `net8.0-windows`
- **Language:** C#
- **Namespaces:**
  - `OneTsuUninstaller` – main UI and application logic
  - `OneTsuUninstaller.Models` – data model for installed apps
  - `OneTsuUninstaller.Services` – registry reading and force-removal services
- **Registry locations scanned:**
  - `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall`
  - `HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall`
  - `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall`

### Main components

- `MainWindow` – main UI: app list, menu, search, context menu, and summary.
- `InstalledApp` – model representing an installed application (name, publisher, install path, uninstall string, registry path, install date, size, URLs, etc.).
- `InstalledAppsReader` – static service that enumerates installed apps from the registry and builds `InstalledApp` objects.
- `ForceRemover` – static service implementing the Force Remove workflow (kill processes, delete folders, clean registry and leftovers).
- `AppPropertiesWindow` – dialog that shows detailed properties of a selected app.
- `App` – application-level exception handler that shows a friendly error dialog and shuts down on unhandled exceptions.

---

## Error Handling

- Unhandled UI exceptions are caught by `Application_DispatcherUnhandledException`.
- A detailed error dialog is shown, including the exception message and stack trace.
- After showing the dialog, the app exits cleanly.

Per-action errors (e.g., failing to open a folder, website, or registry key) are handled with message boxes that explain what went wrong.

---

## Version & Copyright

- **Version:** 1.0
- **Title/Product:** 1Tsu Uninstaller
- **Company/Author:** KenpSoft
- **Copyright:** © KenpSoft

This information is embedded in the assembly metadata and appears in the EXE file properties (Details tab).
