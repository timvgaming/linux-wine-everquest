# Installing EverQuest in Linux Wine

## *EQAscendant Version* 

**Author note & disclaimer:** I (Hrain on EQAscendant Discord) and (timvgaming on GitHub), do not serve in any official capacity with EQAscendant, nor do I speak for the project or its staff. I am simply an EverQuest player who enjoys this server, and is sharing what I’ve learned to help other players. All configuration guidance here reflects personal experience, not official project policy. <span id="toc"></span>

## Table of Contents

\- [Guide Goals](#guidegoals) 

\- [Technical Notes](#technotes) 

\- [Installation Overview](#overview) 

\- [Prerequisites](#prerequisites)

\- [How to Copy & Paste to Terminal](#copypaste) 

\- [One-time Host-level Software Stack Installation](#onetimehost) 

\- [One-time Wine Prefix Configuration](#onetimewine) 

\- [Begin Repeatable Prefix Installation Process](#beginrepeat) 

\- [Create Prefix eq1](#prefixeq1) 

\- [Create Prefix eq2](#prefixeq2) 

\- [Create Prefix eq3](#prefixeq3) 

\- [Create Prefix eq4](#prefixeq4) 

\-  [EverQuest Window Resolutions](#windowres) 

\-  [Defining the Repeatable Wine Prefix Process](#definerepeat) 

\-  [Appendix: Concepts and Terminology](#appendix) 

​		[Wine](#Wine) is a compatibility layer that allows Windows applications to run on Linux.

​		A  [Wine Prefix](#wineprefix) is a self-contained directory that represents a *simulated Windows environment*.

​		Wine’s [Virtual Desktop](#virtdt) option runs Windows applications inside a fixed-size window.

​		Understanding  [32‑bit vs 64‑bit (WoW64)](#wow64)

​		[DXVK](#DXVK)  is a translation layer that converts DirectX 9/10/11 calls into Vulkan.

​		[Vulkan](#Vulkan) is a graphics API.

​		[Wine-Mono](#Wine-Mono) provides .NET application support.

​		[Wine-Gecko](#Wine-Gecko) provides Internet Explorer–like HTML rendering.

​		[winetricks](#winetricks) is a helper tool that installs common Windows runtime components.

​		[Boxing](#boxing)  refers to running multiple EverQuest, and is supported by this guide.

​		Why This Guide Is [Opinionated](#opion).

------

<a id="guidegoals"></a>

## Guide Goals 

This guide will hopefully get you up and running with one or more EverQuest clients on Linux using Wine, while giving you a practical understanding of the pieces and processes involved. While this guide is specifically constructed and worded to support playing on the EQAscendant EMU server, it will support other EMUs with the differences being the EMU specific EverQuest game files (typically Rain of Fear or Titanium) and the EMU specific patcher files. Both of those are only referenced in one section of the guide and are **user placed** in a specific folder. All terminal commands and processes in the guide are not specific to any particular EMU with the exception of the patcher launch and .desktop shortcuts setup, where it is assumed that your patchername.exe launches EverQuest automatically. If it doesn't do that, then the remainder of the guide won't apply to you. I may append the guide later to support that case. Principle goals with links to any applicable Appendix expanded descriptions are:

- ⚠️ Launch and play EverQuest via the EQAscendant patcher. 
- [Wine](#Wine) is a compatibility layer that allows Windows applications to run on Linux.
- A  [Wine Prefix](#wineprefix) is a self-contained directory that represents a *simulated Windows environment*.
- Wine’s [Virtual Desktop](#virtdt) option runs Windows applications inside a fixed-size window.
- Understanding  [32‑bit vs 64‑bit (WoW64)](#wow64)
- [DXVK](#DXVK)  is a translation layer that converts DirectX 9/10/11 calls into Vulkan.
- [Vulkan](#Vulkan) is a graphics API.
- [Wine-Mono](#Wine-Mono) provides .NET application support.
- [Wine-Gecko](#Wine-Gecko) provides Internet Explorer–like HTML rendering.
- [winetricks](#winetricks) is a helper tool that installs common Windows runtime components.
- [Boxing](#boxing)  refers to running multiple EverQuest clients simultaneously, and is supported by this guide.
- Why This Guide Is [Opinionated](#opion) .

[ToC](#toc)

------

<a id="technotes"></a>

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

[ToC](#toc)

------

<span id="overview"></span>

## Installation Overview ## 

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

[ToC](#toc)

------

## Prerequisites ##

⚠️ **Critical Safety Note**

This guide assumes EverQuest is always run inside a Wine virtual desktop. Running EverQuest outside a virtual desktop can cause unrecoverable fullscreen display failures requiring a hard reboot.

- Fullscreen inside the Wine virtual desktop is safe and recommended.
- Windowed mode inside the virtual desktop is what causes most accidental breakage.

- If you are not willing to run inside the virtual desktop, stop here.

**EverQuest game and patcher files**

Before installing any software or creating Wine prefixes, you must already have the following game‑specific assets. If you cannot locate these, you should **stop here** — the remaining steps depend on them.

1. A legally obtained EverQuest client (Rain of Fear era) that you are licensed to use:
   - Existing Rain of Fear client directories from a prior Windows or Linux installation may be reused. 
   - A web search on "download everquest rof" or "Getting Started on Addicted Dads" will yield some sources.
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

1. EQAscendant (at the time of this writing) allows 3 clients out in the world killing stuff, and an additional client in the Bazaar selling your **Phat Lewts⚠️ **. For each EverQuest instance that you intend to run simultaneously you will need a login server account. To get login server accounts:
   1. If you don't have one, create an EQEmulator account at:
      - https://www.eqemulator.org/

   2. Login to your EQEmulator account and:
      - On the left side of the window you should see **Loginserver Accounts**. Beneath that:
      - Click **Create Account**, and follow the prompts to create a login server account.
      - I think they have a limit on how many accounts you can create in a day. 
      - You can use any existing accounts that you may have made for other EverQuest EMUs. They are not EMU exclusive.
      - You will need at least **1 login server account** (❌ not your EQEmulator account) to complete the EQ install below

[ToC](#toc)

------

<span id="copypaste"></span>

## **How to copy & paste to terminal:**

For the new and uninitiated Linux user, that was me a few weeks ago, the prospects of needing to use terminal can be a bit daunting. Rest assured that you are not going to have to learn any commands here, though that could be a nice side benefit of following this guide. Any step requiring the use of terminal (by the way you can open terminal with Ctrl+Alt+T) will be a simple matter of copying the command from the guide and pasting it into terminal and then pressing Enter.

A quick primer on copying commands from this guide and pasting into terminal. You'll be doing a lot of that soon. Copying is the intuitive part. In the code block example below, you'll see a Copy button at the far right of the block. **Click** the button and the code block contents are placed in your clipboard.

```bash
some cryptic terminal command -r whodat reXing my system
```

Pasting into your terminal may not be as intuitive. 1) Clicking anywhere in terminal, then pressing Ctrl+Shift+V should paste the clipboard contents into terminal, or 2) Clicking the Right mouse button anywhere in terminal should display a context menu that includes a paste option. Both options will paste the clipboard contents at the command prompt, and then you just press Enter. 

[ToC](#toc)

------

<span id="onetimehost"></span>

## One-time host-level software stack installation

These components form the baseline environment required to get EverQuest up and running. **Important guardrail (package sources):** Whenever possible, use your distro’s managed packages first (e.g., Linux Mint Software Manager / Driver Manager). Only fall back to command‑line installs (APT) when the software is not available or is materially outdated in the managed repositories. This minimizes dependency conflicts and keeps upgrades clean and supportable. Before configuring a Wine prefix or installing EverQuest, ensure the following **host‑level software** is installed on your system:

**Source order used in this guide:**

1. Distro Software/Driver Manager (preferred) 
2. Distro APT repositories (fallback) 
3. Upstream installers (used only when necessary and called out explicitly) 

#### Host-level software stack Installation procedures

**A)  GPU driver:** No standalone driver installation commands or version pinning are required for this guide. Use your distro Driver Manager to select and maintain your GPU driver.

1) Review the drivers offered for your GPU. 

- Either: Select the recommended driver and apply it, or 

- Keep your current driver if it is already working well and you are satisfied with it.

**B) Wine:** Wine is the compatibility layer that allows Windows applications to run on Linux.

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

[ToC](#toc)

------

<span id="onetimewine"></span>

## One-time Wine Prefix Configuration  ##

Before running any commands, it’s important to understand exactly how this guide organizes files on disk. The layout below is not only a recommendation — it is the structure this guide **will** use when installing EverQuest on your system if you follow the guide verbatim. Using your own preferred layout is discussed at the bottom of this section.

**Goals of using this layout:**

- Folder organization
- One Wine prefix per EverQuest client 
- No shared state between clients 
- Clear, readable paths that scale cleanly for boxing 
- Easy cleanup, deletion, backup, and troubleshooting 

**Canonical directory tree** 

⚠️ **Note**: The full Windows file system is shown only under `eq1` for clarity; the other `eqN` prefixes contain the same structure.


```text
~/Games/EQAscendant/  # Root folder, 1st half of all Wine prefixes (Created by user)
├── eq1/              # Unique 2nd half of a Wine prefix (Created by user)
│  └── drive_c/                 # p/o the Windows file system (Created by Wine)
│    └── Program Data/          # p/o the Windows file system (Created by Wine)
│    └── users/                 # p/o the Windows file system (Created by Wine)
│    └── windows/               # p/o the Windows file system (Created by Wine)
│    └── Program Files (x86)/   # p/o the Windows file system (Created by Wine)
│    └── Program Files/         # p/o the Windows file system (Created by Wine)
│      └── eq1/       # EverQuest game folder (Created by user)
├── eq2/              # Unique 2nd half of a Wine prefix (Created by user)
│  └── drive_c/                 # p/o the Windows file system (Created by Wine)
│    └── Program Files/         # p/o the Windows file system (Created by Wine)
│      └── eq2/       # EverQuest game folder (Created by user)
├── eq3/              # Unique 2nd half of a Wine prefix (Created by user)
│  └── drive_c/                 # p/o the Windows file system (Created by Wine)
│    └── Program Files/         # p/o the Windows file system (Created by Wine)
│      └── eq3/       # EverQuest game folder (Created by user)
├── eq4/              # Unique 2nd half of a Wine prefix (Created by user)
│  └── drive_c/                 # p/o the Windows file system (Created by Wine)
│    └── Program Files/         # p/o the Windows file system (Created by Wine)
│      └── eq4/       # EverQuest game folder (Created by user)
└── EQ-game-files/    # Staging folder for EQ game files (Created by user)
```

How to read the **Canonical directory tree** :

- ~/Games/EQAscendant/ is the root folder for all Wine prefixes and EverQuest installs. It is created by the user.
- Each eqN/ prefix folder is part of a unique, isolated Wine prefix. They are created by the user. This guide shows 4 eqN prefix folders. You can create as many as you like. Each complete and unique Wine prefix becomes (root folder + eqN prefix) as shown below:
  - ~/Games/EQAscendant/eq1
  - ~/Games/EQAscendant/eq2
  - ~/Games/EQAscendant/eq3
  - ~/Games/EQAscendant/eq4

- Wine will install the Windows file system into each prefix. The entire Windows file system is only shown above in prefix eq1, but exists in all eqN prefixes. The part we care about particularly is the Program Files folder. The complete path to Program Files/ is (unique wine prefix + Windows file system) as shown below:
  - ~/Games/EQAscendant/eq1/drive_c/Program Files/
  - ~/Games/EQAscendant/eq2/drive_c/Program Files/
  - ~/Games/EQAscendant/eq3/drive_c/Program Files/
  - ~/Games/EQAscendant/eq4/drive_c/Program Files/


⚠️ **Note**: Wine owns everything in the Windows file system.  ❌ Don't modify, rename or delete any part of it.

- Each eqN EverQuest game folder is created by the user after Wine installs the Windows file system. The complete path to a eqN game folder becomes  (unique wine prefix + Windows file system + eqN) as shown below:
  - ~/Games/EQAscendant/eq1/drive_c/Program Files/eq1/
  - ~/Games/EQAscendant/eq2/drive_c/Program Files/eq2/
  - ~/Games/EQAscendant/eq3/drive_c/Program Files/eq3/
  - ~/Games/EQAscendant/eq4/drive_c/Program Files/eq4/

- There is no shared Program Files/EverQuest directory. The one‑to‑one mapping (Unique Wine prefix ↔ matching EverQuest game folder) described above, lends itself to understanding your Wine/EverQuest directory structure. In technical benefits, it prevents cross‑contamination, makes boxing predictable, and ensures uninstalling a client is as simple as deleting its eqN/ directory. 
- EQ-game-files/ is a staging folder created by the user, and is used to hold clean copies of the EverQuest game files and the EQAscendant patcher; its contents are copied into the EverQuest game folders during installs and ❌ **are never run directly**. This ensures the same known good source files are used for each of your EverQuest installs.

#### Using your own layout design 

- ⚠️ Please use the layout described above for the purpose of getting through this guide with the provided copy and paste commands. 

- After you have gone through this guide and are comfortable with the process you can create your EQ installs using the example shown and described above and replacing the root folder, Wine prefixes and EQ game folders with your preferred layout. Uninstalling an existing Wine prefix and the associated EQ game folder is a simple matter of:

  - Delete the prefix folder (or any higher level folder in your home directory). None of the previously installed host-level software packages are effected. No part of a Wine prefix lives outside of the prefix folder. Deleting it is a targeted total annihilation and the ultimate uninstall process. 

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

#### Pre‑configure EverQuest client.ini

EverQuest will soon be launching for the first time inside a Wine prefix. To ensure a predictable, accessible first launch—and to standardize behavior across this prefix and any future prefixes—we will edit eqclient.ini in the staging folder with first‑launch settings to define known window parameters. You are free to edit the eqclient.ini in your eq1 game folder per your usual preferences once you've logged in and out one time.

1. Edit the staging folder eqclient.ini file: 

```bash
# Edit EQ eqclient.ini: enforce safe fullscreen-in-virtual-desktop defaults

X=1024
Y=768

INI="$HOME/Games/EQAscendant/EQ-game-files/eqclient.ini"

# Safety backup
cp -a "$INI" "$INI.bak.$(date +%Y%m%d-%H%M%S)"

awk -v x="$X" -v y="$Y" '
BEGIN {
  in_vm = in_def = in_opt = 0

  saw_w = saw_h = saw_ww = saw_wh = 0

  saw_ar = saw_max = saw_xo = saw_yo = 0
  saw_rx = saw_ry = saw_rw = saw_rh = 0
  saw_wm = saw_gamma = 0

  saw_fps = 0
}

# Section headers
/^\[VideoMode\][[:space:]]*$/ { in_vm=1; in_def=in_opt=0; print; next }
/^\[Defaults\][[:space:]]*$/  { in_def=1; in_vm=in_opt=0; print; next }
/^\[Options\][[:space:]]*$/   { in_opt=1; in_vm=in_def=0; print; next }

/^\[[^]]+\][[:space:]]*$/ {
  if (in_vm) {
    if (!saw_w)  print "Width=" x
    if (!saw_h)  print "Height=" y
    if (!saw_ww) print "WindowedWidth=" x
    if (!saw_wh) print "WindowedHeight=" y
  }

  if (in_def) {
    if (!saw_ar) print "AllowResize=1"
    if (!saw_max) print "Maximized=0"
    if (!saw_xo) print "WindowedModeXOffset=0"
    if (!saw_yo) print "WindowedModeYOffset=0"
    if (!saw_rx) print "RestoredXOffset=" x
    if (!saw_ry) print "RestoredYOffset=" y
    if (!saw_rw) print "RestoredWidth=" x
    if (!saw_rh) print "RestoredHeight=" y
    if (!saw_wm) print "WindowedMode=TRUE"
    if (!saw_gamma) print "Gamma=4"
  }

  if (in_opt && !saw_fps) {
    print "MaxFPS=60"
  }

  in_vm=in_def=in_opt=0
  print
  next
}

# --- [VideoMode]
in_vm && /^Width=/         { print "Width=" x;         saw_w=1;  next }
in_vm && /^Height=/        { print "Height=" y;        saw_h=1;  next }
in_vm && /^WindowedWidth=/ { print "WindowedWidth=" x; saw_ww=1; next }
in_vm && /^WindowedHeight=/{ print "WindowedHeight=" y;saw_wh=1; next }

# --- [Defaults]
in_def && /^AllowResize=/          { print "AllowResize=1"; saw_ar=1; next }
in_def && /^Maximized=/            { print "Maximized=0"; saw_max=1; next }
in_def && /^WindowedModeXOffset=/  { print "WindowedModeXOffset=0"; saw_xo=1; next }
in_def && /^WindowedModeYOffset=/  { print "WindowedModeYOffset=0"; saw_yo=1; next }
in_def && /^RestoredXOffset=/      { print "RestoredXOffset=" x; saw_rx=1; next }
in_def && /^RestoredYOffset=/      { print "RestoredYOffset=" y; saw_ry=1; next }
in_def && /^RestoredWidth=/        { print "RestoredWidth=" x; saw_rw=1; next }
in_def && /^RestoredHeight=/       { print "RestoredHeight=" y; saw_rh=1; next }
in_def && /^WindowedMode=/         { print "WindowedMode=TRUE"; saw_wm=1; next }
in_def && /^Gamma=/                { print "Gamma=4"; saw_gamma=1; next }

# --- [Options]
in_opt && /^MaxFPS=/ { print "MaxFPS=60"; saw_fps=1; next }

# pass-through
{ print }

END {
  if (in_vm) {
    if (!saw_w)  print "Width=" x
    if (!saw_h)  print "Height=" y
    if (!saw_ww) print "WindowedWidth=" x
    if (!saw_wh) print "WindowedHeight=" y
  }

  if (in_def) {
    if (!saw_ar) print "AllowResize=1"
    if (!saw_max) print "Maximized=0"
    if (!saw_xo) print "WindowedModeXOffset=0"
    if (!saw_yo) print "WindowedModeYOffset=0"
    if (!saw_rx) print "RestoredXOffset=" x
    if (!saw_ry) print "RestoredYOffset=" y
    if (!saw_rw) print "RestoredWidth=" x
    if (!saw_rh) print "RestoredHeight=" y
    if (!saw_wm) print "WindowedMode=TRUE"
    if (!saw_gamma) print "Gamma=4"
  }

  if (in_opt && !saw_fps) {
    print "MaxFPS=60"
  }
}
' "$INI" > "$INI.tmp" && mv "$INI.tmp" "$INI"

echo "Updated: $INI"
echo "Backup:  $(ls -1t "$INI".bak.* | head -n 1)"
```

- Gama is set low. A low initial gamma (such as Gamma=4, approximately 15% in‑game) mitigates display gamma bleed into the desktop environment under Wine while still providing a comfortable baseline. You can fine‑tune gamma and dimensions later using the in‑game options. 

- This creates a clean, deterministic baseline for the initial launch. No further editing is required at this stage. At this point, the EQ-game-files/ folder contains a clean, reproducible EverQuest + EQAscendant baseline. 

[ToC](#toc)

<span id="beginrepeat"></span>

------

## **BEGIN REPEATABLE PREFIX INSTALLATION PROCESS**

Everything you did above set the stage and never has to be done again. From here on, we build one EverQuest client at a time, each isolated in its own Wine prefix. The steps that follow focus on layout and prefix creation first, installing the game and patcher into that prefix, verifying a good launch and creating a desktop launcher.  Each EverQuest client lives in its own Wine prefix. ⚠️ **Here's the first one!**:

<span id="prefixeq1"></span>

### Create PREFIX eq1

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

- **Optional:** Verify the file copy & paste succeeded: Confirm the copy completed without errors. Browse to ~/Games/EQAscendant/eq1/drive_c/Program\ Files/eq1/ using your file manager or terminal. Confirm the destination contains many files and subdirectories.

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
  - Ensure the Automatically capture the mouse in full-screen windows box is unchecked
  - Set the desktop size to 1024×768
  - Click **Apply**, then **OK**. 
- The wine configuration window will close.

- This containment step prevents full screen rendering during DirectX initialization and avoids display mode switching while EverQuest establishes its video state.  

8. Install wine-mono: Wine-Mono is Wine’s open-source replacement for Microsoft’s .NET Framework. The EQAscendant patcher is a .NET application.  **(eq1 x 1)** 

```bash
WINEPREFIX=~/Games/EQAscendant/eq1 wine msiexec /i ~/Downloads/wine-mono-11.0.0-x86.msi
```

- Success is indicated by the wine desktop being visible for a few seconds and terminal returning to the command prompt.

9. Launch the EQAscendant patcher, which will in turn launch EverQuest:

- In terminal, cd to the EverQuest install directory: **(eq1 x 2)**

```bash
cd ~/Games/EQAscendant/eq1/drive_c/Program\ Files/eq1
```

⚠️ **Note**: If you are following this process for another EMU, edit EQAscendant.exe in the following command to your specific patcher.exe file name. (1 place) Your patcher.exe must auto-launch EverQuest, or the remainder of this guide will ❌ **not work for you**.

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


- After the patcher finishes patching, EverQuest will launch inside the Wine 1024×600 virtual desktop. (**the success signal for this step**).

⚠️ Note: Always launch EverQuest via the EQAscendant patcher. 


- Login to the EQAscendant server

  - To speed server login along, when the SOE splash screen pops, **click** it to move to the login screen. If you don't "click", the splash screen remains in place for 30 seconds before progressing to login.

- Create a test character if you don't already have a character
- Enter world, and look around
- OK, that's enough. Back to work. Open the in-game Options
- **⚠️ Important:**

  - Click the Display tab. On the upper left side of Display options, there is a button that will either read "Switch to Windowed" or "Switch to Fullscreen". If it reads "Switch to Windowed", do nothing. If it reads "Switch to Fullscreen" then click it. 
  - The button should now read "Switch to Windowed". As you play the game and fiddle with options, never click the button when it reads "Switch to Windowed". You must always be in Fullscreen mode.
- Log out and close the game completely. ❌ **Do not proceed** until the game and Wine desktop have completely closed.

⚠️ Note that terminal is not presenting a command prompt. 

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

- Copy the patcher icon to  ~/.local/share/icons.

⚠️ **Note**: If you are following this process for another EMU, edit eqemupatcher.png in the following command to your specific patcher icon file name. (1 place)

- ⚠️ This is our first use of "sudo", so you'll have to enter your password after entering the following command: **(eq1 x 3)**

```bash
sudo cp ~/Games/EQAscendant/eq1/drive_c/Program\ Files/eq1/eqemupatcher.png ~/.local/share/icons/eqascendant-eq1.png
```

⚠️ **Note**: If you are following this process for another EMU, edit EQAscendant.exe in the following command to your specific patcher.exe file name. (1 place)

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

- Refresh the desktop application database:

```bash
update-desktop-database ~/.local/share/applications
```

- The launcher will now appear in your desktop environment’s application menu (Games category). You can right‑click it to Add to Desktop or Pin to Panel. If your distro doesn't support adding to desktop from the menu, you can place a copy on your desktop by:

- Copy the launcher from /applications to /Desktop: **(eq1 x 2)**

```bash
cp ~/.local/share/applications/EQAscendant-eq1.desktop ~/Desktop/EQAscendant-eq1.desktop
```

- This places a clickable EverQuest launcher directly on your desktop. 

- Final Notes:

  - You can edit the eqclient.ini file with nano (default terminal based text editor for most distro)  **(eq1 x 2)**

  ```bash
  nano ~/Games/EQAscendant/eq1/drive_c/Program\ Files/eq1/eqclient.ini
  ```

  - or edit with kate (desktop editor similar to notepad++) install with: sudo apt install kate  **(eq1 x 2)**

  ```bash
  kate ~/Games/EQAscendant/eq1/drive_c/Program\ Files/eq1/eqclient.ini
  ```

  - or use your preferred text editor

#### **⚠️ Note: EverQuest (eq1) is yours to play now. Recommended reading is:**

-  [Defining the Repeatable Wine Prefix Process](#definerepeat) if you want to install additional EverQuest clients.
-  [EverQuest Window Resolutions](#windowres) if you want to see some typical EverQuest Video Modes settings.

[ToC](#toc)

------

<span id="windowres"></span>

### EverQuest Window Resolutions

When using Wine’s virtual desktop and running EverQuest in full screen mode, it is often helpful to choose resolutions that are *smaller than your native monitor resolution*. This improves usability when boxing, and reduces GPU overhead.

The table below lists common monitor resolutions and several descending, practical window sizes that work well for EverQuest client windows.

These are *recommendations*, not requirements. Feel free to experiment once your setup is stable. Use the in-game setting (Options>Display>Video Modes) to choose from the games' listed resolutions. The Wine desktop will resize based on your in-game selection as long as your are set to full screen in-game

| Your Monitor (Native) | Large     | Medium (Good Default) | Small (Boxing‑Friendly) | Very Small (Utility/Box) |
| --------------------- | --------- | --------------------- | ----------------------- | ------------------------ |
| 3840×2160 (4K)        | 2560×1440 | 1920×1080             | 1600×900                | 1280×720                 |
| 2560×1440 (1440p)     | 1920×1080 | 1600×900              | 1280×720                | 1152×864                 |
| 1920×1080 (1080p)     | 1600×900  | 1400×1050             | 1152×864                | 1024×600                 |
| 1680×1050             | 1440×900  | 1280×720              | 1024×600                | 768×576                  |
| 1366×768              | 1280×720  | 1024×768              | 768×576                 | 640×80                   |

Guidance:

- 1024×600 was chosen as the first-launch resolution to fit most monitor sizes.
- The EverQuest UI scales better when resolutions follow standard 16:9 or 16:10 ratios.
- You can use *different resolutions per client* when boxing.

[ToC](#toc)

------

<span id="definerepeat"></span>

### Defining the Repeatable Wine Prefix Process

You may have noticed that just above most of the code blocks in the repeatable process is the notation **(eq1 x N)** This is how many times that eq1 appears in the ensuing code block. To create additional eq prefixes (eq2 for example) you need to edit the existing eq1 to eq2 before you run the code. There are a couple of ways to do this:

1. Run the **REPEATABLE PREFIX INSTALLATION PROCESS** above. Paste the code as is from the guide to terminal, but before you press Enter: 
   - use the Left-arrow and Right-arrow keys to scroll non-destructively through the command and change each occurrence of eq1 to eq2. Use backspace or del to remove the 1 depending on where your cursor is, then type 2
   - After making the changes you can press Enter from anywhere in the command to execute it
2. (**Preferred method for lower risk**) Run the **REPEATABLE PREFIX INSTALLATION PROCESS** above. Paste the code as is from the guide into a text editor. I like kate for its desktop notepad++ like interface. Install with: sudo apt install kate 
   - Edit each occurrence of eq1 to eq2 in the text editor. 
   - Copy the result and paste in terminal, then press Enter

3) or **Don't do any of that.** Be lazy and safe and just copy and paste from my prefabs for eq2 - eq4 installs. They are found directly below. If you want more than 4 installs, do method 1 or 2 above

[ToC](#toc) - [Create Prefix eq2](#prefixeq2)  - [Create Prefix eq3](#prefixeq3) - [Create Prefix eq4](#prefixeq4) 

------

<span id="prefixeq1"></span>

### Create PREFIX eq2

1. ⚠️ Quick Note: The annotation **(eq2 x N)** is a count of the eq2 occurrences in the code block. More on that later.

1. Create the prefix directory: **(eq2 x 1)**

```bash
mkdir -p ~/Games/EQAscendant/eq2
```

2. Initialize the Wine prefix: **(eq2 x 1)**

```bash
WINEPREFIX=~/Games/EQAscendant/eq2 winecfg
```

- Accept prompts to install Wine Mono and/or Wine Gecko if offered. Wine may print warnings, noise or what even looks like errors while it's running — this is normal. The Wine configuration window should open **(success signal for this step)**.

3. In the Wine configuration window make these changes:

- Graphics tab: 
  - uncheck everything. 
  - Click **Apply**, then **OK**. 
- The wine configuration window will close.

4. Install DXVK into the eq2 prefix: **(eq2 x 1)**

```bash
WINEPREFIX=~/Games/EQAscendant/eq2 winetricks dxvk
```

- Non‑fatal warnings or fixme messages are expected as long as the command completes and returns terminal to the command prompt. **(success signal for this operation)**

5. Create the EverQuest install directory inside the prefix:  **(eq2 x 2)**

```bash
mkdir -p ~/Games/EQAscendant/eq2/drive_c/Program\ Files/eq2
```

6. Copy the EverQuest and patcher files into the eq2 prefix: **(eq2 x 2)**

```bash
cp -a ~/Games/EQAscendant/EQ-game-files/. ~/Games/EQAscendant/eq2/drive_c/Program\ Files/eq2/
```

- **Optional:** Verify the file copy & paste succeeded: Confirm the copy completed without errors. Browse to ~/Games/EQAscendant/eq2/drive_c/Program\ Files/eq2/ using your file manager or terminal. Confirm the destination contains many files and subdirectories.

7. In this Step, we explicitly finalize first‑launch display containment before starting the game. This ensures a predictable, non‑disruptive first run. 

- Enable Wine Virtual Desktop: Open Wine configuration for this prefix: **(eq2 x 1)**

```bash
WINEPREFIX=~/Games/EQAscendant/eq2 winecfg
```

- When the Wine configuration window opens **(success signal for this operation)**, make the following changes:

- Applications tab:
  - Click Add application

  - Navigate to the eq2 folder (Program Files/eq2), select eqgame.exe and click Open

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
  - Ensure the Automatically capture the mouse in full-screen windows box is unchecked
  - Set the desktop size to 1024×768
  - Click **Apply**, then **OK**. 
- The wine configuration window will close.

- This containment step prevents full screen rendering during DirectX initialization and avoids display mode switching while EverQuest establishes its video state.  

8. Install wine-mono: Wine-Mono is Wine’s open-source replacement for Microsoft’s .NET Framework. The EQAscendant patcher is a .NET application.  **(eq2 x 1)** 

```bash
WINEPREFIX=~/Games/EQAscendant/eq2 wine msiexec /i ~/Downloads/wine-mono-11.0.0-x86.msi
```

- Success is indicated by the wine desktop being visible for a few seconds and terminal returning to the command prompt.

9. Launch the EQAscendant patcher, which will in turn launch EverQuest:

- In terminal, cd to the EverQuest install directory: **(eq1 x 2)**

```bash
cd ~/Games/EQAscendant/eq2/drive_c/Program\ Files/eq2
```

⚠️ **Note**: If you are following this process for another EMU, edit EQAscendant.exe in the following command to your specific patcher.exe file name. (1 place) Your patcher.exe must auto-launch EverQuest, or the remainder of this guide will ❌ **not work for you**.

- Launch the EQAscendant patcher: **(eq1 x 1)**

```bash
WINEPREFIX=~/Games/EQAscendant/eq2 wine EQAscendant.exe
```

- Success is indicated by the EQAscendant patcher launching.

- Configure the patcher for hands‑off operation:

  - ✅ Check Auto Patch 

  - ✅ Check Auto Play 
    - On future launches, the patcher will automatically patch and start EverQuest without user interaction. 

  - click **Patch**


- After the patcher finishes patching, EverQuest will launch inside the Wine 1024×600 virtual desktop. (**the success signal for this step**).

⚠️ Note: Always launch EverQuest via the EQAscendant patcher. 


- Login to the EQAscendant server

  - To speed server login along, when the SOE splash screen pops, **click** it to move to the login screen. If you don't "click", the splash screen remains in place for 30 seconds before progressing to login.

- Create a test character if you don't already have a character
- Enter world, and look around
- OK, that's enough. Back to work. Open the in-game Options
- **⚠️ Important:**

  - Click the Display tab. On the upper left side of Display options, there is a button that will either read "Switch to Windowed" or "Switch to Fullscreen". If it reads "Switch to Windowed", do nothing. If it reads "Switch to Fullscreen" then click it. 
  - The button should now read "Switch to Windowed". As you play the game and fiddle with options, never click the button when it reads "Switch to Windowed". You must always be in Fullscreen mode.
- Log out and close the game completely. ❌ **Do not proceed** until the game and Wine desktop have completely closed.

⚠️ Note that terminal is not presenting a command prompt. 

- Click in terminal and press Ctrl+C to end the running process
- You should now have a command prompt


10. Create a desktop launcher (eq2). 

- Create the applications directory (it's ok if it already exists) .

```bash
mkdir -p ~/.local/share/applications
```

- Create the icons directory (it's ok if it already exists).

```bash
mkdir -p ~/.local/share/icons
```

- Copy the patcher icon to  ~/.local/share/icons.

⚠️ **Note**: If you are following this process for another EMU, edit eqemupatcher.png in the following command to your specific patcher icon file name. (1 place)

- ⚠️ This is our first use of "sudo", so you'll have to enter your password after entering the following command: **(eq1 x 3)**

```bash
sudo cp ~/Games/EQAscendant/eq2/drive_c/Program\ Files/eq2/eqemupatcher.png ~/.local/share/icons/eqascendant-eq2.png
```

⚠️ **Note**: If you are following this process for another EMU, edit EQAscendant.exe in the following command to your specific patcher.exe file name. (1 place)

- Create the launcher .desktop file: **(eq1 x 8)**

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

- Refresh the desktop application database:

```bash
update-desktop-database ~/.local/share/applications
```

- The launcher will now appear in your desktop environment’s application menu (Games category). You can right‑click it to Add to Desktop or Pin to Panel. If your distro doesn't support adding to desktop from the menu, you can place a copy on your desktop by:

- Copy the launcher from /applications to /Desktop: **(eq1 x 2)**

```bash
cp ~/.local/share/applications/EQAscendant-eq2.desktop ~/Desktop/EQAscendant-eq2.desktop
```

- This places a clickable EverQuest launcher directly on your desktop. 

- Final Notes:

  - You can edit the eqclient.ini file with nano (default terminal based text editor for most distro)  **(eq1 x 2)**

  ```bash
  nano ~/Games/EQAscendant/eq2/drive_c/Program\ Files/eq2/eqclient.ini
  ```

  - or edit with kate (desktop editor similar to notepad++) install with: sudo apt install kate  **(eq2 x 2)**

  ```bash
  kate ~/Games/EQAscendant/eq2/drive_c/Program\ Files/eq2/eqclient.ini
  ```

  - or use your preferred text editor

#### **⚠️ Note: EverQuest (eq2) is yours to play now. Recommended reading is:**

-  [Defining the Repeatable Wine Prefix Process](#definerepeat) if you want to install additional EverQuest clients.
-  [EverQuest Window Resolutions](#windowres) if you want to see some typical EverQuest Video Modes settings.

**END eq2 PREFIX INSTALLATION PROCESS**

[ToC](#toc)

------

<span id="prefixeq3"></span>

### Create PREFIX eq3

1. ⚠️ Quick Note: The annotation **(eq3 x N)** is a count of the eq3 occurrences in the code block. More on that later.

1. Create the prefix directory: **(eq3 x 1)**

```bash
mkdir -p ~/Games/EQAscendant/eq3
```

2. Initialize the Wine prefix: **(eq3 x 1)**

```bash
WINEPREFIX=~/Games/EQAscendant/eq3 winecfg
```

- Accept prompts to install Wine Mono and/or Wine Gecko if offered. Wine may print warnings, noise or what even looks like errors while it's running — this is normal. The Wine configuration window should open **(success signal for this step)**.

3. In the Wine configuration window make these changes:

- Graphics tab: 
  - uncheck everything. 
  - Click **Apply**, then **OK**. 
- The wine configuration window will close.

4. Install DXVK into the eq3 prefix: **(eq3 x 1)**

```bash
WINEPREFIX=~/Games/EQAscendant/eq3 winetricks dxvk
```

- Non‑fatal warnings or fixme messages are expected as long as the command completes and returns terminal to the command prompt. **(success signal for this operation)**

5. Create the EverQuest install directory inside the prefix:  **(eq3 x 2)**

```bash
mkdir -p ~/Games/EQAscendant/eq3/drive_c/Program\ Files/eq3
```

6. Copy the EverQuest and patcher files into the eq3 prefix: **(eq3 x 2)**

```bash
cp -a ~/Games/EQAscendant/EQ-game-files/. ~/Games/EQAscendant/eq3/drive_c/Program\ Files/eq3/
```

- **Optional:** Verify the file copy & paste succeeded: Confirm the copy completed without errors. Browse to ~/Games/EQAscendant/eq3/drive_c/Program\ Files/eq3/ using your file manager or terminal. Confirm the destination contains many files and subdirectories.

7. In this Step, we explicitly finalize first‑launch display containment before starting the game. This ensures a predictable, non‑disruptive first run. 

- Enable Wine Virtual Desktop: Open Wine configuration for this prefix: **(eq3 x 1)**

```bash
WINEPREFIX=~/Games/EQAscendant/eq3 winecfg
```

- When the Wine configuration window opens **(success signal for this operation)**, make the following changes:

- Applications tab:
  - Click Add application

  - Navigate to the eq3 folder (Program Files/eq3), select eqgame.exe and click Open

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
  - Ensure the Automatically capture the mouse in full-screen windows box is unchecked
  - Set the desktop size to 1024×768
  - Click **Apply**, then **OK**. 
- The wine configuration window will close.

- This containment step prevents full screen rendering during DirectX initialization and avoids display mode switching while EverQuest establishes its video state.  

8. Install wine-mono: Wine-Mono is Wine’s open-source replacement for Microsoft’s .NET Framework. The EQAscendant patcher is a .NET application.  **(eq3 x 1)** 

```bash
WINEPREFIX=~/Games/EQAscendant/eq3 wine msiexec /i ~/Downloads/wine-mono-11.0.0-x86.msi
```

- Success is indicated by the wine desktop being visible for a few seconds and terminal returning to the command prompt.

9. Launch the EQAscendant patcher, which will in turn launch EverQuest:

- In terminal, cd to the EverQuest install directory: **(eq3 x 2)**

```bash
cd ~/Games/EQAscendant/eq3/drive_c/Program\ Files/eq3
```

⚠️ **Note**: If you are following this process for another EMU, edit EQAscendant.exe in the following command to your specific patcher.exe file name. (1 place) Your patcher.exe must auto-launch EverQuest, or the remainder of this guide will ❌ **not work for you**.

- Launch the EQAscendant patcher: **(eq3 x 1)**

```bash
WINEPREFIX=~/Games/EQAscendant/eq3 wine EQAscendant.exe
```

- Success is indicated by the EQAscendant patcher launching.

- Configure the patcher for hands‑off operation:

  - ✅ Check Auto Patch 

  - ✅ Check Auto Play 
    - On future launches, the patcher will automatically patch and start EverQuest without user interaction. 

  - click **Patch**


- After the patcher finishes patching, EverQuest will launch inside the Wine 1024×600 virtual desktop. (**the success signal for this step**).

⚠️ Note: Always launch EverQuest via the EQAscendant patcher. 


- Login to the EQAscendant server

  - To speed server login along, when the SOE splash screen pops, **click** it to move to the login screen. If you don't "click", the splash screen remains in place for 30 seconds before progressing to login.

- Create a test character if you don't already have a character
- Enter world, and look around
- OK, that's enough. Back to work. Open the in-game Options
- **⚠️ Important:**

  - Click the Display tab. On the upper left side of Display options, there is a button that will either read "Switch to Windowed" or "Switch to Fullscreen". If it reads "Switch to Windowed", do nothing. If it reads "Switch to Fullscreen" then click it. 
  - The button should now read "Switch to Windowed". As you play the game and fiddle with options, never click the button when it reads "Switch to Windowed". You must always be in Fullscreen mode.
- Log out and close the game completely. ❌ **Do not proceed** until the game and Wine desktop have completely closed.

⚠️ Note that terminal is not presenting a command prompt. 

- Click in terminal and press Ctrl+C to end the running process
- You should now have a command prompt


10. Create a desktop launcher (eq3). 

- Create the applications directory (it's ok if it already exists) .

```bash
mkdir -p ~/.local/share/applications
```

- Create the icons directory (it's ok if it already exists).

```bash
mkdir -p ~/.local/share/icons
```

- Copy the patcher icon to  ~/.local/share/icons.


⚠️ **Note**: If you are following this process for another EMU, edit eqemupatcher.png in the following command to your specific patcher icon file name. (1 place)

- ⚠️ This is our first use of "sudo", so you'll have to enter your password after entering the following command: **(eq3 x 3)**

```bash
sudo cp ~/Games/EQAscendant/eq3/drive_c/Program\ Files/eq3/eqemupatcher.png ~/.local/share/icons/eqascendant-eq3.png
```

⚠️ **Note**: If you are following this process for another EMU, edit EQAscendant.exe in the following command to your specific patcher.exe file name. (1 place)

- Create the launcher .desktop file: **(eq3 x 8)**

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

- Refresh the desktop application database:

```bash
update-desktop-database ~/.local/share/applications
```

- The launcher will now appear in your desktop environment’s application menu (Games category). You can right‑click it to Add to Desktop or Pin to Panel. If your distro doesn't support adding to desktop from the menu, you can place a copy on your desktop by:

- Copy the launcher from /applications to /Desktop: **(eq3 x 2)**

```bash
cp ~/.local/share/applications/EQAscendant-eq3.desktop ~/Desktop/EQAscendant-eq3.desktop
```

- This places a clickable EverQuest launcher directly on your desktop. 

- Final Notes:

  - You can edit the eqclient.ini file with nano (default terminal based text editor for most distro)  **(eq3 x 2)**

  ```bash
  nano ~/Games/EQAscendant/eq3/drive_c/Program\ Files/eq3/eqclient.ini
  ```

  - or edit with kate (desktop editor similar to notepad++) install with: sudo apt install kate  **(eq3 x 2)**

  ```bash
  kate ~/Games/EQAscendant/eq3/drive_c/Program\ Files/eq3/eqclient.ini
  ```

  - or use your preferred text editor

#### **⚠️ Note: EverQuest (eq3) is yours to play now. Recommended reading is:**

-  [Defining the Repeatable Wine Prefix Process](#definerepeat) if you want to install additional EverQuest clients.
-  [EverQuest Window Resolutions](#windowres) if you want to see some typical EverQuest Video Modes settings.

1. **END eq3 PREFIX INSTALLATION PROCESS**

[ToC](#toc)

------

 <span id="prefixeq4"></span>

### Create PREFIX eq4

1. ⚠️ Quick Note: The annotation **(eq4 x N)** is a count of the eq4 occurrences in the code block. More on that later.

1. Create the prefix directory: **(eq4 x 1)**

```bash
mkdir -p ~/Games/EQAscendant/eq4
```

2. Initialize the Wine prefix: **(eq4 x 1)**

```bash
WINEPREFIX=~/Games/EQAscendant/eq4 winecfg
```

- Accept prompts to install Wine Mono and/or Wine Gecko if offered. Wine may print warnings, noise or what even looks like errors while it's running — this is normal. The Wine configuration window should open **(success signal for this step)**.

3. In the Wine configuration window make these changes:

- Graphics tab: 
  - uncheck everything. 
  - Click **Apply**, then **OK**. 
- The wine configuration window will close.

4. Install DXVK into the eq4 prefix: **(eq4 x 1)**

```bash
WINEPREFIX=~/Games/EQAscendant/eq4 winetricks dxvk
```

- Non‑fatal warnings or fixme messages are expected as long as the command completes and returns terminal to the command prompt. **(success signal for this operation)**

5. Create the EverQuest install directory inside the prefix:  **(eq4 x 2)**

```bash
mkdir -p ~/Games/EQAscendant/eq4/drive_c/Program\ Files/eq4
```

6. Copy the EverQuest and patcher files into the eq4 prefix: **(eq4 x 2)**

```bash
cp -a ~/Games/EQAscendant/EQ-game-files/. ~/Games/EQAscendant/eq4/drive_c/Program\ Files/eq4/
```

- **Optional:** Verify the file copy & paste succeeded: Confirm the copy completed without errors. Browse to ~/Games/EQAscendant/eq4/drive_c/Program\ Files/eq4/ using your file manager or terminal. Confirm the destination contains many files and subdirectories.

7. In this Step, we explicitly finalize first‑launch display containment before starting the game. This ensures a predictable, non‑disruptive first run. 

- Enable Wine Virtual Desktop: Open Wine configuration for this prefix: **(eq4 x 1)**

```bash
WINEPREFIX=~/Games/EQAscendant/eq4 winecfg
```

- When the Wine configuration window opens **(success signal for this operation)**, make the following changes:

- Applications tab:
  - Click Add application

  - Navigate to the eq4 folder (Program Files/eq4), select eqgame.exe and click Open

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
  - Ensure the Automatically capture the mouse in full-screen windows box is unchecked
  - Set the desktop size to 1024×768
  - Click **Apply**, then **OK**. 
- The wine configuration window will close.

- This containment step prevents full screen rendering during DirectX initialization and avoids display mode switching while EverQuest establishes its video state.  

8. Install wine-mono: Wine-Mono is Wine’s open-source replacement for Microsoft’s .NET Framework. The EQAscendant patcher is a .NET application.  **(eq4 x 1)** 

```bash
WINEPREFIX=~/Games/EQAscendant/eq4 wine msiexec /i ~/Downloads/wine-mono-11.0.0-x86.msi
```

- Success is indicated by the wine desktop being visible for a few seconds and terminal returning to the command prompt.

9. Launch the EQAscendant patcher, which will in turn launch EverQuest:

- In terminal, cd to the EverQuest install directory: **(eq4 x 2)**

```bash
cd ~/Games/EQAscendant/eq4/drive_c/Program\ Files/eq4
```

⚠️ **Note**: If you are following this process for another EMU, edit EQAscendant.exe in the following command to your specific patcher.exe file name. (1 place) Your patcher.exe must auto-launch EverQuest, or the remainder of this guide will ❌ **not work for you**.

- Launch the EQAscendant patcher: **(eq4 x 1)**

```bash
WINEPREFIX=~/Games/EQAscendant/eq4 wine EQAscendant.exe
```

- Success is indicated by the EQAscendant patcher launching.

- Configure the patcher for hands‑off operation:

  - ✅ Check Auto Patch 

  - ✅ Check Auto Play 
    - On future launches, the patcher will automatically patch and start EverQuest without user interaction. 

  - click **Patch**


- After the patcher finishes patching, EverQuest will launch inside the Wine 1024×600 virtual desktop. (**the success signal for this step**).

⚠️ Note: Always launch EverQuest via the EQAscendant patcher. 


- Login to the EQAscendant server

  - To speed server login along, when the SOE splash screen pops, **click** it to move to the login screen. If you don't "click", the splash screen remains in place for 30 seconds before progressing to login.

- Create a test character if you don't already have a character
- Enter world, and look around
- OK, that's enough. Back to work. Open the in-game Options
- **⚠️ Important:**

  - Click the Display tab. On the upper left side of Display options, there is a button that will either read "Switch to Windowed" or "Switch to Fullscreen". If it reads "Switch to Windowed", do nothing. If it reads "Switch to Fullscreen" then click it. 
  - The button should now read "Switch to Windowed". As you play the game and fiddle with options, never click the button when it reads "Switch to Windowed". You must always be in Fullscreen mode.
- Log out and close the game completely. ❌ **Do not proceed** until the game and Wine desktop have completely closed.

⚠️ Note that terminal is not presenting a command prompt. 

- Click in terminal and press Ctrl+C to end the running process
- You should now have a command prompt


10. Create a desktop launcher (eq4). 

- Create the applications directory (it's ok if it already exists) .

```bash
mkdir -p ~/.local/share/applications
```

- Create the icons directory (it's ok if it already exists).

```bash
mkdir -p ~/.local/share/icons
```

- Copy the patcher icon to  ~/.local/share/icons.

⚠️ **Note**: If you are following this process for another EMU, edit eqemupatcher.png in the following command to your specific patcher icon file name. (1 place)

- ⚠️ This is our first use of "sudo", so you'll have to enter your password after entering the following command: **(eq4 x 3)**

```bash
sudo cp ~/Games/EQAscendant/eq4/drive_c/Program\ Files/eq4/eqemupatcher.png ~/.local/share/icons/eqascendant-eq4.png
```

⚠️ **Note**: If you are following this process for another EMU, edit EQAscendant.exe in the following command to your specific patcher.exe file name. (1 place)

- Create the launcher .desktop file: **(eq4 x 8)**

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

- Refresh the desktop application database:

```bash
update-desktop-database ~/.local/share/applications
```

- The launcher will now appear in your desktop environment’s application menu (Games category). You can right‑click it to Add to Desktop or Pin to Panel. If your distro doesn't support adding to desktop from the menu, you can place a copy on your desktop by:

- Copy the launcher from /applications to /Desktop: **(eq4 x 2)**

```bash
cp ~/.local/share/applications/EQAscendant-eq4.desktop ~/Desktop/EQAscendant-eq4.desktop
```

- This places a clickable EverQuest launcher directly on your desktop. 

- Final Notes:

  - You can edit the eqclient.ini file with nano (default terminal based text editor for most distro)  **(eq4 x 2)**

  ```bash
  nano ~/Games/EQAscendant/eq4/drive_c/Program\ Files/eq4/eqclient.ini
  ```

  - or edit with kate (desktop editor similar to notepad++) install with: sudo apt install kate  **(eq4 x 2)**

  ```bash
  kate ~/Games/EQAscendant/eq4/drive_c/Program\ Files/eq4/eqclient.ini
  ```

  - or use your preferred text editor

#### **⚠️ Note: EverQuest (eq4) is yours to play now. Recommended reading is:**

-  [Defining the Repeatable Wine Prefix Process](#definerepeat) if you want to install additional EverQuest clients.
-  [EverQuest Window Resolutions](#windowres) if you want to see some typical EverQuest Video Modes settings.

1. **END eq4 PREFIX INSTALLATION PROCESS**

[ToC](#toc)

------

<a id="appendix"></a>

## Appendix: Concepts and Terminology

This appendix provides mostly short, practical explanations of key concepts and technologies referenced throughout this guide. The goal is not academic completeness, but enough context that you understand *what each piece is*, *why it exists*, and *why this guide uses it the way it does*.

Back to [ToC](#toc)  

------

### Wine

Wine is a compatibility layer that allows Windows applications to run on Linux. It is *not* an emulator. Instead of simulating a full Windows OS, Wine translates Windows system calls into native Linux calls in real time.

Why this matters:

- Wine is lightweight compared to virtual machines.
- Applications run close to native performance.
- Each application behaves according to how well Wine implements the Windows APIs it depends on.

In this guide, Wine is the foundation that allows EverQuest and the EQAscendant patcher to run on Linux at all.

Back to [ToC](#toc) or [Guide Goals](#guidegoals) 

------

<a id="wineprefix"></a>

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

Back to [ToC](#toc) or [Guide Goals](#guidegoals) 

------

<a id="wow64"></a>

### 32‑bit vs 64‑bit (WoW64)

EverQuest (Rain of Fear era) is a 32‑bit Windows application. Modern Wine installations are typically 64‑bit Wine with WoW64 support, which means:

- Wine itself runs as 64‑bit
- 32‑bit Windows applications are fully supported inside the same prefix

You do *not* need a separate 32‑bit Wine installation. The guide assumes a standard modern Wine setup that supports both.

Back to [ToC](#toc) or [Guide Goals](#guidegoals) 

------

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

Back to [ToC](#toc) or [Guide Goals](#guidegoals) 

------

### Vulkan

Vulkan is a modern, low-overhead graphics API supported by current GPUs and drivers (including NVIDIA, AMD, and Intel).

DXVK relies on Vulkan as its backend. If Vulkan is working correctly on your system:

- DXVK can operate efficiently
- The GPU driver, not Wine, does most of the heavy lifting

Your distribution’s graphics driver packages handle Vulkan support. This guide does not require any manual Vulkan configuration.

Back to [ToC](#toc) or [Guide Goals](#guidegoals) 

------

### Wine-Mono

Wine Mono is Wine’s open-source replacement for Microsoft’s .NET Framework.

The EQAscendant patcher is a .NET application, which means:

- Wine Mono (or a real .NET runtime) must be present in the prefix
- Without it, Wine will emit errors like: `Wine Mono is not installed`

Why this guide installs Wine Mono manually:

- Wine does not always prompt to install Mono automatically
- The installer often runs silently
- Mono may not appear in `wine uninstaller` even when installed

By installing Wine Mono *before* first launch, the guide ensures deterministic behavior with no reliance on pop-ups or prompts.

Back to [ToC](#toc) or [Guide Goals](#guidegoals) 

------

### Wine-Gecko

Wine Gecko provides an Internet Explorer–like HTML rendering engine inside Wine. It is mainly used by applications that embed web views.

EverQuest itself does not depend on Gecko. Some patchers and launchers may.

If Wine prompts to install Gecko during prefix creation, it is safe to allow it. Gecko does not interfere with EQ or DXVK.

Back to [ToC](#toc) or [Guide Goals](#guidegoals) 

------

### winetricks

winetricks is a helper tool that installs common Windows runtime components into a Wine prefix.

In this guide, winetricks is used *only* for:

- Installing DXVK

Why the guide keeps winetricks usage minimal:

- Reduces hidden side-effects
- Makes the prefix easier to reason about
- Avoids version mismatches between components

The fewer moving parts inside a prefix, the easier it is to debug and reproduce.

Back to [ToC](#toc) or [Guide Goals](#guidegoals) 

------

<a id="virtdt"></a>

### Wine Virtual Desktop

EverQuest is a very old Windows application. When run under WINE *without* a virtual desktop, it can attempt to take direct control of your display mode. On modern systems this has been observed to cause:

- full‑screen blackouts
- loss of keyboard and mouse input
- display modes that do not recover
- situations where the only way out is a **hard power‑off of the PC**

These failures can happen **instantly**, without warning, and without a safe recovery path.

- Once this happens, keyboard shortcuts may not work. Alt‑Tab may not work.
- You may be forced to reboot using the power button.

#### ✅ The Virtual Desktop Is Your Safety Net

For this reason, **this guide assumes EverQuest is always run inside a WINE virtual desktop**. Running outside the virtual desktop is **not supported by this guide**. The WINE virtual desktop provides a containment layer that protects your system:

- EverQuest cannot change your real display resolution
- Alt‑Tab always works
- A crashed client cannot take down your desktop
- Fullscreen mode becomes safe and predictable

Inside the virtual desktop, fullscreen **does not mean real fullscreen**.
It means:

> “Fullscreen within a window that WINE controls.”

That distinction is critical for stability. Wine’s virtual desktop option runs Windows applications inside a fixed-size window instead of allowing them to take over the real display.

Why it is recommended for first launch:

- Prevents fullscreen mode switching during initial DirectX setup
- Avoids display reconfiguration glitches
- Makes first-time configuration safer and more predictable
- Once you are up and running on your EverQuest client you can use in-game options to change video modes to other screen dimensions. The virtual desktop will resize automatically. 

You can disable the virtual desktop later if you prefer native window management. This author won't take responsibility for any resulting window behavior issues.

Back to [ToC](#toc) or [Guide Goals](#guidegoals) 

## ✅ Recommended Display Mode: Fullscreen (Inside the Virtual Desktop)

Once EverQuest is running **inside the WINE virtual desktop**, you should use **Fullscreen mode from within EverQuest**. ⚠️ **ALWAYS**.

This is intentional and recommended.

### Why fullscreen is preferred here

Running fullscreen *inside the virtual desktop*:

- hides the title bar (nothing to accidentally drag)
- prevents window movement
- avoids broken window geometry
- preserves the chosen size across restarts
- makes boxed setups easier to reproduce

Most importantly:

> **Fullscreen inside the virtual desktop is safe.**

It does **not**:

- change your monitor resolution
- lock your system
- trap your keyboard
- risk black‑screen failures

Back to [ToC](#toc) or [Guide Goals](#guidegoals) 

## 🚫 Avoid Windowed Mode in EverQuest

EverQuest’s **Windowed Mode** exposes a draggable title bar and relies on legacy window behavior. Under WINE this can lead to:

- accidental window movement
- corrupted saved window positions
- clients launching off‑screen
- unstable multi‑client layouts

For this reason, the guide strongly recommends:

> **Do not enable Windowed Mode in EverQuest.**

If you need to adjust the size of the game view, use:

> **Options → Display → Video Modes**
> (while remaining in Fullscreen mode)

Changes made there are preserved cleanly across launches.

Back to [ToC](#toc) or [Guide Goals](#guidegoals) 

### 🔁 If You Accidentally Switch to Windowed Mode

This will happen. People click things.

If it does, **do not drag the window around** trying to fix it.

Instead:

1. Open **Options → Display → Video Modes**
2. Switch back to **Fullscreen**
3. Re‑select your desired resolution
4. Exit EverQuest normally and relaunch

This returns the client to a stable state.

------

## ✅ Quick Rules to Remember

> **Always launch inside the WINE virtual desktop.**
> **Always use Fullscreen mode inside EverQuest.**
> **Resize using the Video Modes dialog, not by dragging windows.**

Breaking any of these rules can lead to instability that is difficult—or impossible—to recover from cleanly.

------

## TL;DR

>  **Do NOT run EverQuest outside the WINE virtual desktop.**
>  **Fullscreen inside the virtual desktop is safe and recommended.**
>  **Windowed Mode is discouraged due to instability and drag issues.**

Back to [ToC](#toc) or [Guide Goals](#guidegoals) 

------

<a id="boxing"></a>

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

Back to [ToC](#toc) or [Guide Goals](#guidegoals) 

------

<a id="opion"></a>

### Why This Guide Is Opinionated

Many Linux/Wine guides present multiple paths and leave decisions to the reader. This guide intentionally does not.

The layout, prefix model, and install order are chosen to:

- Minimize undefined behavior
- Maximize repeatability
- Favor clarity over flexibility

Once you understand the process, you can deviate safely. Until then, following a single, consistent model produces the best results.

If you ever wonder *why* a step exists, it should now be answerable somewhere in this appendix.

Back to [ToC](#toc) or [Guide Goals](#guidegoals) 

