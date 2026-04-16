# linux-wine-everquest
How to install everquest in linux wine

Installing EverQuest in Linux Wine
EQAscendant Guide
Author note & disclaimer: I (Hrain on Discord), do not serve in any official capacity with EQAscendant, nor do I speak for the project or its staff. I am simply an EverQuest player who enjoys this server, and is sharing what I’ve learned to help other players. All configuration guidance here reflects personal experience, not official project policy.
Technical note (test environment)
This guide was explicitly tested on the following system:

OS: Linux Mint 22.3 (x86_64)
Kernel: 6.17.0-20-generic
Desktop Environment: Cinnamon 6.6.7 (Muffin)
CPU: Intel 13th‑Gen Core i5‑13600K
GPU: NVIDIA GeForce RTX 4070 Ti
Display: 3840×2160
Memory: 128 GB RAM
Applicable software (versions as installed during testing):

Wine (system‑provided): 9.0~repack-4build3, installed via the Linux Mint Software Manager
Wine (user‑installed): 11.0 series, installed alongside system Wine for comparative testing
DXVK: v2.7.1
winetricks: 20240105-2, installed via the Linux Mint Software Manager
NVIDIA Graphics Driver: nvidia-driver-580-open, installed via the Linux Mint Driver Manager
Wine Mono: wine-mono-11.0.0-x86.msi, user‑installed
All required software dependencies are covered by the above software stack installations.
Results on other distributions, kernels, desktop environments, driver versions, Wine builds, or hardware may vary slightly, but the overall procedure should remain applicable.
Document goals
This guide helps you get up and running quickly with one or more EverQuest clients on Linux using Wine, while giving you a practical understanding of the pieces involved so you can scale (boxing) and troubleshoot with confidence.

Launch and play EverQuest via the EQAscendant patcher (recommended path)
Understand Wine prefixes (one prefix per EQ client)
Apply the minimal Wine configuration needed for correct input and display
Use DXVK where required for stable DirectX 9 rendering
Create repeatable multi‑instance installs (eq1, eq2, eq3…) without cross‑contamination
Recover cleanly from common Wine desktop / fullscreen edge cases
Installation overview
This guide follows a repeatable, per‑client workflow. Each EverQuest client is isolated in its own Wine prefix and follows the same setup steps.
Procedure overview:

Define prerequisites (licensed game files and the EQAscendant patcher).
Install the supporting software stack (Wine, graphics driver, DXVK tooling, and related dependencies).
Create a Wine prefix for the client (for example eq1). Each EverQuest client uses its own prefix.
Run winecfg once for the prefix and apply the documented graphics and input settings.
Install DXVK into the prefix so EverQuest can initialize DirectX 9 reliably.
Install EverQuest into the prefix and launch the EQAscendant patcher.
Enter the game once and configure display mode and resolution.
Create a desktop launcher for the patcher bound to that prefix.
Define a repeatable process for adding additional clients (eq2, eq3, …).
This structure avoids shared state between clients, simplifies troubleshooting, and scales cleanly for multi‑boxing.
Prerequisites
Game and patch prerequisites (hard stop)
Before installing any software or creating Wine prefixes, you must already have the following game‑specific assets. If you cannot locate these, you should stop here — the remaining steps depend on them.

A legally obtained EverQuest client (Rain of Fear era) that you are licensed to use

Existing Rain of Fear client directories from a prior Windows or Linux installation may be reused.
A web search on "download everquest rof"Download responsibly.
Do your due diligence to ensure any downloaded files are safe

The EQAscendant patcher and game executables

The EQAscendant project distributes the patcher as a ZIP archive.
Obtain it from the project’s official Discord distribution location:https://discord.com/channels/1467192820610764871/1467195100961706074

Any locally stored copies you intend to reuse

Previously downloaded or installed EQAscendant patcher files may be reused.
Previously downloaded or installed EverQuest client files may also be reused if they meet the required era.
These assets are copied or installed into each Wine prefix during later steps.
Install the supporting software stack
Before configuring a Wine prefix or installing EverQuest, ensure the following host‑level software is installed on your system. These components form the baseline environment used by this guide.
Important guardrail (package sources): Whenever possible, use your distro’s managed packages first (e.g., Linux Mint Software Manager / Driver Manager). Only fall back to command‑line installs (APT) when the software is not available or is materially outdated in the managed repositories. This minimizes dependency conflicts and keeps upgrades clean and supportable.
Source order used in this guide: 1) Distro Software/Driver Manager (preferred) 2) Distro APT repositories (fallback) 3) Upstream installers (last resort; used only when necessary and called out explicitly)
Installation procedures (Linux Mint 22.x)
A) Video graphics driver
Use your distro’s Driver Manager to select and maintain your video driver.

Review the drivers offered for your hardware.
Either:

Select the recommended driver and apply it, or
Keep your current driver if it is already working well and you are satisfied with it.

Apply changes and reboot only if the Driver Manager requests it.
No standalone driver installation commands or version pinning are required for this guide.
B) Wine
Before installing or upgrading Wine, perform a version check.
Create the desktop launcher by running the following commands (copy and paste each line into your terminal):
cat > ~/.local/share/applications/EQAscendant-eq1.desktop <<'EOF'
[Desktop Entry]
Version=1.0
Type=Application
Name=EverQuest (EQAscendant) - eq1
Comment=Launch EverQuest via EQAscendant using the eq1 Wine prefix
Exec=sh -c 'WINEPREFIX="$HOME/Games/EQAscendant/eq1" exec wine "$HOME/Games/EQAscendant/eq1/drive_c/Program Files/eq1/EQAscendant.exe"'
Icon=eqascendant-eq1
Terminal=false
Categories=Game;
EOF
chmod +x ~/.local/share/applications/EQAscendant-eq1.desktop

Step 3: Refresh desktop application database
update-desktop-database ~/.local/share/applications

The launcher will now appear in your desktop environment’s application menu (Games category). You can also right‑click it to Add to Desktop or Pin to Panel. Some desktop environments do not expose an “Add to Desktop” option in menus. In that case, you ### Create a desktop shortcut (optional)
You can manually create a desktop shortcut using the following commands:
cp ~/.local/share/applications/EQAscendant-eq1.desktop ~/Desktop/
chmod +x ~/Desktop/EQAscendant-eq1.desktop

This places a clickable EverQuest launcher directly on your desktop.
Gamma behavior under Wine (important note)
EverQuest adjusts display gamma as part of its DirectX 9 rendering pipeline. Under Wine, these adjustments are applied at the system display level. This mirrors EverQuest’s historical behavior on Windows.
Use EverQuest’s in‑game gamma slider to adjust brightness to a comfortable level. Many long‑time installations settle on a gamma level that works well both in‑game and on the desktop.
The Wine virtual desktop does not isolate gamma changes; it only contains resolution and window behavior. No desktop‑environment configuration or external gamma tools are required or recommended.
