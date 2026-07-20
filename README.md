# MSIinfo.com — MSI Property Viewer

🌐 Webapp: <https://msiinfo.com>

A browser-based tool that reads Windows Installer (`.msi`) packages and shows their **Property table** — ProductCode, UpgradeCode, ProductVersion, Manufacturer — plus Summary Information and ready-to-use deployment command lines.

Drag & drop an MSI. No upload, no install, no dependencies.

> The modern replacement for the classic "open PowerShell, spin up a `WindowsInstaller.Installer` COM object" routine — just to get a ProductCode.

## Why

Getting a ProductCode out of an MSI normally requires PowerShell + COM interop, Orca, or a Windows machine. Intune admins need it constantly — for detection rules, uninstall command lines, and supersedence. This tool does it in any browser, on any OS, in under a second.

## Features

- 🔑 **Key properties at a glance** — ProductCode (copy-ready), ProductName, ProductVersion, Manufacturer, UpgradeCode, ProductLanguage with LCID name
- 🎯 **Install scope analysis** — interprets `ALLUSERS` / `MSIINSTALLPERUSER` correctly: per-machine, per-user, privilege-dependent, or dual-mode — with an Intune deployment hint
- 📋 **Command lines & Intune detection** — silent install/uninstall, registry detection path (`HKLM\...\Uninstall\{ProductCode}`), PowerShell verification — all one click to copy
- 📄 **Full Property table** — filterable, exportable as JSON or CSV
- ℹ️ **Summary Information** — Package Code, minimum installer version, compressed-source flag, platform/languages, timestamps
- ⚡ **Handles multi-GB files** — the file is read lazily via `File.slice()`; only the sectors needed (header, FAT, directory, Property table) are touched. A 2 GB installer typically costs under 10 MB of RAM and never loads the embedded cab
- 🌗 **Light / auto / dark theme** — remembers your choice, follows the OS in auto
- 🔒 **100% local** — no data leaves the browser; works offline and from `file://`

## How it works

MSI files are OLE Compound File Binary (MS-CFB) containers. The tool parses the format directly in JavaScript:

1. Reads the CFB header, DIFAT/FAT, and directory entries
2. Decodes the MSI-specific stream-name encoding (UTF "base64" packing, `0x3800`–`0x4840` range)
3. Extracts `!_StringPool` / `!_StringData` (with codepage-aware decoding, incl. large string pools with 3-byte refs and >64 KB strings)
4. Reads the column-major `!Property` table and resolves string references
5. Parses the `SummaryInformation` OLE property set (LPSTR, FILETIME, I2/I4)

All reads go through a lazy source backed by `File.slice()`, so only required byte ranges are pulled off disk.

## Usage

Open the site in any modern browser (Chrome, Edge, Firefox, Safari) and drop an `.msi` on it. That's it — no build step, no server.

To self-host: serve the single HTML file from any static host.

## Limitations

- `.mst` transforms and most `.msp` patches don't carry a Property table and will be rejected with a clear message
- Reads only — this is a viewer, not an editor
- Exotic codepages fall back to Windows-1252 if the browser's `TextDecoder` doesn't support them

## Privacy

All processing happens locally in your browser. No file contents, filenames, or telemetry are sent to any server.

## License

Provided as-is. Test before relying on it in production workflows.
