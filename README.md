# apps

Public builds of my apps. No source code here, just the files you download and run.

---

## Popeye

A Windows desktop app for browsing Kubernetes clusters: pods, jobs, cron jobs, events, live pod logs, describe and environment variables.

### Install

PowerShell, no admin needed:

```powershell
irm https://github.com/zeelancer/apps/releases/download/popeye-latest/install.ps1 | iex
```

It downloads the newest build, unpacks it to `%LocalAppData%\Popeye\app`, and makes Desktop and Start Menu shortcuts.

After that **Popeye updates itself**: open it, and when there is a newer build a bar appears at the top of the window with an `Update now` button. It downloads the new files, closes, copies them in and starts again. There is no installer and nothing is written to the registry.

### Options

| Option | What it does |
| --- | --- |
| `-Path D:\Apps\Popeye` | install somewhere else |
| `-NoShortcuts` | do not make the shortcuts |
| `-NoStart` | do not launch it afterwards |

To pass one, download the script first:

```powershell
irm https://github.com/zeelancer/apps/releases/download/popeye-latest/install.ps1 -OutFile install.ps1
.\install.ps1 -Path D:\Apps\Popeye
```

### If running remote scripts is blocked

Some locked-down machines refuse `irm | iex`. Paste this instead; it does the same thing.

```powershell
$dst = "$env:LOCALAPPDATA\Popeye\app"
$zip = "$env:TEMP\popeye.zip"

Get-Process Popeye -ErrorAction SilentlyContinue | Stop-Process -Force
New-Item -ItemType Directory -Force $dst | Out-Null

$ProgressPreference = 'SilentlyContinue'
Invoke-WebRequest 'https://github.com/zeelancer/apps/releases/download/popeye-latest/Popeye-win-x64.zip' -OutFile $zip -UseBasicParsing
Unblock-File $zip
Expand-Archive $zip -DestinationPath $dst -Force
Remove-Item $zip -Force

$shell = New-Object -ComObject WScript.Shell
@(
  (Join-Path ([Environment]::GetFolderPath('Desktop')) 'Popeye.lnk'),
  (Join-Path "$env:APPDATA\Microsoft\Windows\Start Menu\Programs" 'Popeye.lnk')
) | ForEach-Object {
    $link = $shell.CreateShortcut($_)
    $link.TargetPath = "$dst\Popeye.exe"
    $link.WorkingDirectory = $dst
    $link.IconLocation = "$dst\Popeye.exe"
    $link.Save()
}

Start-Process "$dst\Popeye.exe" -WorkingDirectory $dst
```

### Things worth knowing

- **Needs the [.NET 10 Desktop Runtime](https://dotnet.microsoft.com/download/dotnet/10.0).** Popeye will not start without it, and Windows will offer you the download link.
- **A shortcut must have its "Start in" set to the app folder.** Popeye reads `appsettings.json` from the working directory, not from the folder the exe is in. Started from anywhere else it opens with no database and no logging, and looks broken. The install script sets this for you.
- **Keep nothing else in the app folder.** An update mirrors that folder, so anything not in the build is deleted. Your settings, database and logs live elsewhere (`%LocalAppData%\Popeye` and `%ProgramData%\Popeye`), so they survive updates and uninstalls.
- **To uninstall**, delete the app folder and the two shortcuts. Add `%LocalAppData%\Popeye` and `%ProgramData%\Popeye` to remove your settings, database and logs too.

### What is in the release

| File | What it is |
| --- | --- |
| `Popeye-win-x64.zip` | the build itself |
| `latest.json` | version, download url and SHA-256, which Popeye reads to spot a new version |
| `install.ps1` | the install script above |

The `Source code` links GitHub adds to every release are this repository's own contents, not Popeye's source.

---

## How releases here work

One rolling release per app, under a fixed tag, with its assets replaced on every build. So the download links never change:

```
https://github.com/zeelancer/apps/releases/download/<app>-latest/<file>
```

GitHub's own `releases/latest/...` shortcut is per **repository**, not per app, so it cannot be used once this repo holds more than one.
