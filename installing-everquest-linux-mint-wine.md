# Installing EverQuest in Linux Wine
*EQAscendant Guide* 

**Author note & disclaimer:** I (Hrain on EQAscendant Discord) and (timvgaming on github), do not serve in any official capacity with EQAscendant, nor do I speak for the project or its staff. I am simply an EverQuest player who enjoys this server, and is sharing what I’ve learned to help other players. All configuration guidance here reflects personal experience, not official project policy. 

### Table of Contents (Ctrl Click)

\- [Introduction](#installing-everquest-in-linux-wine)

\- [Technical Notes and Test Environment](####Technical Note (test environment))

\- [Document Goals](#document-goals)

\- [Installation Overview](#installation-overview)

\- [Prerequisites](#prerequisites)

\- [Install the Supporting Software Stack](#Install the host-level software stack)

\- [Installing EverQuest](#OK! Let’s install a game)



#### Technical Note (test environment)

This guide was explicitly tested on the following system:

- **OS:** Linux Mint 22.3 (x86_64) Kernel: 6.17.0-20-generic 
- **Desktop Environment:**Cinnamon 6.6.7 (Muffin) 
- **CPU:** Intel 13th‑Gen Core i5‑13600K 
- **GPU:** NVIDIA GeForce RTX 4070 Ti 
- **Display:** 3840×2160 
- **Memory:** 128 GB RAM 

**Applicable Software (versions as installed during testing):**

- **Wine (system‑provided):** 9.0~repack-4build3; installed via the Linux Mint Software Manager 
- **Wine (user‑installed (apt)):** 11.0 installed for comparative testing 
- **DXVK:** v2.7.1 
- **winetricks:** 20240105-2; installed via the Linux Mint Software Manager 
- **NVIDIA Graphics Driver:** nvidia-driver-580-open; installed via the Linux Mint Driver Manager 
- **Wine Mono:** wine-mono-11.0.0-x86.msi; user‑installed 

All required software dependencies are covered by the above software stack installations. Results on other distributions, kernels, desktop environments, driver versions, Wine builds, or hardware may vary slightly, but the overall procedure should remain applicable. 

## Document goals ##

This guide helps you get up and running with one or more EverQuest clients on Linux using Wine, while giving you a practical understanding of the pieces involved so you can scale (boxing) and troubleshoot with confidence.

- Launch and play EverQuest via the EQAscendant patcher (recommended path). 
- Understand Wine prefixes (one prefix per EQ client). 
- Apply the minimal Wine configuration needed for correct input and display. 
- Use DXVK where required for stable DirectX 9 rendering. 
- Create repeatable multi‑instance installs (eq1, eq2, eq3…) without cross‑contamination. 
- Recover cleanly from common Wine desktop / fullscreen edge cases. 

## Installation overview ## 

This guide follows a **repeatable, per‑client workflow.** Each EverQuest client is isolated in its own Wine prefix and follows the same setup steps. All parts of the guides' installation process are executed via your terminal in a **copy** command from guide **& paste** into terminal, and then **press Enter** flow. Successful results of the command execution are predictable and will be shown as part of the guide process in cases where there may be some ambiguity.

**Procedure overview:**

- Define prerequisites (licensed game files and the EQAscendant patcher). 
- Install the supporting software stack (Wine, graphics driver, DXVK tooling, and related dependencies). 
- Create a Wine prefix for the client (for example eq1). Each EverQuest client uses its own prefix. 
- Run winecfg once for the prefix and apply the documented graphics and input settings. 
- Install DXVK into the prefix so EverQuest can initialize DirectX 9 reliably. 
- Install EverQuest into the prefix and launch the EQAscendant patcher. 
- Enter the game once and configure display mode and resolution. 
- Create a desktop launcher for the patcher bound to that prefix. 
- Define a repeatable process for adding additional clients (eq2, eq3, …). 

This structure avoids shared state between clients, simplifies troubleshooting, and scales cleanly for multi‑boxing. 

## Prerequisites ##

 **Game and patch prerequisites (hard stop)**

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

## Install the host-level software stack

These components form the baseline environment used by this guide. **Important guardrail (package sources):** Whenever possible, use your distro’s managed packages first (e.g., Linux Mint Software Manager / Driver Manager). Only fall back to command‑line installs (APT) when the software is not available or is materially outdated in the managed repositories. This minimizes dependency conflicts and keeps upgrades clean and supportable. Before configuring a Wine prefix or installing EverQuest, ensure the following **host‑level software** is installed on your system:

**Source order used in this guide:**

1. Distro Software/Driver Manager (preferred) 
2. Distro APT repositories (fallback) 
3. Upstream installers (last resort; used only when necessary and called out explicitly) 

#### Host-level software stack Installation procedures: ####

**(Tested on Linux Mint 22.3 Cinnamon) My Distro, but should work on other Ubuntu/Debian based**.

**A)  Video graphics driver:** Use your distro’s Driver Manager to select and maintain your video driver.

- Review the drivers offered for your hardware. 
- Either: Select the recommended driver and apply it, or 
- Keep your current driver if it is already working well and you are satisfied with it. 
- Apply changes and reboot only if the Driver Manager requests it. 

No standalone driver installation commands or version pinning are required for this guide.

**B) Wine: **

Before installing or upgrading Wine, perform a version check: 

```bash
wine --version
```

- If the reported version is Wine 9.x or newer, no action is required — proceed directly to **C) winetricks**. 
- If the command prints wine: command not found, Wine is not installed. 
- If Wine is not installed, or the reported version is older than Wine 9, continue below. 
  - Check managed packages first (Software Manager):
  - Open your distro’s Software Manager. 
    - Search for Wine. 
    - If a package providing Wine 9.x or newer is available, install it. 
    - If not, install via the Fallback shown below. 

**Fallback (apt):** sudo apt update sudo apt install wine


```bash
sudo apt update
sudo apt install wine
```

apt will install the distro‑provided stable package. Verify: wine --version

```bash
wine --version
```

**C) winetricks:** Before installing winetricks, perform a version check: 

```bash
winetricks --version
```

- If the command prints any version string, winetricks is already installed — proceed directly to **D) DXVK**. 
- If the command prints winetricks: command not found, winetricks is not installed — continue below. 
  - Open your distro’s Software Manager. Search for winetricks. Install the available package. 
  - If no package is available, install via the Fallback below. 

**Fallback (apt):** sudo apt update sudo apt install winetricks

```bash
sudo apt update
sudo apt install winetricks
```

apt will install the distro‑provided stable package. Verify: winetricks --version

```bash
winetricks --version
```

**D) DXVK:** DXVK is installed per‑Wine prefix, not system‑wide. It will be installed as part of the EverQuest build procedures. 

**E) Wine Mono:** Wine Mono is installed per‑Wine prefix, not system‑wide. It will be installed as part of the EverQuest build procedures. We only need to download it here and pre‑position it for use later by Wine. 

**What to download:**

- Download the matching Wine Mono installer (example tested): wine-mono-11.0.0-x86.msi 
- Official Wine Mono installers are published by WineHQ here:
  - https://dl.winehq.org/wine/wine-mono/ 
- Do **not** install distro Mono packages (e.g. mono-runtime, mono-complete); **Wine requires the Windows Mono .msi**, not the system Mono runtime. 

**Where to put it:**

- Place the MSI in your Downloads folder ($HOME/Downloads). Wine, when needed, should be able to discover it there.

## OK! Let’s install a game ##

Everything above set the stage. From here on, we build one EverQuest client at a time, each isolated in its own Wine prefix. The steps that follow focus on layout and prefix creation first, installing the game and patcher into that prefix, verifying a good launch and creating a desktop launcher.  Before running any commands, it’s important to understand exactly how this guide organizes files on disk. The layout below is not only a recommendation — it is the structure this guide **will** use when installing EverQuest on your system if you follow the guide verbatim. Using your own preferred layout is discussed at the bottom of this section.

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
│  └── drive_c/
│    └── Program Files/
│      └── eq4/       # EverQuest client files (eq4)
├── eq5/              # Wine prefix (eq5)
│  └── drive_c/
│    └── Program Files/
│      └── eq5/       # EverQuest client files (eq5)
└── eq6/              # Wine prefix (eq6)
  └── drive_c/
    └── Program Files/
      └── eq6/        # EverQuest client files (eq6)
```

How to read this

- ~/Games/EQAscendant/ is the root folder for all EverQuest clients. 
- EQ-game-files/ is a staging area used to hold a clean EverQuest client plus the EQAscendant patcher; its contents are copied into prefixes during installs and are never run directly. 
- Each eqN/ directory is a complete, isolated Wine prefix. This guide shows six eq directories. You can create as many as you like. 
- Each prefix installs EverQuest into its own matching directory under Program Files/eqN. 
- There is no shared Program Files/EverQuest directory. This one‑to‑one mapping (prefix ↔ client folder ↔ launcher) prevents cross‑contamination, makes boxing predictable, and ensures uninstalling a client is as simple as deleting its eqN/ directory. 

#### Using your own layout design 

Please use the layout described above for the purpose of getting through this guide with the provided copy and paste commands. After you have gone through this guide and are comfortable with the process you can create your EQ installs using your preferred layout. Uninstalling an existing Wine prefix and the associated EQ client is a simple matter of:

- Delete the prefix folder (or any higher level folder in your home directory). None of the previously installed host-level software packages are effected. No part of a Wine prefix lives outside of the prefix folder. Deleting it is targeted total annihilation and the ultimate uninstall process. 
- Don’t attempt to reuse any part of an existing prefix folder path in a new Wine prefix. You must follow the process to create each new Wine prefix. 

#### Populate the EQ-game-files folder (one‑time setup) 

Before creating the first Wine prefix, place the EverQuest client files and the EQAscendant patcher files to where the guide expects to find them. 

**Step 1:** Create the directory tree mkdir -p ~/Games/EQAscendant/EQ-game-files

```bash
mkdir -p ~/Games/EQAscendant/EQ-game-files
```
Do not create any eqN directories yet; Wine will create them during prefix initialization. 

**Step 2:** Copy and paste the EverQuest client files and the EQAscendant patcher files

- **Order matters.** This step builds the staging copy of EverQuest that will later be duplicated into one or more Wine prefixes. The EverQuest client files must be copied first and then the patcher files. It is necessary that the patcher files overwrite some of the client files.

- **Source and destination**

  - Source 1: your Rain of Fear (RoF) era EverQuest client directory 
  - Source 2: your EQAscendant patcher files directory

  - Destination: ~/Games/EQAscendant/EQ-game-files/

**A)** Using your distro file manager, copy **all files and subdirectories** from the unzipped RoF client into the destination directory. This includes (but is not limited to):

- Game executables and DLLs
- Resources/Maps/uifiles/
- Any other data directories present in the RoF client
- Do **not copy the RoF client folder**; only copy the folders and files within it.
- Do **not** attempt to launch EverQuest from the destination directory. 

**B)** Using your distro file manager, copy **all files and subdirectories** from the unzipped EQAscendant patcher into the destination directory. This includes (but is not limited to):

-  **Important**: If your file manager prompts you to choose an action for existing files (for example, Replace, Overwrite, or Merge), choose the option that replaces existing files. The patcher is expected to overwrite some files shipped with the base client. 

**What not to do:**

- Do not mix files from different EverQuest eras 
- Do not create any eqN Wine prefixes yet 
- Do not run the patcher or the game from EQ-game-files

At the end of this step, EQ-game-files/ should contain a complete EverQuest client plus the EQAscendant patcher, ready to be copied into Wine prefixes. 

### Pre‑configure initial EverQuest window settings

 EverQuest will soon be launching for the first time inside a Wine prefix. To ensure a predictable, accessible first launch—and to standardize behavior across this prefix and any future prefixes—you will replace eqclient.ini in the staging folder with a minimal first‑launch version to define known window parameters. 

Create the minimal eqclient.ini file: 
```bash
cat > ~/Games/EQAscendant/EQ-game-files/eqclient.ini <<'EOF'
[VideoMode]
Width=1920
Height=1080
WindowedWidth=1920
WindowedHeight=1080
[Defaults]
Gamma=5
EOF
```
*Note:* Additional settings are intentionally omitted. EverQuest will populate defaults and user preferences automatically on first launch. A lower initial gamma (such as Gamma=5, approximately 21% in‑game) avoids display gamma bleed into the desktop environment under Wine while still providing a comfortable baseline; you can fine‑tune gamma and dimensions later using the in‑game and winecfg options. 

This creates a clean, deterministic baseline for the initial launch. No further editing is required at this stage. 

At this point, EQ-game-files/ folder contains a clean, reproducible EverQuest + EQAscendant baseline. 

### Create the first Wine prefix (eq1) 

Each EverQuest client lives in its own Wine prefix. Here's the first one!

**Step 1:** Create the prefix directory: mkdir -p ~/Games/EQAscendant/eq1

```bash
mkdir -p ~/Games/EQAscendant/eq1
```

**Step 2:** Initialize the Wine prefix: WINEPREFIX=~/Games/EQAscendant/eq1 winecfg

```bash
WINEPREFIX=~/Games/EQAscendant/eq1 winecfg
```

Accept prompts to install Wine Mono and/or Wine Gecko if offered. Wine may print warnings, noise or what even looks like errors while it's running — this is normal. When the Wine configuration window opens **(success signal for this operation)**, make only the following changes:

- Graphics tab: 
  - uncheck everything. 
  - Click **Apply**, then **OK**. 
- The wine configuration window will close.

**Step 3:** Install DXVK into the eq1 prefix: WINEPREFIX=~/Games/EQAscendant/eq1 winetricks dxvk 

```bash
WINEPREFIX=~/Games/EQAscendant/eq1 winetricks dxvk
```

Non‑fatal warnings or fixme messages are expected as long as the command completes and returns terminal to the command prompt. **(success signal for this operation)**

**Step 4:** Copy EverQuest files into the eq1 prefix 

1. Create the EverQuest install directory inside the prefix: 

```bash
mkdir -p ~/Games/EQAscendant/eq1/drive_c/Program\ Files/eq1
```

2. Copy the game files

```bash
cp -a ~/Games/EQAscendant/EQ-game-files/. ~/Games/EQAscendant/eq1/drive_c/Program\ Files/eq1/
```

 **Verify the file copy paste succeeded:** Confirm the copy completed without errors. Browse to ~/Games/EQAscendant/eq1/drive_c/Program\ Files/eq1/ using your file manager or terminal. Confirm the destination contains many files and subdirectories.

**Step 5:** Launch the EQAscendant patcher (first run) 

At this point, we explicitly finalize first‑launch display containment before starting the game. This ensures a predictable, non‑disruptive first run. 

1. Enable Wine Virtual Desktop: Open Wine configuration for this prefix: 

```bash
WINEPREFIX=~/Games/EQAscendant/eq1 winecfg
```

When the Wine configuration window opens **(success signal for this operation)**, make only the following changes:

- Graphics tab: 
  - Enable: Emulate a virtual desktop
  - Set the desktop size to 1920 x 1080
  - Ensure all other boxes are unchecked
  - Click **Apply**, then **OK**. 
- The wine configuration window will close.

This containment step prevents exclusive full screen during initial DirectX initialization and avoids display mode switching while EverQuest establishes its video state. You may disable the virtual desktop later if you prefer native window management. 

2. Change to the EverQuest install directory:

```bash
cd ~/Games/EQAscendant/eq1/drive_c/Program\ Files/eq1
```

3. Launch the EQAscendant patcher:

```bash
WINEPREFIX=~/Games/EQAscendant/eq1 wine EQAscendant.exe
```

Success is indicated by the EQAscendant patcher launching.

4. Configure the patcher for hands‑off operation:

- ✅ Check Auto Patch 
- ✅ Check Auto Play 
  - On future launches, the patcher will automatically patch and start EverQuest without user interaction. 
- After the patcher finishes patching, EverQuest will launch (**the success signal for this step**).
  -  Note: Always launch EverQuest via the EQAscendant patcher. 

**Step 6:** Create a desktop launcher (eq1) 

Create a launcher so you can start EverQuest with a single click using the correct Wine prefix and patcher settings. 

1. Create the applications directory (if it doesn’t exist) 

```bash
mkdir -p ~/.local/share/applications
```

2. Create the launcher .desktop file:

```bash
# Copy the patcher icon into the user icon directory (one-time setup)
mkdir -p ~/.local/share/icons
cp ~/Games/EQAscendant/eq1/drive_c/Program\ Files/eq1/eqemupatcher.png 
~/.local/share/icons/eqascendant-eq1.png
# Create the desktop launcher
cat > ~/.local/share/applications/EQAscendant-eq1.desktop <<'EOF'
[Desktop Entry]
Version=1.0
Type=Application
Name=EverQuest (EQAscendant) - eq1
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

3. Refresh desktop application database:

```bash
update-desktop-database ~/.local/share/applications
```

The launcher will now appear in your desktop environment’s application menu (Games category). You can also right‑click it to Add to Desktop or Pin to Panel. If your distro doesn't support adding to desktop from the menu, you can place a copy on your desktop by:

```bash
cp ~/.local/share/applications/EQAscendant-eq1.desktop ~/Desktop/ chmod +x ~/Desktop/EQAscendant-eq1.desktop
```

This places a clickable EverQuest launcher directly on your desktop. 

