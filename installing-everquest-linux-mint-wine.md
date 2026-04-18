# Installing EverQuest in Linux Wine

## *EQAscendant Version* 

**Author note & disclaimer:** I (Hrain on EQAscendant Discord) and (timvgaming on GitHub), do not serve in any official capacity with EQAscendant, nor do I speak for the project or its staff. I am simply an EverQuest player who enjoys this server, and is sharing what I’ve learned to help other players. All configuration guidance here reflects personal experience, not official project policy. 



## \## <a id="toc"></a>Table of Contents

\- [Introduction](#installing-everquest-in-linux-wine)

\- [Technical Notes](#Technical Notes)

\- [Document Goals](#document-goals)

\- [Installation Overview](#installation-overview)

\- [Prerequisites](#prerequisites)

\- [Install the Supporting Software Stack](#Install the host-level software stack)

\- [Installing EverQuest](#OK! Let’s install a game)



## Technical Notes 	

This guide was explicitly tested on the following system:

- **OS:** Linux Mint 22.3 (x86_64) Kernel: 6.17.0-20-generic 
- **Desktop Environment:** Cinnamon 6.6.7 (Muffin) 
- **CPU:** Intel 13th‑Gen Core i5‑13600K
- **GPU:** NVIDIA GeForce RTX 4070 Ti 
- **Display:** 3840×2160 
- **Memory:** 128 GB RAM 

Software stack versions as installed during testing:

- **Wine:** 9.0~repack-4build3 (my daily driver); installed via the Linux Mint Software Manager 
- **Wine:** 11.0 installed temporarily for compatibility testing; installed via apt 
- **DXVK:** v2.7.1; installed by Wine as needed
- **winetricks:** 20240105-2; installed via the Linux Mint Software Manager 
- **NVIDIA Graphics Driver:** nvidia-driver-580-open; installed via the Linux Mint Driver Manager 
- **Wine Mono:** wine-mono-11.0.0-x86.msi; user provided and installed by Wine as needed

All required software dependencies are covered by the above software stack installations. Results on other distributions, kernels, desktop environments, driver versions, Wine builds or hardware may vary slightly, but the overall procedure should remain applicable. 

## Document goals 

This guide helps you get up and running with one or more EverQuest clients on Linux using Wine, while giving you a practical understanding of the pieces involved so you can scale (for boxing) with confidence. Principle goals are:

- Launch and play EverQuest via the EQAscendant patcher. 
- Understand Wine prefixes 
- Apply the minimal Wine configuration needed for correct input and display. 
- Use DXVK for stable DirectX 9 rendering. 
- Use wine-mono to provide .NET application support.
- Create repeatable multi‑instance installs (eq1, eq2, eq3…) without cross‑contamination. 

## Installation overview ## 

[Back to ToC](#toc)

This guide follows a **repeatable, per‑client workflow.** Each EverQuest client is isolated in its own Wine prefix and follows the same setup steps. Nearly all parts of the guides' installation process are executed via your terminal in a **copy** command from guide **& paste** into terminal, and then **press Enter** flow. Successful results of the command execution are predictable and will be shown as part of the guide process in cases where there may be some ambiguity. Procedure flow is as follows:

**One-time Process:**

- Define and administer prerequisites.
- Install the supporting software stack 
- Create an organized directory tree / prefix layout
- Populate the EQ Client/Patcher staging folder

**Repeatable (once per EQ client) Process:**

- Create a Wine prefix for the EQ client
- Initialize the Wine prefix and setup the starting Wine Configuration.
- Install DXVK into the Wine prefix .
- Create the EQ directory inside the prefix and copy the EverQuest files to it. 
- Setup first‑launch display containment.
- Install wine-mono.
- Launch the EQAscendant patcher & the EverQuest client
- Create a desktop launcher for the patcher bound to that prefix. 
- Define a repeatable process for adding additional clients (eq2, eq3, …). 

## Prerequisites ##

**EverQuest game and patcher files**

Before installing any software or creating Wine prefixes, you must already have the following game‑specific assets. If you cannot locate these, you should **stop here** — the remaining steps depend on them.

1. A legally obtained EverQuest client (Rain of Fear era) that you are licensed to use:
   - Existing Rain of Fear client directories from a prior Windows or Linux installation may be reused. 
   - A web search on "download everquest rof" will yield some sources.
     - Download responsibly. 
     - Do your due diligence to ensure any downloaded files are safe. 
2. The EQAscendant patcher and game executables:
   - The EQAscendant project distributes the patcher as a ZIP archive. 
     - Obtain it from the project’s official Discord distribution location: 
     - https://discord.com/channels/1467192820610764871/1467195100961706074 
3. Any locally stored copies you intend to reuse:
   - Previously downloaded or installed EQAscendant patcher files may be reused. 
   - Previously downloaded or installed EverQuest client files may also be reused if they meet the required era (rof). 
4. These assets are copied or installed into each Wine prefix during later steps. 

**EQEmulator and login server accounts.** 

1. EQAscendant (at the time of this writing) allows 3 clients out in the world killing stuff, and an additional client in the Bazaar selling your **Phat Lewts!**. For each EverQuest instance that you intend to run simultaneously you will need a login server account. To get login server accounts:
   1. If you don't have one, create an EQEmulator account at:
      - https://www.eqemulator.org/

   2. Login to your EQEmulator account and:
      - On the left side of the window you should see **Loginserver Accounts**. Beneath that:
      - Click **Create Account**, and follow the prompts to create a login server account.
      - I think they have a limit on how many accounts you can create in a day. 
      - You can use any existing accounts that you may have made for other EMUs. They are not EMU exclusive.
      - You will need at least **1 login server account** (❌ not your EQEmulator account) to complete the EQ install below


## Install the host-level software stack

These components form the baseline environment required to get EverQuest up and running. **Important guardrail (package sources):** Whenever possible, use your distro’s managed packages first (e.g., Linux Mint Software Manager / Driver Manager). Only fall back to command‑line installs (APT) when the software is not available or is materially outdated in the managed repositories. This minimizes dependency conflicts and keeps upgrades clean and supportable. Before configuring a Wine prefix or installing EverQuest, ensure the following **host‑level software** is installed on your system:

**Source order used in this guide:**

1. Distro Software/Driver Manager (preferred) 
2. Distro APT repositories (fallback) 
3. Upstream installers (used only when necessary and called out explicitly) 

### Host-level software stack Installation procedures (performed one time): ###

**Using copy & paste to terminal for the fledgling Linux user:**

For the new and uninitiated Linux user, that was me a few weeks ago, the prospects of needing to use terminal can be a bit daunting. Rest assured that you are not going to have to learn any commands here, though that could be a nice side benefit of following this guide. Any step requiring the use of terminal (by the way you can open terminal with Ctrl+Alt+T) will be a simple matter of copying the command from the guide and pasting it into terminal and then pressing Enter.

A quick primer on copying commands from this guide and pasting into terminal. You'll be doing a lot of that soon. Copying is the intuitive part. In the code block example below, you'll see a Copy button at the far right of the block. **Click** the button and the code block contents are placed in your clipboard.

```bash
some cryptic terminal command -r whodat reXing my system
```

Pasting into your terminal may not be as intuitive. 1) Clicking anywhere in terminal, then pressing Ctrl+Shift+V should paste the clipboard contents into terminal, or 2) Clicking the Right mouse button anywhere in terminal should display a context menu that includes a paste option. Both options will paste the clipboard contents at the command prompt, and then you just press Enter. 

#### On to the Host-level software stack one time Installation procedures: 

**A)  GPU driver:** No standalone driver installation commands or version pinning are required for this guide. Use your distro Driver Manager to select and maintain your GPU driver.

1) Review the drivers offered for your GPU. 

- Either: Select the recommended driver and apply it, or 

- Keep your current driver if it is already working well and you are satisfied with it.

**B) Wine: **Wine is the compatibility layer that allows Windows applications to run on Linux.

1) Before installing or upgrading Wine, perform a version check: 

```bash
wine --version
```

- If the reported version is Wine 9.x or newer, no action is required — proceed directly to **C) winetricks**. 
- If the command prints wine: command not found, Wine is **not** installed — continue below.
- If Wine is not installed, or the reported version is older than Wine 9, continue below. 
  - Open your distro Software Manager. 
  - Search for Wine. 
  - If a package providing Wine 9.x or newer is available, install it.
  - **If** the available packages do not provide Wine 9.x or newer, install via the Fallback option shown below.:

**Fallback (APT):** ⚠️ Use only if directed by the previous step.


```bash
sudo apt update
sudo apt install wine
```

APT will install the distro‑provided stable package. Verify version: 

```bash
wine --version
```

**C) winetricks:** Winetricks is a helper tool that installs common Windows runtime components into a Wine prefix.

1) Before installing winetricks, perform a version check: 

```bash
winetricks --version 2>/dev/null | cut -d' ' -f1
```

- If the command prints any version string (something like 20240105, actual version doesn't matter), winetricks is already installed — proceed directly to **D) DXVK**. 
- If the command prints winetricks: command not found, winetricks is **not** installed — continue below. 
  - Open your distro Software Manager. 
  - Search for winetricks. 
  - Install the available package. 
  - **If** no package is available, install via the Fallback below. 

**Fallback (APT):** ⚠️ Use only if directed by the previous step.

```bash
sudo apt update
sudo apt install winetricks
```

APT will install the distro‑provided stable package. Verify version: 

```bash
winetricks --version 2>/dev/null | cut -d' ' -f1
```

**D) DXVK:** DXVK is a translation layer that converts DirectX 9/10/11 calls into Vulkan. It is installed as needed by Wine. Vulkan is part of your GPU driver. There is no standalone install for DXVK or Vulkan, and I'm just showing them here because I know how much you guys like to read.

**E) Wine Mono:** Wine Mono is installed by Wine per‑Wine prefix, not system‑wide. It will be installed as part of the EverQuest build procedures. We only need to download it here and pre‑position it for use later by Wine. 

1. Download this Wine Mono installer version: wine-mono-11.0.0-x86.msi 

- Official Wine Mono installers are published by WineHQ here:
  - https://dl.winehq.org/wine/wine-mono/ 
- ❌  **Do not** install distro Mono packages (e.g. mono-runtime, mono-complete); **Wine requires the Windows Mono .msi**, not the system Mono runtime. 

- Place wine-mono-11.0.0-x86.msi in your Downloads folder ($HOME/Downloads). 

#### Final one-time configuration stuff  ####

Before running any commands, it’s important to understand exactly how this guide organizes files on disk. The layout below is not only a recommendation — it is the structure this guide **will** use when installing EverQuest on your system if you follow the guide verbatim. Using your own preferred layout is discussed at the bottom of this section.

**Goals of using this layout:**

- Folder organization
- One Wine prefix per EverQuest client 
- No shared state between clients 
- Clear, readable paths that scale cleanly for boxing 
- Easy cleanup, backup, and troubleshooting 

**Canonical directory tree** 


```text
~/Games/EQAscendant/
├── EQ-game-files/    # Staging area (EQ source files)
├── eq1/              # Wine prefix (eq1)
│  └── drive_c/
│    └── Program Files/
│      └── eq1/       # EverQuest client files (eq1)
├── eq2/              # Wine prefix (eq2)
│  └── drive_c/
│    └── Program Files/
│      └── eq2/       # EverQuest client files (eq2)
├── eq3/              # Wine prefix (eq3)
│  └── drive_c/
│    └── Program Files/
│      └── eq3/       # EverQuest client files (eq3)
├── eq4/              # Wine prefix (eq4)
   └── drive_c/
     └── Program Files/
       └── eq4/       # EverQuest client files (eq4)
```

How to read this

- ~/Games/EQAscendant/ is the root folder for all Wine prefixes and EverQuest clients. 
- EQ-game-files/ is a staging area used to hold a clean EverQuest client plus the EQAscendant patcher; its contents are copied into prefixes during installs and are never run directly. 
- Each eqN/ directory is a complete, isolated Wine prefix. This guide shows 4 eq directories. You can create as many as you like. 
- Each prefix installs EverQuest into its own matching directory under Program Files/eqN. 
- There is no shared Program Files/EverQuest directory. This one‑to‑one mapping (prefix ↔ client folder ↔ launcher) prevents cross‑contamination, makes boxing predictable, and ensures uninstalling a client is as simple as deleting its eqN/ directory. 

#### Using your own layout design 

Please use the layout described above for the purpose of getting through this guide with the provided copy and paste commands. After you have gone through this guide and are comfortable with the process you can create your EQ installs using your preferred layout. Uninstalling an existing Wine prefix and the associated EQ client is a simple matter of:

- Delete the prefix folder (or any higher level folder in your home directory). None of the previously installed host-level software packages are effected. No part of a Wine prefix lives outside of the prefix folder. Deleting it is targeted total annihilation and the ultimate uninstall process. 
- ❌ **Don’t** attempt to reuse any part of an existing prefix folder path in a new Wine prefix. You must follow the process to create each new Wine prefix. 

#### Populate the EQ-game-files folder (one‑time setup continued) 

Before creating the first Wine prefix, place the EverQuest client files and the EQAscendant patcher files to where the guide expects to find them. 

1. Create the directory tree:

```bash
mkdir -p ~/Games/EQAscendant/EQ-game-files
```

- ❌ **Do not** create any eqN directories yet; Wine will create them during prefix initialization. 

2. Copy and paste the EverQuest client files and the EQAscendant patcher files into the staging folder

- **Order matters.** This step builds the staging copy of EverQuest that will later be duplicated into one or more Wine prefixes. The EverQuest client files must be copied first and then the patcher files. It is necessary that the patcher files overwrite some of the client files.

- **Source and destination**

  - Source 1: your Rain of Fear (RoF) era EverQuest client directory 
  - Source 2: your EQAscendant patcher files directory

  - Destination: ~/Games/EQAscendant/EQ-game-files/

- Using your distro file manager, copy **all files and subdirectories** from the unzipped RoF client into the destination directory. Do **not copy the RoF client folder**; only copy the folders and files within it. This includes (but is not limited to):

  - Game executables and DLLs

  - Resources/Maps/uifiles/

  - Any other data directories present in the RoF client


- Do **not** attempt to launch EverQuest from the destination directory. 

3. Using your distro file manager, copy **all files and subdirectories** from the unzipped EQAscendant patcher into the destination directory. 

-  ⚠️ **Important**: If your file manager prompts you to choose an action for existing files (for example, Replace, Overwrite, or Merge), choose the option that replaces existing files. The patcher is expected to overwrite some files shipped with the base client. 

-  ❌ **What not to do:**

-  Do not mix files from different EverQuest eras 
-  Do not create any eqN Wine prefixes yet 
-  Do not run the patcher or the game from EQ-game-files

At the end of this step, EQ-game-files/ should contain a complete EverQuest client plus the EQAscendant patcher, ready to be copied into Wine prefixes. 

#### Pre‑configure initial EverQuest window settings

EverQuest will soon be launching for the first time inside a Wine prefix. To ensure a predictable, accessible first launch—and to standardize behavior across this prefix and any future prefixes—you will replace eqclient.ini in the staging folder with a minimal first‑launch version to define known window parameters. 

1. Create the minimal eqclient.ini file: 

```bash
cat > ~/Games/EQAscendant/EQ-game-files/eqclient.ini <<'EOF'
[VideoMode]
Width=1366
Height=768
WindowedWidth=1366
WindowedHeight=768
[Defaults]
WindowedMode=True
Gamma=4
EOF
```

- Additional settings are intentionally omitted. EverQuest will populate defaults and user preferences automatically on first launch. A low initial gamma (such as Gamma=4, approximately 15% in‑game) mitigates display gamma bleed into the desktop environment under Wine while still providing a comfortable baseline. You can fine‑tune gamma and dimensions later using the in‑game and winecfg options. 

- This creates a clean, deterministic baseline for the initial launch. No further editing is required at this stage. At this point, the EQ-game-files/ folder contains a clean, reproducible EverQuest + EQAscendant baseline. 

### Create the first Wine prefix (eq1) 

Everything you did above set the stage and never has to be done again. From here on, we build one EverQuest client at a time, each isolated in its own Wine prefix. The steps that follow focus on layout and prefix creation first, installing the game and patcher into that prefix, verifying a good launch and creating a desktop launcher.  Each EverQuest client lives in its own Wine prefix. Here's the first one!:

#############################################################################

###### **BEGIN REPEATABLE PREFIX INSTALLATION PROCESS**

#############################################################################

- ⚠️ Quick Note: The annotation **(eq1 x N)** is a count of the eq1 occurrences in the code block. More on that later.

1. Create the prefix directory: **(eq1 x 1)**

```bash
mkdir -p ~/Games/EQAscendant/eq1
```

2. Initialize the Wine prefix: **(eq1 x 1)**

```bash
WINEPREFIX=~/Games/EQAscendant/eq1 winecfg
```

- Accept prompts to install Wine Mono and/or Wine Gecko if offered. Wine may print warnings, noise or what even looks like errors while it's running — this is normal. The Wine configuration window should open **(success signal for this step)**.

3. In the Wine configuration window make these changes:

- Graphics tab: 
  - uncheck everything. 
  - Click **Apply**, then **OK**. 
- The wine configuration window will close.

4. Install DXVK into the eq1 prefix: **(eq1 x 1)**

```bash
WINEPREFIX=~/Games/EQAscendant/eq1 winetricks dxvk
```

- Non‑fatal warnings or fixme messages are expected as long as the command completes and returns terminal to the command prompt. **(success signal for this operation)**

5. Create the EverQuest install directory inside the prefix:  **(eq1 x 2)**

```bash
mkdir -p ~/Games/EQAscendant/eq1/drive_c/Program\ Files/eq1
```

6. Copy the EverQuest and patcher files into the eq1 prefix: **(eq1 x 2)**

```bash
cp -a ~/Games/EQAscendant/EQ-game-files/. ~/Games/EQAscendant/eq1/drive_c/Program\ Files/eq1/
```

- **Verify the file copy & paste succeeded:** Confirm the copy completed without errors. Browse to ~/Games/EQAscendant/eq1/drive_c/Program\ Files/eq1/ using your file manager or terminal. Confirm the destination contains many files and subdirectories.

7. In this Step, we explicitly finalize first‑launch display containment before starting the game. This ensures a predictable, non‑disruptive first run. 

- Enable Wine Virtual Desktop: Open Wine configuration for this prefix: **(eq1 x 1)**

```bash
WINEPREFIX=~/Games/EQAscendant/eq1 winecfg
```

- When the Wine configuration window opens **(success signal for this operation)**, make the following changes:

- Applications tab:
  - Click Add application

  - Navigate to the eq1 folder (Program Files/eq1), select eqgame.exe and click Open

- Select eqgame.exe in the Applications tab and then click the Libraries tab

- Libraries tab:
  - In New override for library, select dinput8 and click Add

  - Click the Edit button and Select (Native then Builtin), then OK

  - ⚠️ Note: This override is specific to EQAscendant and is required to correct an in-game issue where the user could not adjust Alternate Advancement "Exp to AA %"  

- Applications tab:
  - Select Default Settings and then click the Graphics tab

- Graphics tab: 

- - Enable: ✅ Allow the window manager to decorate the windows
  - Enable: ✅ Allow the window manager to control the windows
  - Enable: ✅ Emulate a virtual desktop
  - Ensure all other boxes are unchecked
  - Set the desktop size to 1366×768
  - Click **Apply**, then **OK**. 
- The wine configuration window will close.

- This containment step prevents full screen rendering during DirectX initialization and avoids display mode switching while EverQuest establishes its video state.  

8. Install wine-mono: **(eq1 x 1)** Wine-Mono is Wine’s open-source replacement for Microsoft’s .NET Framework. The EQAscendant patcher is a .NET application.

```bash
WINEPREFIX=~/Games/EQAscendant/eq1 wine msiexec /i ~/Downloads/wine-mono-11.0.0-x86.msi
```

- Success is indicated by the wine desktop being visible for a few seconds and terminal returning to the command prompt.

9. Launch the EQAscendant patcher, which will in turn launch EverQuest:

- In terminal, cd to the EverQuest install directory: **(eq1 x 2)**

```bash
cd ~/Games/EQAscendant/eq1/drive_c/Program\ Files/eq1
```

- Launch the EQAscendant patcher: **(eq1 x 1)**

```bash
WINEPREFIX=~/Games/EQAscendant/eq1 wine EQAscendant.exe
```

- Success is indicated by the EQAscendant patcher launching.

- Configure the patcher for hands‑off operation:

  - ✅ Check Auto Patch 

  - ✅ Check Auto Play 
    - On future launches, the patcher will automatically patch and start EverQuest without user interaction. 

  - click **Patch**


- After the patcher finishes patching, EverQuest will launch inside the Wine 1366×768 virtual desktop. (**the success signal for this step**).
  -  ⚠️ Note: Always launch EverQuest via the EQAscendant patcher. 
- Login to the Ascendant server

  - To speed server login along, when the SOE splash screen pops, **click** it to move to the login screen. If you don't "click", the splash screen remains in place for 30 seconds before progressing to login.

- Create a test character if you don't already have a character
- Enter world, and look around
- OK, that's enough. Log out and close the game completely.

  - ⚠️ Note that terminal is not presenting a command prompt. 
  - Click in terminal and press Ctrl+C to end the running process
  - You should now have a command prompt


10. Create a desktop launcher (eq1). 

- Create the applications directory (it's ok if it already exists) .

```bash
mkdir -p ~/.local/share/applications
```

- Create the icons directory (it's ok if it already exists).

```bash
mkdir -p ~/.local/share/icons
```

- Copy the patcher icon to  ~/.local/share/icons. **(eq1 x 3)**
- ⚠️ Note: This is our first use of "sudo" so you'll have to enter your password after entering this command:

```bash
sudo cp ~/Games/EQAscendant/eq1/drive_c/Program\ Files/eq1/eqemupatcher.png ~/.local/share/icons/eqascendant-eq1.png
```

- Create the launcher .desktop file: **(eq1 x 8)**

```bash
cat > ~/.local/share/applications/EQAscendant-eq1.desktop <<'EOF'
[Desktop Entry]
Version=1.0
Type=Application
Name=EQAscendant-eq1
Comment=Launch EverQuest via EQAscendant using the eq1 Wine prefix
# Use a shell wrapper so quoting, spaces, and environment are handled reliably by all desktops
Exec=sh -c 'WINEPREFIX="$HOME/Games/EQAscendant/eq1" exec wine "$HOME/Games/EQAscendant/eq1/drive_c/Program Files/eq1/EQAscendant.exe"'
# Icon is resolved by name from ~/.local/share/icons
Icon=eqascendant-eq1
Terminal=false
Categories=Game;
EOF
chmod +x ~/.local/share/applications/EQAscendant-eq1.desktop
```

- Refresh desktop application database:

```bash
update-desktop-database ~/.local/share/applications
```

- The launcher will now appear in your desktop environment’s application menu (Games category). You can right‑click it to Add to Desktop or Pin to Panel. If your distro doesn't support adding to desktop from the menu, you can place a copy on your desktop by:

- Copy the launcher from /applications to /Desktop:

```bash
cp ~/.local/share/applications/EQAscendant-eq1.desktop ~/Desktop/EQAscendant-eq1.desktop
```

- This places a clickable EverQuest launcher directly on your desktop. 

#############################################################################

###### **END REPEATABLE PREFIX INSTALLATION PROCESS**

#############################################################################

- Enable Wine Virtual Desktop: Open Wine configuration for this prefix: **(eq1 x 1)**

```bash
WINEPREFIX=~/Games/EQAscendant/eq1 winecfg
```

- When the Wine configuration window opens **(success signal for this operation)**, make only the following changes:

- Graphics tab: 
  - Enable: ✅ Emulate a virtual desktop
  - Set the desktop size to 1366×768
  - Ensure all other boxes are unchecked
  - Click **Apply**, then **OK**. 
- The wine configuration window will close.

1. Open eqclient and edit the video settings:

- Open with nano (default terminal based text editor for most distro)

```bash
nano ~/Games/EQAscendant/eq1/drive_c/Program\ Files/eq1/eqclient.ini
```

- or open with kate (editor similar to notepad/notepad++) install with: sudo apt install kate

```bash
kate ~/Games/EQAscendant/eq1/drive_c/Program\ Files/eq1/eqclient.ini
```

- 

### Suggested Wine Virtual Desktop and EverQuest Window Resolutions

When using Wine’s virtual desktop and running EverQuest in windowed mode, it is often helpful to choose resolutions that are *smaller than your native monitor resolution*. This improves usability when boxing, reduces GPU overhead, and avoids UI scaling issues.

The table below lists common monitor resolutions and several descending, practical window sizes that work well for Wine virtual desktops and EverQuest client windows.

These are *recommendations*, not requirements. Feel free to experiment once your setup is stable. They key to choosing a EQ resolution and a matching Wine desktop resolution is to initially set both to large, or to your Monitors Native resolution, and then in-game set display video modes to various options and find one you're satisfied with. Note that the Wine desktop will not resize based on your in-game selection. 

| Your Monitor (Native) | Large     | Medium (Good Default) | Small (Boxing‑Friendly) | Very Small (Utility/Box) |
| --------------------- | --------- | --------------------- | ----------------------- | ------------------------ |
| 3840×2160 (4K)        | 2560×1440 | 1920×1080             | 1600×900                | 1280×720                 |
| 2560×1440 (1440p)     | 1920×1080 | 1600×900              | 1366×768                | 1280×720                 |
| 1920×1080 (1080p)     | 1600×900  | 1366×768              | 1280×720                | 1024×768                 |
| 1680×1050             | 1440×900  | 1280×800              | 1280×720                | 1024×768                 |
| 1366×768              | 1280×720  | 1024×768              | 1024×600                | 800×600                  |

Guidance:

- Use a medium resolution for your first successful launch.
- Use smaller resolutions when running multiple clients simultaneously.
- EverQuest’s UI scales better when resolutions follow standard 16:9 or 16:10 ratios.
- You can use *different resolutions per client* when boxing.

Wine’s virtual desktop only constrains window boundaries — it does *not* affect in‑game gamma behavior, input handling, or rendering quality.

```bash##### 
# Edit eq1's eqclient.ini [VideoMode] dimensions (preserves rest of file)
# Set these:
X=1600
Y=900

INI="$HOME/Games/EQAscendant/eq1/drive_c/Program Files/eq1/eqclient.ini"

# Safety backup
cp -a "$INI" "$INI.bak.$(date +%Y%m%d-%H%M%S)"

awk -v x="$X" -v y="$Y" '
BEGIN {
  in_vm = 0
  saw_w = saw_h = saw_ww = saw_wh = 0
}
# Detect section headers
/^\[VideoMode\][[:space:]]*$/ {
  in_vm = 1
  print
  next
}
/^\[[^]]+\][[:space:]]*$/ {
  # Leaving [VideoMode]: if we did not see keys, add them before next section
  if (in_vm) {
    if (!saw_w)  print "Width=" x
    if (!saw_h)  print "Height=" y
    if (!saw_ww) print "WindowedWidth=" x
    if (!saw_wh) print "WindowedHeight=" y
  }
  in_vm = 0
  print
  next
}

# While inside [VideoMode], replace or mark keys
in_vm && $0 ~ /^Width=/         { print "Width=" x;         saw_w=1;  next }
in_vm && $0 ~ /^Height=/        { print "Height=" y;        saw_h=1;  next }
in_vm && $0 ~ /^WindowedWidth=/ { print "WindowedWidth=" x; saw_ww=1; next }
in_vm && $0 ~ /^WindowedHeight=/{ print "WindowedHeight=" y;saw_wh=1; next }

# Otherwise, pass through unchanged
{ print }

END {
  # If file ended while still in [VideoMode], append missing keys at EOF
  if (in_vm) {
    if (!saw_w)  print "Width=" x
    if (!saw_h)  print "Height=" y
    if (!saw_ww) print "WindowedWidth=" x
    if (!saw_wh) print "WindowedHeight=" y
  }
}
' "$INI" > "$INI.tmp" && mv "$INI.tmp" "$INI"

echo "Updated: $INI"
echo "Backup:  $(ls -1t "$INI".bak.* | head -n 1)"
```

### Defining the repeatable process

You may have noticed that just above most of the code blocks in the repeatable process is the notation **(eq1 x N)** This is how many times that eq1 appears in the ensuing code block. To create additional eq prefixes (eq2 for example) you need to edit the existing eq1 to eq2 before you run the code. There are a couple of ways to do this:

1. Run the **REPEATABLE PREFIX INSTALLATION PROCESS** above. Paste the code as is from the guide to terminal, but before you press Enter: 
   - use the Left-arrow and Right-arrow keys to scroll non-destructively through the command and change each occurrence of eq1 to eq2. Use backspace or del to remove the 1 depending on where your cursor is, then type 2
   - After making the changes you can press Enter from anywhere in the command to execute it
2. (**Preferred method for lower risk**) Run the **REPEATABLE PREFIX INSTALLATION PROCESS** above. Paste the code as is from the guide into a text editor. I like kate (sudo apt install kate) for its notepad/notepad++ like interface. 
   - Edit each occurrence of eq1 to eq2 in the text editor. 
   - Copy the result and paste in terminal, then press Enter

3) or **Don't do any of that.** Be lazy and safe and just copy and paste from my prefabs for eq2 - eq4 installs. They are found directly below. If you want more than 4 installs, do method 1 or 2 above

#############################################################################

### Cheater!

**BEGIN eq2 PREFIX INSTALLATION PROCESS**

1. Create the prefix directory: 

```bash
mkdir -p ~/Games/EQAscendant/eq2
```

2. Initialize the Wine prefix:

```bash
WINEPREFIX=~/Games/EQAscendant/eq2 winecfg
```

- Accept prompts to install Wine Mono and/or Wine Gecko if offered. Wine may print warnings, noise or what even looks like errors while it's running — this is normal. The Wine configuration window should open **(success signal for this step)**.

3. In the Wine configuration window make these changes:

- Graphics tab: 
  - uncheck everything. 
  - Click **Apply**, then **OK**. 
- The wine configuration window will close.

4. Install DXVK into the eq2 prefix: 

```bash
WINEPREFIX=~/Games/EQAscendant/eq2 winetricks dxvk
```

- Non‑fatal warnings or fixme messages are expected as long as the command completes and returns terminal to the command prompt. **(success signal for this operation)**

5. Create the EverQuest install directory inside the prefix:

```bash
mkdir -p ~/Games/EQAscendant/eq2/drive_c/Program\ Files/eq2
```

6. Copy the EverQuest and patcher files into the prefix:

```bash
cp -a ~/Games/EQAscendant/EQ-game-files/. ~/Games/EQAscendant/eq2/drive_c/Program\ Files/eq2/
```

- **Verify the file copy & paste succeeded:** Confirm the copy completed without errors. Browse to ~/Games/EQAscendant/eq1/drive_c/Program\ Files/eq1/ using your file manager or terminal. Confirm the destination contains many files and subdirectories.

7. In this Step, we explicitly finalize first‑launch display containment before starting the game. This ensures a predictable, non‑disruptive first run. 

- Enable Wine Virtual Desktop: Open Wine configuration for this prefix:

```bash
WINEPREFIX=~/Games/EQAscendant/eq2 winecfg
```

- When the Wine configuration window opens **(success signal for this operation)**, make only the following changes:
- Graphics tab: 
  - Enable: ✅ Emulate a virtual desktop
  - Set the desktop size to 1366×768
  - Ensure all other boxes are unchecked
  - Click **Apply**, then **OK**. 
- The wine configuration window will close.

- This containment step prevents full screen rendering during DirectX initialization and avoids display mode switching while EverQuest establishes its video state.  

8. Install wine-mono: Wine-Mono is Wine’s open-source replacement for Microsoft’s .NET Framework. The EQAscendant patcher is a .NET application.

```bash
WINEPREFIX=~/Games/EQAscendant/eq2 wine msiexec /i ~/Downloads/wine-mono-11.0.0-x86.msi
```

- Success is indicated by the wine desktop being visible for a few seconds and terminal returning to the command prompt.

9. Launch the EQAscendant patcher, which will in turn launch EverQuest:

- In terminal, cd to the EverQuest install directory: 

```bash
cd ~/Games/EQAscendant/eq2/drive_c/Program\ Files/eq2
```

- Launch the EQAscendant patcher: 

```bash
WINEPREFIX=~/Games/EQAscendant/eq2 wine EQAscendant.exe
```

- Success is indicated by the EQAscendant patcher launching.

- Configure the patcher for hands‑off operation:

  - ✅ Check Auto Patch 

  - ✅ Check Auto Play 
    - On future launches, the patcher will automatically patch and start EverQuest without user interaction. 

  - click **Patch**

- After the patcher finishes patching, EverQuest will launch (**the success signal for this step**).
  -  Note: Always launch EverQuest via the EQAscendant patcher. 

- Login to the Ascendant server

  - To speed server login along, when the SOE splash screen pops, **click** it to move to the login screen. If you don't "click", the splash screen remains in place for 30 seconds before progressing to login.
- Create a test character if you don't already have a character
- Enter world, and look around
- OK, that's enough. Log out and close the game completely.
- ⚠️ Note that terminal is not presenting a command prompt. 
  - Click in terminal and press Ctrl+C to end the running process
  - You should now have a command prompt

10. Create a desktop launcher:

- Create the applications directory (if it doesn’t already exist).

```bash
mkdir -p ~/.local/share/applications
```

- Create the icons directory (if it doesn't already exist).

```bash
mkdir -p ~/.local/share/icons
```

- Copy the patcher icon to  ~/.local/share/icons.

```bash
sudo cp ~/Games/EQAscendant/eq2/drive_c/Program\ Files/eq2/eqemupatcher.png ~/.local/share/icons/eqascendant-eq2.png
```

- Create the launcher .desktop file:

```bash
cat > ~/.local/share/applications/EQAscendant-eq2.desktop <<'EOF'
[Desktop Entry]
Version=1.0
Type=Application
Name=EQAscendant-eq2
Comment=Launch EverQuest via EQAscendant using the eq2 Wine prefix
# Use a shell wrapper so quoting, spaces, and environment are handled reliably by all desktops
Exec=sh -c 'WINEPREFIX="$HOME/Games/EQAscendant/eq2" exec wine "$HOME/Games/EQAscendant/eq2/drive_c/Program Files/eq2/EQAscendant.exe"'
# Icon is resolved by name from ~/.local/share/icons
Icon=eqascendant-eq2
Terminal=false
Categories=Game;
EOF
chmod +x ~/.local/share/applications/EQAscendant-eq2.desktop
```

- Refresh desktop application database:

```bash
update-desktop-database ~/.local/share/applications
```

- The launcher will now appear in your desktop environment’s application menu (Games category). You can right‑click it to Add to Desktop or Pin to Panel. If your distro doesn't support adding to desktop from the menu, you can place a copy on your desktop by:

- Copy the launcher from /applications to /Desktop:

```bash
cp ~/.local/share/applications/EQAscendant-eq2.desktop ~/Desktop/EQAscendant-eq2.desktop
```

- This places a clickable EverQuest launcher directly on your desktop. 

**END eq2 PREFIX INSTALLATION PROCESS**

#############################################################################

**BEGIN eq3 PREFIX INSTALLATION PROCESS**

1. Create the prefix directory: 

```bash
mkdir -p ~/Games/EQAscendant/eq3
```

2. Initialize the Wine prefix:

```bash
WINEPREFIX=~/Games/EQAscendant/eq3 winecfg
```

- Accept prompts to install Wine Mono and/or Wine Gecko if offered. Wine may print warnings, noise or what even looks like errors while it's running — this is normal. The Wine configuration window should open **(success signal for this step)**.

3. In the Wine configuration window make these changes:

- Graphics tab: 
  - uncheck everything. 
  - Click **Apply**, then **OK**. 
- The wine configuration window will close.

4. Install DXVK into the prefix: 

```bash
WINEPREFIX=~/Games/EQAscendant/eq3 winetricks dxvk
```

- Non‑fatal warnings or fixme messages are expected as long as the command completes and returns terminal to the command prompt. **(success signal for this operation)**

5. Create the EverQuest install directory inside the prefix:

```bash
mkdir -p ~/Games/EQAscendant/eq3/drive_c/Program\ Files/eq3
```

6. Copy the EverQuest and patcher files into the prefix:

```bash
cp -a ~/Games/EQAscendant/EQ-game-files/. ~/Games/EQAscendant/eq3/drive_c/Program\ Files/eq3/
```

- **Verify the file copy & paste succeeded:** Confirm the copy completed without errors. Browse to ~/Games/EQAscendant/eq1/drive_c/Program\ Files/eq1/ using your file manager or terminal. Confirm the destination contains many files and subdirectories.

7. In this Step, we explicitly finalize first‑launch display containment before starting the game. This ensures a predictable, non‑disruptive first run. 

- Enable Wine Virtual Desktop: Open Wine configuration for this prefix:

```bash
WINEPREFIX=~/Games/EQAscendant/eq3 winecfg
```

- When the Wine configuration window opens **(success signal for this operation)**, make only the following changes:
- Graphics tab: 
  - Enable: ✅ Emulate a virtual desktop
  - Set the desktop size to 1366×768
  - Ensure all other boxes are unchecked
  - Click **Apply**, then **OK**. 
- The wine configuration window will close.

- This containment step prevents full screen rendering during DirectX initialization and avoids display mode switching while EverQuest establishes its video state.  

8. Install wine-mono: Wine-Mono is Wine’s open-source replacement for Microsoft’s .NET Framework. The EQAscendant patcher is a .NET application.

```bash
WINEPREFIX=~/Games/EQAscendant/eq3 wine msiexec /i ~/Downloads/wine-mono-11.0.0-x86.msi
```

- Success is indicated by the wine desktop being visible for a few seconds and terminal returning to the command prompt.

9. Launch the EQAscendant patcher, which will in turn launch EverQuest:

- In terminal, cd to the EverQuest install directory: 

```bash
cd ~/Games/EQAscendant/eq3/drive_c/Program\ Files/eq3
```

- Launch the EQAscendant patcher: 

```bash
WINEPREFIX=~/Games/EQAscendant/eq3 wine EQAscendant.exe
```

- Success is indicated by the EQAscendant patcher launching.

- Configure the patcher for hands‑off operation:

  - ✅ Check Auto Patch 

  - ✅ Check Auto Play 
    - On future launches, the patcher will automatically patch and start EverQuest without user interaction. 

  - click **Patch**

- After the patcher finishes patching, EverQuest will launch (**the success signal for this step**).
  -  Note: Always launch EverQuest via the EQAscendant patcher. 

- Login to the Ascendant server

  - To speed server login along, when the SOE splash screen pops, **click** it to move to the login screen. If you don't "click", the splash screen remains in place for 30 seconds before progressing to login.
- Create a test character if you don't already have a character
- Enter world, and look around
- OK, that's enough. Log out and close the game completely.
- ⚠️ Note that terminal is not presenting a command prompt. 
  - Click in terminal and press Ctrl+C to end the running process
  - You should now have a command prompt

10. Create a desktop launcher:

- Create the applications directory (if it doesn’t already exist).

```bash
mkdir -p ~/.local/share/applications
```

- Create the icons directory (if it doesn't already exist).

```bash
mkdir -p ~/.local/share/icons
```

- Copy the patcher icon to  ~/.local/share/icons.

```bash
sudo cp ~/Games/EQAscendant/eq2/drive_c/Program\ Files/eq2/eqemupatcher.png ~/.local/share/icons/eqascendant-eq2.png
```

- Create the launcher .desktop file:

```bash
cat > ~/.local/share/applications/EQAscendant-eq3.desktop <<'EOF'
[Desktop Entry]
Version=1.0
Type=Application
Name=EQAscendant-eq3
Comment=Launch EverQuest via EQAscendant using the eq3 Wine prefix
# Use a shell wrapper so quoting, spaces, and environment are handled reliably by all desktops
Exec=sh -c 'WINEPREFIX="$HOME/Games/EQAscendant/eq3" exec wine "$HOME/Games/EQAscendant/eq3/drive_c/Program Files/eq3/EQAscendant.exe"'
# Icon is resolved by name from ~/.local/share/icons
Icon=eqascendant-eq3
Terminal=false
Categories=Game;
EOF
chmod +x ~/.local/share/applications/EQAscendant-eq3.desktop
```

- Refresh desktop application database:

```bash
update-desktop-database ~/.local/share/applications
```

- The launcher will now appear in your desktop environment’s application menu (Games category). You can right‑click it to Add to Desktop or Pin to Panel. If your distro doesn't support adding to desktop from the menu, you can place a copy on your desktop by:

- Copy the launcher from /applications to /Desktop:

```bash
cp ~/.local/share/applications/EQAscendant-eq3.desktop ~/Desktop/EQAscendant-eq3.desktop
```

- This places a clickable EverQuest launcher directly on your desktop. 

1. **END eq3 PREFIX INSTALLATION PROCESS**

#############################################################################

**BEGIN eq4 PREFIX INSTALLATION PROCESS**

1. Create the prefix directory: 

```bash
mkdir -p ~/Games/EQAscendant/eq4
```

2. Initialize the Wine prefix:

```bash
WINEPREFIX=~/Games/EQAscendant/eq4 winecfg
```

- Accept prompts to install Wine Mono and/or Wine Gecko if offered. Wine may print warnings, noise or what even looks like errors while it's running — this is normal. The Wine configuration window should open **(success signal for this step)**.

3. In the Wine configuration window make these changes:

- Graphics tab: 
  - uncheck everything. 
  - Click **Apply**, then **OK**. 
- The wine configuration window will close.

4. Install DXVK into the eq2 prefix: 

```bash
WINEPREFIX=~/Games/EQAscendant/eq4 winetricks dxvk
```

- Non‑fatal warnings or fixme messages are expected as long as the command completes and returns terminal to the command prompt. **(success signal for this operation)**

5. Create the EverQuest install directory inside the prefix:

```bash
mkdir -p ~/Games/EQAscendant/eq4/drive_c/Program\ Files/eq4
```

6. Copy the EverQuest and patcher files into the prefix:

```bash
cp -a ~/Games/EQAscendant/EQ-game-files/. ~/Games/EQAscendant/eq4/drive_c/Program\ Files/eq4/
```

- **Verify the file copy & paste succeeded:** Confirm the copy completed without errors. Browse to ~/Games/EQAscendant/eq1/drive_c/Program\ Files/eq1/ using your file manager or terminal. Confirm the destination contains many files and subdirectories.

7. In this Step, we explicitly finalize first‑launch display containment before starting the game. This ensures a predictable, non‑disruptive first run. 

- Enable Wine Virtual Desktop: Open Wine configuration for this prefix:

```bash
WINEPREFIX=~/Games/EQAscendant/eq4 winecfg
```

- When the Wine configuration window opens **(success signal for this operation)**, make only the following changes:
- Graphics tab: 
  - Enable: ✅ Emulate a virtual desktop
  - Set the desktop size to 1366×768
  - Ensure all other boxes are unchecked
  - Click **Apply**, then **OK**. 
- The wine configuration window will close.

- This containment step prevents full screen rendering during DirectX initialization and avoids display mode switching while EverQuest establishes its video state.  

8. Install wine-mono: Wine-Mono is Wine’s open-source replacement for Microsoft’s .NET Framework. The EQAscendant patcher is a .NET application.

```bash
WINEPREFIX=~/Games/EQAscendant/eq4 wine msiexec /i ~/Downloads/wine-mono-11.0.0-x86.msi
```

- Success is indicated by the wine desktop being visible for a few seconds and terminal returning to the command prompt.

9. Launch the EQAscendant patcher, which will in turn launch EverQuest:

- In terminal, cd to the EverQuest install directory: 

```bash
cd ~/Games/EQAscendant/eq4/drive_c/Program\ Files/eq4
```

- Launch the EQAscendant patcher: 

```bash
WINEPREFIX=~/Games/EQAscendant/eq4 wine EQAscendant.exe
```

- Success is indicated by the EQAscendant patcher launching.

- Configure the patcher for hands‑off operation:

  - ✅ Check Auto Patch 

  - ✅ Check Auto Play 
    - On future launches, the patcher will automatically patch and start EverQuest without user interaction. 

  - click **Patch**

- After the patcher finishes patching, EverQuest will launch (**the success signal for this step**).
  -  Note: Always launch EverQuest via the EQAscendant patcher. 

- Login to the Ascendant server

  - To speed server login along, when the SOE splash screen pops, **click** it to move to the login screen. If you don't "click", the splash screen remains in place for 30 seconds before progressing to login.
- Create a test character if you don't already have a character
- Enter world, and look around
- OK, that's enough. Log out and close the game completely.
- ⚠️ Note that terminal is not presenting a command prompt. 
  - Click in terminal and press Ctrl+C to end the running process
  - You should now have a command prompt

10. Create a desktop launcher:

- Create the applications directory (if it doesn’t already exist).

```bash
mkdir -p ~/.local/share/applications
```

- Create the icons directory (if it doesn't already exist).

```bash
mkdir -p ~/.local/share/icons
```

- Copy the patcher icon to  ~/.local/share/icons.

```bash
sudo cp ~/Games/EQAscendant/eq4/drive_c/Program\ Files/eq4/eqemupatcher.png ~/.local/share/icons/eqascendant-eq4.png
```

- Create the launcher .desktop file:

```bash
cat > ~/.local/share/applications/EQAscendant-eq4.desktop <<'EOF'
[Desktop Entry]
Version=1.0
Type=Application
Name=EQAscendant-eq4
Comment=Launch EverQuest via EQAscendant using the eq4 Wine prefix
# Use a shell wrapper so quoting, spaces, and environment are handled reliably by all desktops
Exec=sh -c 'WINEPREFIX="$HOME/Games/EQAscendant/eq4" exec wine "$HOME/Games/EQAscendant/eq4/drive_c/Program Files/eq4/EQAscendant.exe"'
# Icon is resolved by name from ~/.local/share/icons
Icon=eqascendant-eq4
Terminal=false
Categories=Game;
EOF
chmod +x ~/.local/share/applications/EQAscendant-eq4.desktop
```

- Refresh desktop application database:

```bash
update-desktop-database ~/.local/share/applications
```

- The launcher will now appear in your desktop environment’s application menu (Games category). You can right‑click it to Add to Desktop or Pin to Panel. If your distro doesn't support adding to desktop from the menu, you can place a copy on your desktop by:

- Copy the launcher from /applications to /Desktop:

```bash
cp ~/.local/share/applications/EQAscendant-eq4.desktop ~/Desktop/EQAscendant-eq4.desktop
```

- This places a clickable EverQuest launcher directly on your desktop. 

1. **END eq4 PREFIX INSTALLATION PROCESS**

#############################################################################

## Appendix: Concepts and Terminology

This appendix provides short, practical explanations of key concepts and technologies referenced throughout this guide. The goal is not academic completeness, but enough context that you understand *what each piece is*, *why it exists*, and *why this guide uses it the way it does*.

### Wine

Wine is a compatibility layer that allows Windows applications to run on Linux. It is *not* an emulator. Instead of simulating a full Windows OS, Wine translates Windows system calls into native Linux calls in real time.

Why this matters:

- Wine is lightweight compared to virtual machines.
- Applications run close to native performance.
- Each application behaves according to how well Wine implements the Windows APIs it depends on.

In this guide, Wine is the foundation that allows EverQuest and the EQAscendant patcher to run on Linux at all.

### Wine Prefix

A Wine prefix is a self-contained directory that represents a *simulated Windows environment*. Each prefix contains:

- Its own registry
- Its own installed programs
- Its own C: drive
- Its own runtime components (DXVK, Wine Mono, Gecko, etc.)

Think of a Wine prefix as:

- A per-application Windows install
- A lightweight alternative to a VM snapshot

Why this guide uses one prefix per EverQuest client:

- Prevents shared state and file conflicts
- Allows multiple clients (boxing) to run simultaneously
- Makes uninstalling or rebuilding a client as simple as deleting a folder
- Eliminates “mystery breakage” caused by one application changing another’s environment

In short: one prefix = one EverQuest client = predictable behavior.

### 32‑bit vs 64‑bit (WoW64)

EverQuest (Rain of Fear era) is a 32‑bit Windows application. Modern Wine installations are typically 64‑bit Wine with WoW64 support, which means:

- Wine itself runs as 64‑bit
- 32‑bit Windows applications are fully supported inside the same prefix

You do *not* need a separate 32‑bit Wine installation. The guide assumes a standard modern Wine setup that supports both.

### DXVK

DXVK is a translation layer that converts DirectX 9/10/11 calls into Vulkan.

Why DXVK is used:

- EverQuest uses DirectX 9
- Wine’s builtin DirectX 9 implementation can be unstable or slow on modern systems
- DXVK provides: 
  - Better performance
  - Better GPU driver compatibility
  - More consistent fullscreen and windowed behavior

DXVK is installed per Wine prefix, not system-wide. This ensures:

- Each client gets the same rendering behavior
- Changes don’t affect other Wine applications

Without DXVK, EverQuest may still launch — but visual glitches, crashes, or erratic fullscreen behavior are far more likely.

### Vulkan

Vulkan is a modern, low-overhead graphics API supported by current GPUs and drivers (including NVIDIA, AMD, and Intel).

DXVK relies on Vulkan as its backend. If Vulkan is working correctly on your system:

- DXVK can operate efficiently
- The GPU driver, not Wine, does most of the heavy lifting

Your distribution’s graphics driver packages handle Vulkan support. This guide does not require any manual Vulkan configuration.

### Wine Mono

Wine Mono is Wine’s open-source replacement for Microsoft’s .NET Framework.

The EQAscendant patcher is a .NET application, which means:

- Wine Mono (or a real .NET runtime) must be present in the prefix
- Without it, Wine will emit errors like: `Wine Mono is not installed`

Why this guide installs Wine Mono manually:

- Wine does not always prompt to install Mono automatically
- The installer often runs silently
- Mono may not appear in `wine uninstaller` even when installed

By installing Wine Mono *before* first launch, the guide ensures deterministic behavior with no reliance on pop-ups or prompts.

### Wine Gecko

Wine Gecko provides an Internet Explorer–like HTML rendering engine inside Wine. It is mainly used by applications that embed web views.

EverQuest itself does not depend on Gecko. Some patchers and launchers may.

If Wine prompts to install Gecko during prefix creation, it is safe to allow it. Gecko does not interfere with EQ or DXVK.

### winetricks

winetricks is a helper tool that installs common Windows runtime components into a Wine prefix.

In this guide, winetricks is used *only* for:

- Installing DXVK

Why the guide keeps winetricks usage minimal:

- Reduces hidden side-effects
- Makes the prefix easier to reason about
- Avoids version mismatches between components

The fewer moving parts inside a prefix, the easier it is to debug and reproduce.

### Virtual Desktop (Wine)

Wine’s virtual desktop option runs Windows applications inside a fixed-size window instead of allowing them to take over the real display.

Why it is recommended for first launch:

- Prevents fullscreen mode switching during initial DirectX setup
- Avoids display reconfiguration glitches
- Makes first-time configuration safer and more predictable

You can disable the virtual desktop later if you prefer native window management.

### Boxing / Multi‑Client Setup

Boxing refers to running multiple EverQuest clients simultaneously.

This guide is explicitly designed to scale:

- Each client gets its own prefix (eq1, eq2, eq3, …)
- Each prefix is isolated
- Launchers map one‑to‑one with prefixes

This avoids the classic problems seen on both Windows and Wine:

- Shared config files
- Input conflicts
- Patchers modifying the wrong installation

### Why This Guide Is Opinionated

Many Linux/Wine guides present multiple paths and leave decisions to the reader. This guide intentionally does not.

The layout, prefix model, and install order are chosen to:

- Minimize undefined behavior
- Maximize repeatability
- Favor clarity over flexibility

Once you understand the process, you can deviate safely. Until then, following a single, consistent model produces the best results.

If you ever wonder *why* a step exists, it should now be answerable somewhere in this appendix.

##### 

