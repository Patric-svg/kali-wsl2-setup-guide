# Kali Linux on Windows with WSL2

A practical, terminal-first guide to using **Kali Linux inside Windows 10/11 through Windows Subsystem for Linux 2 (WSL2)**.

The goal is to keep Windows as the main desktop environment while using Kali's Linux shell, development tools, networking utilities, Python, Git, Android Debug Bridge (ADB), and `scrcpy` without rebooting into a separate operating system.

> **Important:** WSL2 Kali and a dual-boot Kali installation are separate environments. Packages, settings, users, and files stored inside one are not automatically available in the other. WSL2 is excellent for everyday command-line work, but it does not replace a full Linux installation for every hardware or kernel-level task.

---

## Table of contents

- [1. What this setup provides](#1-what-this-setup-provides)
- [2. Requirements](#2-requirements)
- [3. Install or verify WSL2 and Kali](#3-install-or-verify-wsl2-and-kali)
- [4. Open and close Kali](#4-open-and-close-kali)
- [5. Understand the terminal](#5-understand-the-terminal)
- [6. Access Windows files from Kali](#6-access-windows-files-from-kali)
- [7. Update and maintain Kali](#7-update-and-maintain-kali)
- [8. Install essential terminal tools](#8-install-essential-terminal-tools)
- [9. Python development](#9-python-development)
- [10. Git and GitHub](#10-git-and-github)
- [11. Android debugging with ADB](#11-android-debugging-with-adb)
- [12. Wireless ADB on Android 11 and newer](#12-wireless-adb-on-android-11-and-newer)
- [13. Mirror and control Android with scrcpy](#13-mirror-and-control-android-with-scrcpy)
- [14. Connect USB devices to WSL2 with usbipd-win](#14-connect-usb-devices-to-wsl2-with-usbipd-win)
- [15. Networking and diagnostics](#15-networking-and-diagnostics)
- [16. Terminal productivity](#16-terminal-productivity)
- [17. Suggested workspace layout](#17-suggested-workspace-layout)
- [18. WSL management and backup](#18-wsl-management-and-backup)
- [19. Troubleshooting](#19-troubleshooting)
- [20. What WSL2 can and cannot replace](#20-what-wsl2-can-and-cannot-replace)
- [21. Daily workflow](#21-daily-workflow)
- [22. Security and responsible use](#22-security-and-responsible-use)
- [23. Setup checklist](#23-setup-checklist)
- [24. Official references](#24-official-references)

---

## 1. What this setup provides

This arrangement has two operating environments:

1. **Windows** — the main desktop operating system, applications, browser, and access to Windows files.
2. **Kali Linux on WSL2** — a Linux distribution running in a lightweight managed virtual machine, available from Windows Terminal or PowerShell.

If a computer also has Kali installed through dual boot, that installation remains independent of the WSL2 copy.

### Why use Kali through WSL2?

- Open a Kali Bash terminal without rebooting.
- Use Linux command-line tools while working in Windows.
- Develop with Python, Git, Bash, and common build tools.
- Access Windows files through `/mnt/c`, `/mnt/d`, and other mounted drives.
- Use ADB for Android development and debugging.
- Run graphical Linux applications through WSLg where supported.
- Keep a full dual-boot Kali installation for tasks that need direct hardware access.

### WSL2 is not a full hardware replacement

WSL2 runs Linux with a Microsoft-managed virtualised kernel. Some hardware, kernel modules, wireless features, and low-level interfaces behave differently from a native Kali boot. See [What WSL2 can and cannot replace](#20-what-wsl2-can-and-cannot-replace).

---

## 2. Requirements

Recommended starting point:

- Windows 11, or a supported, updated Windows 10 installation.
- Hardware virtualisation enabled in the computer firmware.
- WSL2 installed and updated.
- Kali Linux installed as a WSL distribution.
- An internet connection for package installation and updates.
- Windows Terminal (recommended, but PowerShell also works).

Check the Windows version by pressing `Win + R`, entering `winver`, and pressing Enter.

---

## 3. Install or verify WSL2 and Kali

> The following commands are run in **Windows PowerShell**, not in the Kali terminal. Use an Administrator PowerShell window for installation or system-level changes.

### 3.1 Install WSL and Kali

On supported Windows versions, the standard installation command is:

```powershell
wsl --install -d kali-linux
```

Restart Windows if the installer requests it. Complete any first-launch setup and create the Linux username and password.

If WSL is already installed, update it:

```powershell
wsl --update
```

Check the WSL version:

```powershell
wsl --version
```

List installed distributions:

```powershell
wsl --list --verbose
```

The Kali distribution should show `2` in the `VERSION` column. If it shows `1`, convert it:

```powershell
wsl --set-version kali-linux 2
```

Set WSL2 as the default for distributions installed in the future:

```powershell
wsl --set-default-version 2
```

### 3.2 If PowerShell cannot find `wsl`

On some systems, the WSL executable may not resolve through the command search path. Call it by its full path:

```powershell
C:\Windows\System32\wsl.exe --version
C:\Windows\System32\wsl.exe --list --verbose
C:\Windows\System32\wsl.exe -d kali-linux
```

If that path is unavailable, check that WSL is installed and that the Windows optional features required by WSL are enabled.

### 3.3 Confirm the Kali distribution

Open Kali:

```powershell
wsl -d kali-linux
```

Inside Kali, check:

```bash
whoami
pwd
cat /etc/os-release
uname -a
```

The first command prints the Linux username. The second prints the current directory. The remaining commands show operating-system and kernel information.

---

## 4. Open and close Kali

### Start Kali from PowerShell

```powershell
wsl -d kali-linux
```

If Kali is the default WSL distribution, this may also work:

```powershell
wsl
```

### Start Kali from Windows Terminal

1. Open Windows Terminal.
2. Open the profile dropdown.
3. Select Kali Linux.
4. A Bash prompt should appear.

You can configure a Windows Terminal profile to launch `wsl.exe -d kali-linux`.

### Exit the Kali shell

Run this **inside Kali**:

```bash
exit
```

This closes the current shell session; it does not uninstall Kali.

### Stop WSL

Run this in **PowerShell**:

```powershell
wsl --shutdown
```

This stops all running WSL distributions and the WSL2 virtual machine. It is useful when applying WSL configuration changes or recovering from a stuck environment.

### List distributions

Run this in PowerShell:

```powershell
wsl --list --verbose
```

---

## 5. Understand the terminal

Commands in this guide are labelled by where they should run:

- **PowerShell** — Windows commands.
- **Kali** — Linux commands inside the Kali shell.
- **Android** — actions performed in the phone's Settings interface.

Do not paste a PowerShell command into Bash or a Bash command into PowerShell unless the guide explicitly says it works in both.

### 5.1 Know where you are

Run these in Kali:

```bash
pwd
whoami
hostname
uname -a
cat /etc/os-release
```

- `pwd` — current directory.
- `whoami` — current Linux user.
- `hostname` — Linux hostname.
- `uname -a` — kernel information.
- `/etc/os-release` — distribution information.

### 5.2 Navigate directories

```bash
cd ~       # go to the Linux home directory
cd ..      # go up one directory
cd /       # go to filesystem root
cd -       # return to the previous directory
```

### 5.3 List files

```bash
ls
ls -lah
ls -a
```

- `ls` lists visible files.
- `ls -lah` shows details, sizes, and hidden files in a readable format.
- `ls -a` includes hidden entries.

### 5.4 Create, copy, move, and delete

```bash
mkdir projects
mkdir -p ~/projects/python
touch notes.txt
cp notes.txt notes-backup.txt
mv notes.txt project-notes.txt
rm project-notes.txt
rmdir empty-folder
```

**Be careful with deletion.** `rm -r` recursively removes directories. `rm -rf` can remove large amounts of data without prompting. Check the path before running either command.

### 5.5 Read and search files

```bash
cat notes.txt
less notes.txt
head notes.txt
tail notes.txt
tail -f application.log
grep "error" application.log
grep -R "search_term" .
```

Press `q` to leave `less` or stop following a log.

### 5.6 Get help

```bash
man ls
ls --help
which python3
command -v adb
apt show adb
```

Press `q` to leave a manual page.

---

## 6. Access Windows files from Kali

WSL normally mounts Windows drives under `/mnt`.

Examples:

```bash
cd /mnt/c
ls /mnt/c/Users
ls /mnt/d
```

A Windows path such as:

```text
C:\Users\YOUR_WINDOWS_USERNAME\Desktop
```

is usually accessible from Kali as:

```bash
/mnt/c/Users/YOUR_WINDOWS_USERNAME/Desktop
```

Replace `YOUR_WINDOWS_USERNAME` with the actual Windows profile folder name. Do not assume that the Windows account display name is the same as the folder name.

### 6.1 Create a shared workspace

For example:

```bash
mkdir -p /mnt/c/Users/YOUR_WINDOWS_USERNAME/Desktop/LinuxProjects
cd /mnt/c/Users/YOUR_WINDOWS_USERNAME/Desktop/LinuxProjects
echo "Hello from Kali on WSL2" > test.txt
```

Open the current directory in Windows File Explorer:

```bash
explorer.exe .
```

### 6.2 Linux filesystem or Windows filesystem?

Use the Linux filesystem for Linux-heavy development:

```bash
mkdir -p ~/projects
cd ~/projects
```

Use `/mnt/c/...` when the same files need to be opened frequently by Windows applications.

**Performance note:** Linux tools generally perform better when projects are stored in the WSL Linux filesystem (for example, under `/home/<linux-user>/projects`). Windows-mounted drives are convenient for sharing, but can be slower for workloads involving many small file operations.

**File access note:** WSL can access Windows files, but Windows applications and Linux applications may apply different permissions and metadata rules. Keep backups of important work.

---

## 7. Update and maintain Kali

Run these commands inside Kali.

Refresh the package index:

```bash
sudo apt update
```

Upgrade installed packages:

```bash
sudo apt upgrade
```

For a more comprehensive Kali rolling distribution upgrade:

```bash
sudo apt full-upgrade
```

Review the proposed package changes before confirming, especially when packages will be removed or replaced.

List packages with available upgrades:

```bash
apt list --upgradable
```

Clean downloaded package archives:

```bash
sudo apt clean
```

Remove dependencies that are no longer required:

```bash
sudo apt autoremove
```

Read the removal list before accepting it. Do not run upgrades or removals blindly in the middle of important work.

---

## 8. Install essential terminal tools

The following is a practical starter set, not a requirement to install every tool available in Kali.

### 8.1 Refresh package information

```bash
sudo apt update
```

### 8.2 Install general command-line and development tools

```bash
sudo apt install -y \
  git curl wget tree htop tmux \
  unzip zip jq \
  build-essential \
  python3 python3-pip python3-venv pipx \
  iproute2 iputils-ping dnsutils net-tools
```

What these tools are for:

| Package/tool | Purpose |
|---|---|
| `git` | Version control |
| `curl`, `wget` | Download and HTTP requests |
| `tree` | Display directory trees |
| `htop` | Interactive process monitor |
| `tmux` | Terminal multiplexer |
| `unzip`, `zip` | ZIP archives |
| `jq` | Process JSON in the terminal |
| `build-essential` | Common compilers and build tools |
| `python3` | Python interpreter |
| `python3-venv` | Python virtual environments |
| `pipx` | Install Python command-line applications in isolated environments |
| `iproute2` | Network interface and route tools |
| `iputils-ping` | `ping` |
| `dnsutils` | DNS utilities such as `dig` and `nslookup` |
| `net-tools` | Legacy network utilities such as `ifconfig` and `netstat` |

### 8.3 Install Android and typing tools

```bash
sudo apt install -y adb scrcpy usbutils gtypist klavaro
```

| Package/tool | Purpose |
|---|---|
| `adb` | Android Debug Bridge |
| `scrcpy` | Mirror and control an Android device |
| `usbutils` | USB inspection tools such as `lsusb` |
| `gtypist` | Terminal-based typing tutor |
| `klavaro` | Graphical typing tutor |

`klavaro` and `scrcpy` are graphical applications. They need a working WSLg graphical environment. If a package is unavailable in the configured Kali repositories, check the package name and repository status rather than downloading an installer from an untrusted site.

### 8.4 Verify installations

```bash
git --version
python3 --version
pipx --version
adb version
scrcpy --version
gtypist --version
```

Launch the typing tutor:

```bash
gtypist
```

Launch the graphical typing tutor:

```bash
klavaro
```

Launch a graphical app only after confirming that WSLg is available. On supported Windows 11 installations, WSLg is normally integrated with WSL.

---

## 9. Python development

Python is useful for automation, scripting, data processing, APIs, and application development.

### 9.1 Create a project

```bash
mkdir -p ~/projects/python/hello-python
cd ~/projects/python/hello-python
```

### 9.2 Create a virtual environment

```bash
python3 -m venv .venv
```

Activate it:

```bash
source .venv/bin/activate
```

The shell prompt usually displays the virtual environment name.

### 9.3 Install a library

```bash
python -m pip install requests
```

Use virtual environments for project dependencies. Modern Debian-derived distributions may prevent installing Python packages globally with `pip` because the system Python environment is managed by APT. Avoid bypassing that protection with `--break-system-packages`.

### 9.4 Create and run a Python file

```bash
nano app.py
```

Add:

```python
print("Hello from Kali on WSL2!")
```

Save in nano with `Ctrl + O`, press Enter, then exit with `Ctrl + X`.

Run:

```bash
python app.py
```

Deactivate the virtual environment when finished:

```bash
deactivate
```

### 9.5 Install Python command-line applications with pipx

For Python-based command-line applications:

```bash
pipx ensurepath
```

You may need to close and reopen the shell for the PATH change to take effect. Then install an application by its package name, for example:

```bash
pipx install httpie
```

---

## 10. Git and GitHub

Check Git:

```bash
git --version
```

Configure your identity. Use your own name and email:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Check the configuration:

```bash
git config --global --list
```

### 10.1 Create a repository

```bash
mkdir -p ~/projects/example-project
cd ~/projects/example-project
git init
```

Create a file, then:

```bash
git status
git add .
git commit -m "Initial commit"
```

### 10.2 Clone a repository

```bash
git clone https://github.com/OWNER/REPOSITORY.git
```

Replace the example URL with the repository you intend to clone.

### 10.3 Publish a README to GitHub

A typical repository structure:

```text
my-project/
├── README.md
├── .gitignore
└── src/
```

To commit a README:

```bash
git add README.md
git commit -m "Add project documentation"
```

Authentication to GitHub should use a supported method such as SSH keys or Git Credential Manager. Do not put passwords, access tokens, private keys, or other secrets in a public repository.

---

## 11. Android debugging with ADB

ADB means **Android Debug Bridge**. It is a command-line tool for communicating with an Android device for development and debugging.

Only enable debugging and run ADB commands on devices you own or are authorised to administer.

### 11.1 Enable Developer Options

The exact menu names vary by Android manufacturer and software version.

1. Open Android Settings.
2. Open About phone / My phone.
3. Find Build number (sometimes inside Software information).
4. Tap Build number repeatedly until Developer Options are enabled.
5. Return to Settings and open Developer Options.
6. Enable USB debugging when using a USB connection.

On an Infinix phone, menu labels and locations can vary by XOS version.

### 11.2 Check ADB in Kali

```bash
adb version
adb devices
adb devices -l
```

The first time a device connects, unlock the phone and approve the **Allow USB debugging?** prompt. Only authorise computers you trust.

Possible device states:

- `device` — connected and authorised.
- `unauthorized` — unlock the phone and approve the authorisation prompt.
- `offline` — the device is detected but not currently responding correctly.
- No device listed — check the cable, USB mode, USB passthrough, and ADB server.

### 11.3 Useful ADB commands

Get the device model:

```bash
adb shell getprop ro.product.model
```

Get the Android release:

```bash
adb shell getprop ro.build.version.release
```

Open a shell on Android:

```bash
adb shell
```

Exit the Android shell:

```bash
exit
```

List installed package identifiers:

```bash
adb shell pm list packages
```

Search package identifiers:

```bash
adb shell pm list packages | grep KEYWORD
```

Copy a file from Android to Kali:

```bash
adb pull /sdcard/Download/example.txt ~/Downloads/
```

Copy a file from Kali to Android:

```bash
adb push example.txt /sdcard/Download/
```

Take a screenshot and copy it to the current Kali directory:

```bash
adb shell screencap -p /sdcard/screenshot.png
adb pull /sdcard/screenshot.png .
```

Restart the ADB server:

```bash
adb kill-server
adb start-server
```

---

## 12. Wireless ADB on Android 11 and newer

Android's Wireless debugging feature can pair a computer over a local network. The phone and computer should be on the same Wi-Fi network, and the network must allow devices to communicate with each other.

### 12.1 Enable Wireless debugging on Android

1. Enable Developer Options.
2. Open Developer Options.
3. Enable **Wireless debugging**.
4. Choose **Pair device with pairing code**.
5. The phone displays an IP address, a pairing port, and a temporary pairing code.

Keep the pairing code private. Do not publish it or send it to anyone.

### 12.2 Pair from Kali

Use the IP address and **pairing port** displayed by the phone:

```bash
adb pair PHONE_IP:PAIRING_PORT
```

Enter the pairing code when prompted.

Example format only:

```bash
adb pair 192.168.1.25:37123
```

The example address and port are placeholders; use the values currently displayed on your phone.

### 12.3 Connect to the debugging port

Return to the main Wireless debugging screen. It displays a separate IP address and **debugging port**.

Connect using that address and port:

```bash
adb connect PHONE_IP:DEBUG_PORT
```

Verify:

```bash
adb devices
```

The device should appear with an IP address and the state `device`.

> **Important:** The pairing port and debugging port are not necessarily the same. Use the pairing port for `adb pair`; use the main debugging port for `adb connect`.

Disconnect:

```bash
adb disconnect PHONE_IP:DEBUG_PORT
```

### 12.4 Wireless ADB troubleshooting

If pairing or connecting fails:

- Confirm the phone and computer are on the same Wi-Fi network.
- Check that Wireless debugging is still enabled.
- Use the current IP address and port shown on the phone; they can change.
- Check whether the Wi-Fi network has client isolation enabled.
- Check Windows Firewall and any network security software.
- Remember that WSL2 networking can affect access from the Linux environment to devices on the local network.

If WSL cannot reach the phone, try the ADB connection from Windows using Google's Android SDK Platform-Tools. You can still use a Windows-native Android toolchain if that is more reliable for your network.

---

## 13. Mirror and control Android with scrcpy

`scrcpy` displays an Android device on the computer and allows interaction using the computer's keyboard and mouse. It uses ADB and does not require root access.

Official project: [Genymobile/scrcpy](https://github.com/Genymobile/scrcpy). Download scrcpy only from the official project or your distribution's trusted package repository.

### 13.1 Verify the installation

```bash
scrcpy --version
adb devices
```

ADB must detect an authorised device before starting scrcpy.

### 13.2 Start mirroring

```bash
scrcpy
```

For a specific wireless device:

```bash
scrcpy -s PHONE_IP:DEBUG_PORT
```

If more than one device is connected, specify the serial shown by `adb devices`.

### 13.3 Useful options

Limit the mirrored size:

```bash
scrcpy --max-size 1280
```

Limit the frame rate:

```bash
scrcpy --max-fps 30
```

Disable audio forwarding:

```bash
scrcpy --no-audio
```

Record a session:

```bash
scrcpy --record phone-session.mp4
```

Combine options:

```bash
scrcpy --max-size 1280 --max-fps 30 --no-audio
```

### 13.4 If the scrcpy window does not appear

Check:

1. `scrcpy --version` confirms that it is installed.
2. `adb devices` lists the phone as `device`.
3. WSLg is available and graphical Linux applications can open.
4. The terminal output does not show a display, SDL, or ADB error.

For USB-connected phones, WSL2 may need USB passthrough configured as described in the next section. For wireless connections, confirm that ADB can reach the phone before launching scrcpy.

---

## 14. Connect USB devices to WSL2 with usbipd-win

Windows normally owns USB devices. To make a USB device available to a WSL2 distribution, use **usbipd-win**, which shares USB devices with WSL through USB/IP.

Official project: [dorssel/usbipd-win](https://github.com/dorssel/usbipd-win). Microsoft guide: [Connect USB devices to WSL](https://learn.microsoft.com/windows/wsl/connect-usb).

### 14.1 Install usbipd-win

Run in **Administrator PowerShell**:

```powershell
winget install --interactive --exact dorssel.usbipd-win
```

If it is already installed, check:

```powershell
usbipd --version
```

Follow the current usbipd-win documentation if the installation command or options change.

### 14.2 Identify the Android phone

1. Connect the phone with a USB data cable.
2. Enable USB debugging on the phone.
3. Run in Administrator PowerShell:

```powershell
usbipd list
```

Find the phone and note its actual BUSID.

Do not copy a BUSID from an example. It is assigned by Windows and can differ between computers or connections.

### 14.3 Bind the device

In Administrator PowerShell, replace `BUSID` with the value from `usbipd list`:

```powershell
usbipd bind --busid BUSID
```

Binding makes the device available for attachment. Administrative permission is required.

### 14.4 Attach it to WSL

Start Kali first, then in PowerShell run:

```powershell
usbipd attach --wsl --busid BUSID
```

Replace `BUSID` with the real value.

The USB device is passed to WSL, so Windows applications may no longer be able to use it while it is attached to Linux.

### 14.5 Verify from Kali

Inside Kali:

```bash
lsusb
adb devices
```

Unlock the phone and approve the USB debugging prompt if it appears.

If the phone is not listed, check the cable, USB mode, BUSID, attachment state, and usbipd-win output. You may need to attach the device again after disconnecting it or restarting WSL.

---

## 15. Networking and diagnostics

Run these in Kali.

Show network interfaces and addresses:

```bash
ip addr
```

Show routing information:

```bash
ip route
```

Test DNS and internet connectivity:

```bash
ping -c 4 kali.org
```

Look up a domain:

```bash
nslookup kali.org
```

Query DNS:

```bash
dig kali.org
```

Check HTTP response headers:

```bash
curl -I https://www.kali.org
```

Show listening TCP and UDP sockets:

```bash
ss -tuln
```

Show listening sockets and owning processes:

```bash
sudo ss -tulpn
```

A failed ping does not always mean internet access is unavailable; some networks block ICMP. Test DNS and HTTPS as well.

---

## 16. Terminal productivity

### 16.1 Command history

```bash
history
history | grep apt
```

Repeat the previous command:

```bash
!!
```

Review a command before repeating it, especially if it contains `sudo`, file deletion, or system changes.

### 16.2 Useful keyboard shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl + C` | Interrupt the foreground process |
| `Ctrl + D` | Send end-of-input or exit the shell |
| `Ctrl + L` | Clear the terminal display |
| `Ctrl + A` | Move to the beginning of the command line |
| `Ctrl + E` | Move to the end of the command line |
| `Ctrl + U` | Delete from cursor to beginning |
| `Ctrl + K` | Delete from cursor to end |
| `Tab` | Complete a command or path |
| Up arrow | Previous command |
| Down arrow | Next command |

### 16.3 Use tmux

Start tmux:

```bash
tmux
```

Detach from a session with `Ctrl + B`, then `D`.

List sessions:

```bash
tmux ls
```

Reattach:

```bash
tmux attach
```

Create a named session:

```bash
tmux new -s development
```

Reattach to that session:

```bash
tmux attach -t development
```

---

## 17. Suggested workspace layout

A useful Linux-side workspace:

```text
/home/<linux-user>/
├── projects/
│   ├── python/
│   ├── bash/
│   ├── networking/
│   └── android/
├── scripts/
├── tools/
├── notes/
└── Downloads/
```

Create it:

```bash
mkdir -p ~/projects/{python,bash,networking,android}
mkdir -p ~/scripts ~/tools ~/notes ~/Downloads
```

For files shared with Windows, create a separate folder under your Windows profile:

```bash
mkdir -p /mnt/c/Users/YOUR_WINDOWS_USERNAME/Desktop/LinuxProjects
```

Replace the placeholder with the real Windows profile folder name.

---

## 18. WSL management and backup

These commands run in **PowerShell**.

### 18.1 Check WSL status

```powershell
wsl --status
wsl --version
wsl --list --verbose
```

### 18.2 Shut down WSL

```powershell
wsl --shutdown
```

### 18.3 Export a Kali backup

Create a backup archive:

```powershell
wsl --export kali-linux "$env:USERPROFILE\kali-wsl-backup.tar"
```

Choose a drive with enough free space. The export can be large.

### 18.4 Importing a backup

Importing a distribution is a separate operation and requires a destination directory. Do not import over an existing installation without understanding the effect. Consult Microsoft's current WSL command reference before restoring:

[Basic commands for WSL](https://learn.microsoft.com/windows/wsl/basic-commands)

### 18.5 Back up project files too

A WSL export is useful for recovering the distribution, but important projects should also be backed up independently (for example, to a separate drive or a private Git repository). A backup is only useful if it can be restored.

---

## 19. Troubleshooting

### `wsl` is not recognised in PowerShell

Try the full path:

```powershell
C:\Windows\System32\wsl.exe --status
```

If it works, investigate the Windows PATH or terminal environment.

### Kali does not start

Run in PowerShell:

```powershell
wsl --list --verbose
wsl --status
wsl --update
```

If necessary, close WSL and try again:

```powershell
wsl --shutdown
wsl -d kali-linux
```

### APT cannot download packages

Inside Kali:

```bash
sudo apt update
```

Check the network:

```bash
ip route
getent hosts http.kali.org
curl -I https://http.kali.org
```

If name resolution or HTTPS fails, investigate the Windows connection, VPN, proxy, DNS, and WSL networking.

### A command is not found

Check whether the package is installed:

```bash
command -v COMMAND_NAME
```

Search for a package:

```bash
apt search PACKAGE_NAME
```

Install the correct package using APT if it is available in the configured repositories.

### ADB shows `unauthorized`

Unlock the Android phone and approve the debugging authorisation prompt. Disconnect and reconnect if needed.

### ADB shows no devices over USB

Check that:

- The cable supports data, not only charging.
- USB debugging is enabled.
- The phone is unlocked.
- usbipd-win has attached the phone to WSL.
- `lsusb` can see the USB device.

### Wireless ADB cannot connect

Confirm the phone and PC share a network, use the current IP and ports, and check whether the Wi-Fi blocks device-to-device traffic. If necessary, test ADB from Windows to separate a WSL networking issue from a phone issue.

### scrcpy cannot open a window

Confirm ADB sees an authorised device, check WSLg, and read the error printed by scrcpy. Test a simple graphical Linux application to determine whether the issue is specific to scrcpy or to GUI forwarding.

### WSL networking warning

WSL may report a networking-mode warning and fall back to another supported mode. If internet access works, the warning may not prevent ordinary terminal use. If local-network device discovery or wireless ADB fails, consult the current WSL networking documentation before changing `.wslconfig`.

---

## 20. What WSL2 can and cannot replace

### Good use cases for Kali WSL2

- Bash and Linux command-line learning.
- Python, Git, scripting, and software development.
- Database clients and command-line database services.
- Android ADB and, where the display/device connection works, scrcpy.
- Network diagnostics and many user-space networking tools.
- Accessing Windows files from Linux.
- Everyday Linux terminal workflows without rebooting.

### Tasks better suited to native Kali or dedicated hardware

- Wi-Fi monitor mode and packet injection.
- Some wireless adapters and specialised USB hardware.
- Kernel module development and kernel-level experiments.
- Hardware access that WSL does not expose.
- Security labs that explicitly require a native Linux kernel or a dedicated virtual machine.
- Certain low-level networking and forensic workflows.

Use the full dual-boot Kali installation when a task requires direct hardware or kernel capabilities that WSL2 does not provide.

---

## 21. Daily workflow

A simple routine:

1. Start Windows.
2. Open Windows Terminal.
3. Open the Kali profile.
4. Go to the project directory.
5. Check the current directory and Git status.
6. Work with the required Linux tools.
7. Save and back up important work.

Example startup commands inside Kali:

```bash
cd ~/projects
pwd
whoami
```

Before installing a new package:

```bash
sudo apt update
```

There is no need to run `sudo apt update` every time Kali opens. Run it periodically and before installing packages.

---

## 22. Security and responsible use

- Use `sudo` only when elevated privileges are required.
- Read commands before running them, especially commands copied from the internet.
- Avoid installing large Kali metapackages unless you actually need them.
- Keep Windows security protections enabled.
- Only use ADB with devices you own or have permission to administer.
- Do not expose development services to public networks unnecessarily.
- Back up important data before major upgrades or filesystem changes.
- Never commit passwords, API keys, access tokens, private keys, pairing codes, or personal data to a public repository.
- Check paths carefully before using `rm -r` or `rm -rf`.
- Use native Kali when a task needs direct hardware access unavailable in WSL2.

---

## 23. Setup checklist

Use this checklist when building a new WSL2 Kali environment:

- [ ] Confirm Windows supports WSL2 and hardware virtualisation is enabled.
- [ ] Install or update WSL.
- [ ] Install Kali Linux.
- [ ] Confirm Kali is running as WSL version 2.
- [ ] Open Kali from Windows Terminal.
- [ ] Learn the basic navigation and file commands.
- [ ] Confirm Windows files are accessible under `/mnt`.
- [ ] Create a Linux-side project workspace.
- [ ] Install essential terminal tools.
- [ ] Verify Python, Git, ADB, and scrcpy.
- [ ] Create a Python virtual environment for each Python project.
- [ ] Configure Git identity.
- [ ] Test Android ADB authorisation.
- [ ] Test Wireless debugging if needed.
- [ ] Install usbipd-win if USB passthrough is needed.
- [ ] Test scrcpy with an authorised Android device.
- [ ] Test graphical Linux applications if needed.
- [ ] Export a WSL backup after configuring the environment.
- [ ] Back up important project files separately.

---

## 24. Official references

Use official documentation for changes to commands, supported Windows versions, and current tool releases.

- [Kali Linux — WSL documentation](https://www.kali.org/docs/wsl/)
- [Kali Linux — WSL preparation](https://www.kali.org/docs/wsl/wsl-preparations/)
- [Microsoft — Install WSL](https://learn.microsoft.com/windows/wsl/install)
- [Microsoft — Basic WSL commands](https://learn.microsoft.com/windows/wsl/basic-commands)
- [Microsoft — Access Linux files from Windows and Windows files from Linux](https://learn.microsoft.com/windows/wsl/filesystems)
- [Microsoft — Connect USB devices to WSL](https://learn.microsoft.com/windows/wsl/connect-usb)
- [usbipd-win — Official GitHub repository](https://github.com/dorssel/usbipd-win)
- [Android — ADB documentation](https://developer.android.com/tools/adb)
- [Android — Enable developer options and USB debugging](https://developer.android.com/studio/debug/dev-options)
- [scrcpy — Official GitHub repository](https://github.com/Genymobile/scrcpy)
- [scrcpy — Connection documentation](https://github.com/Genymobile/scrcpy/blob/master/doc/connection.md)

---

## Final note

This setup makes Windows the main desktop environment and Kali WSL2 the everyday Linux terminal. It avoids rebooting just to use common Linux command-line tools and makes Windows files accessible from Kali. A separate dual-boot Kali installation remains valuable for hardware-intensive and kernel-level work.

**The key habit is to know which environment a command belongs to:** PowerShell for Windows and WSL management, Kali Bash for Linux commands, and Android Settings for phone-side debugging permissions.
