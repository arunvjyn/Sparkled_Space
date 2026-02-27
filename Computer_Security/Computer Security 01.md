Cyber Security Author: Arun
Author contact: arunvjyn அட் gmail.காம்
Location: Cuddalore (கடலூர்), Tamilnadu


# Master Instructions:

Part 1 – Core Principles & Mindset  
Part 2 – Immediate Post-Fresh-Install Hardening (before any 3rd-party software)  
Part 3 – Account & Access Baseline  
Part 4 – Network & Firewall Tightening  
Part 5 – Execution Control & Allowlisting  
Part 6 – Sandboxing & Isolation Choices  
Part 7 – Malware Detection & Persistence Monitoring  
Part 8 – Credential & 2FA Safety  
Part 9 – Data Protection, Backups & Recovery  
Part 10 – Physical & Operational Practices  
Part 11 – Risky Software Ritual (daily/weekly/monthly checklist)  
Part 12 – Quick Implementation Priority Order


## Part 1 – Core Principles & Mindset

- You are advanced / power users with strong security awareness and caution protocols already in place.  
- Gray-area, unaffordable legitimate licenses for needed educational tools, limited/reliable internet.  
- Long-term goal = migrate to open-source + legal/low-cost alternatives when possible, but impossible to enforce on shared machine immediately.  
- Security posture = layered defense assuming endpoint compromise is always possible when running untrusted binaries.  
- Key mantras repeated across conversation:  
  - Least privilege (standard/non-admin daily account)  
  - Isolation first (Sandbox / VM / portable USB)  
  - Revert / delete / snapshot after risky sessions  
  - Offline / low-bandwidth first (USB-based everything)  
  - Monitor persistence vectors aggressively (registry, filesystem, startup, scheduled tasks, services)  
  - Never run unknown installers in main / daily account  
  - Encrypt at rest + in transit where feasible  
  - Test recovery paths regularly (restore, snapshot revert, 2FA codes)

## Part 2 – Immediate Post-Fresh-Install Hardening (before installing any 3rd-party software)

This is the exact sequence to apply right after a clean Windows Pro reinstall + all Windows Updates + zero third-party files present.

1. **Create accounts immediately**  
   - Keep the built-in Administrator account (rename if desired).  
   - Create one dedicated admin account for installs/changes only (strong passphrase).  
   - Create daily standard (non-admin) accounts for each user/neighbor.  
   - Set auto-lock screen after 1–5 minutes idle on all accounts.

2. **Enable full-disk encryption**  
   - Use BitLocker (Windows Pro built-in) on the system drive.  
   - Backup recovery key to encrypted USB or printed/stored securely off-site.  
   - Alternative: VeraCrypt full-drive encryption if BitLocker issues arise.

3. **Tighten outbound network by default (before any app exists)**  
   - Open Windows Defender Firewall → Advanced Settings.  
   - Create a new Outbound Rule for All Programs: Action = Block (apply to Domain/Private/Public profiles).  
   - Create explicit Allow rules only for essential system services:  
     - Windows Update / svchost.exe (limited to Microsoft update servers if possible).  
     - DNS resolver (system).  
     - Time sync (w32time).  
   - Enable firewall logging (dropped packets) to review later.

4. **Set up execution control baseline (AppLocker or SRP)**  
   - Prefer AppLocker (Windows Pro):  
     - Go to Local Security Policy → Application Control Policies → AppLocker.  
     - Create default rules: Allow signed Microsoft files + Windows Installer in trusted paths.  
     - Create Deny rules for executables in user-writable folders (%APPDATA%, %TEMP%, %USERPROFILE%\Downloads, removable drives).  
     - Start in Audit-only mode → review Event Viewer logs → enforce after testing.  
   - Fallback: Software Restriction Policies (via secpol.msc) with similar path/hash/zone rules.

5. **Enable Controlled Folder Access**  
   - Windows Security → Virus & threat protection → Manage ransomware protection.  
   - Turn on Controlled folder access.  
   - Add protected folders: Documents, Desktop, Pictures, Downloads, etc.  
   - Allow only trusted Microsoft apps initially (add more later after verification).

6. **Restrict registry & service writes (non-admin hardening)**  
   - Use Local Security Policy or regedit permissions:  
     - Deny non-admin write to persistence keys:  
       HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run  
       HKCU\Software\Microsoft\Windows\CurrentVersion\Run  
       HKLM\SYSTEM\CurrentControlSet\Services  
     - Be cautious — test in admin session first; keep rollback plan.

7. **Disable autorun/autoplay completely**  
   - Registry: Set NoDriveTypeAutoRun = 255 (all drives) or use Group Policy.  
   - Disable AutoPlay in Settings → Bluetooth & devices → AutoPlay.

8. **Baseline integrity snapshot**  
   - Use PowerShell (as admin):  
     ```powershell
     Get-ChildItem -Path 'C:\Windows','C:\Program Files','C:\Program Files (x86)' -Recurse -File | 
     Get-FileHash -Algorithm SHA256 | 
     Export-Csv -Path 'D:\baseline-hashes.csv' -NoTypeInformation
     ```  
   - Store baseline CSV on encrypted external USB.

9. **Prepare offline tools USB**  
   - Create encrypted VeraCrypt container on USB for future installers, portables, configs.  
   - Download (on another trusted machine when possible): Sysinternals suite (Autoruns, Process Explorer, Process Monitor, Sigcheck), Defender Offline ISO.

10. **Final pre-install posture check**  
    - Disconnect network entirely until first verified offline install.  
    - Verify everything above with a test (e.g., try running .exe from Downloads → should be blocked by AppLocker/SRP).

Say **"next"** when ready for Part 3 – Account & Access Baseline (and continuing).


## Part 3 – Account & Access Baseline

(Applies after fresh install hardening in Part 2 — this is the always-on foundation for shared laptop use.)

- **Daily usage rule** — Everyone (including you) must use a **standard / non-admin account** for all normal work.  
  - Prevents most OS-level changes from malware (cannot install drivers/services, modify protected system files, or easily alter other users' data).  
  - Limits damage scope even if privilege escalation occurs (many exploits still need admin rights to persist system-wide).  
  - Combine with other layers — non-admin alone is **partial** protection, not complete.

- **Admin account discipline**  
  - Use admin account **only** for:  
    - Installing verified software  
    - Changing system settings  
    - Running maintenance tools (Autoruns, Sysmon install, etc.)  
  - Log out of admin immediately after use; never browse, email, or run risky files as admin.

- **Screen lock & idle timeout**  
  - Set automatic screen lock after 1–5 minutes idle (via Settings → Accounts → Sign-in options or Group Policy).  
  - Enforce on all accounts (prevents unattended access on shared machine).

- **Password / passphrase strength**  
  - Use long, unique passphrases (20+ characters, memorable but complex).  
  - No reuse across accounts.  
  - Never store in plain text, browser autofill, or shared locations on the machine.

- **Shared machine reality note**  
  - You cannot force neighbors to follow perfect hygiene — therefore design assumes worst-case (someone else may run risky things).  
  - Mitigation: dedicated non-admin account for risky tasks (see Part 6), delete files after sessions, revert snapshots/VMs.

Say **"next"** when ready for Part 4 – Network & Firewall Tightening.

## Part 4 – Network & Firewall Tightening

(Applies after baseline accounts and disk encryption. Designed for low-bandwidth / intermittent connectivity and shared machine where apps may be untrusted.)

- **Default posture: Block outbound by default**  
  - Windows Defender Firewall → Advanced Settings → Outbound Rules.  
  - Main rule: Block all programs (Domain/Private/Public profiles).  
  - Explicitly allow only:  
    - System essentials (Windows Update, DNS, NTP/time sync).  
    - Later: specific verified apps (by full executable path, not just port).  
  - Enable logging for dropped/allowed packets (review periodically via %systemroot%\system32\LogFiles\Firewall\pfirewall.log).

- **Per-app / post-install tightening**  
  - After installing any app (offline, verified):  
    - Create custom outbound rule for that .exe path.  
    - Restrict to known required destinations (IPs/ports/domains) if documented; otherwise block network initially.  
    - Use kill-switch equivalent: block all unless explicitly allowed.  
  - For risky / gray-area apps: run offline or with outbound blocked; monitor first runs.

- **Avoid risky networks**  
  - Never use public/open Wi-Fi for sensitive work.  
  - Prefer trusted mobile hotspot (your phone) or fully offline workflows.  
  - Disable network file/printer sharing (Settings → Network & Internet → Advanced network settings).

- **Portable VPN usage (when anonymity needed)**  
  - Use portable / no-install clients: OpenVPN Portable or WireGuard standalone binary.  
  - Store binaries + .ovpn/.conf files (with keys) in encrypted VeraCrypt container on USB.  
  - Run from non-admin account or inside Sandbox/VM to minimize registry changes.  
  - Caveat: TAP/WireGuard drivers usually require admin install once (do offline).  
  - Must-haves: enable DNS leak protection + kill-switch (drop traffic if tunnel fails).  
  - Important reminder: local malware can still capture data before encryption (keystrokes, screenshots, files) — VPN protects transit only.

- **Low-bandwidth / offline network practices**  
  - Download updates/signatures/installers on another trusted machine → transfer via encrypted USB.  
  - Avoid online account recovery / password resets on shared laptop.  
  - Monitor active connections: use TCPView (Sysinternals) or Resource Monitor during risky sessions.

Say **"next"** when ready for Part 5 – Execution Control & Allowlisting.


## Part 5 – Execution Control & Allowlisting

(This builds directly on Part 2's baseline AppLocker/SRP setup. Goal: prevent unauthorized / untrusted executables from running, especially from user-writable locations common in gray-area software scenarios.)

- **Primary mechanism: AppLocker (preferred on Windows Pro)**  
  - Executable rules: Allow signed Microsoft binaries + explicit paths (e.g., C:\Program Files\*, C:\Windows\*)  
  - Deny rules: Block execution from  
    - %APPDATA%\*  
    - %LOCALAPPDATA%\*  
    - %TEMP%\*  
    - %USERPROFILE%\Downloads\*  
    - Removable drives / USB mounts  
  - Script rules: Similar denies for .ps1, .vbs, .js, etc. in writable folders  
  - DLL rules: Block unsigned / untrusted DLLs in user folders  
  - Start in **Audit mode** (logs violations without blocking) → review Event Viewer (Applications and Services Logs → Microsoft → Windows → AppLocker) → switch to **Enforce** after confirming no breakage of legitimate use  
  - Test thoroughly: attempt to run portable .exe from Downloads or USB → should be blocked

- **Fallback: Software Restriction Policies (if AppLocker unavailable or too complex)**  
  - secpol.msc → Software Restriction Policies → Enforcement → All software files  
  - Additional Rules:  
    - Disallowed: %AppData%\*, %Temp%\*, %UserProfile%\Downloads\*, removable media  
    - Unrestricted: C:\Program Files\*, C:\Windows\* (signed)  
    - Hash rules for specific trusted installers (compute SHA256 once, add rule)

- **Controlled Folder Access (ransomware-style write protection)**  
  - Already enabled in Part 2 → add more protected folders as needed (e.g., entire user profile if cautious)  
  - Allow only verified apps to write (add them post-install after testing in Sandbox)

- **Practical controlled install workflow** (for every gray-area or third-party app)  
  1. Disconnect network completely  
  2. Verify installer hash/checksum on trusted separate device (if possible)  
  3. Scan installer with Windows Defender (offline scan preferred)  
  4. Install as admin (only in C:\Program Files if possible, avoid user folders)  
  5. Immediately add the main .exe path to AppLocker/SRP allow list  
  6. Add the .exe to Controlled Folder Access allowed apps (if it needs to write protected folders)  
  7. Create strict outbound firewall rule for that .exe (allow only known endpoints or block initially)  
  8. First run: in Sandbox or VM (see Part 6) or dedicated risky non-admin account  
  9. Monitor with Process Explorer / Sysmon / TCPView during run

- **Key benefit in shared / gray-area context**  
  - Even if neighbor runs cracked installer from Downloads or USB without thinking, AppLocker/SRP deny prevents execution from writable locations → forces portable/isolated use or blocks outright.

## Part 6 – Sandboxing & Isolation Choices

(For running gray-area / risky software with maximum containment on low-spec hardware. Prioritize lowest overhead first.)

- **Tier 1: Windows Sandbox (lightest, ephemeral – preferred for one-off / high-risk runs)**  
  - Available on Windows 10/11 Pro (virtualization enabled in BIOS + Hyper-V features).  
  - Each session = fresh, clean Windows instance; everything discarded on close (strong security feature).  
  - Low resource use compared to full VM.  
  - **Persistence workarounds** (since no built-in save):  
    - Map host folder → Sandbox config (.wsb file): place installers, portable apps, scripts in mapped folder (e.g., C:\SandboxTools on host).  
    - Use startup provisioning script (PowerShell/batch in mapped folder): auto-run on Sandbox start to copy/install minimal tools from mapped folder.  
    - Store all needed portables + data in encrypted VeraCrypt container on USB → mount USB inside Sandbox or copy from mapped folder.  
  - Workflow: Run risky .exe → do work → close Sandbox → all changes gone. Rebuild quickly via script/USB.

- **Tier 2: Lightweight Virtual Machine (for repeatable / persistent needs)**  
  - Use VirtualBox (free, low overhead).  
  - Allocate minimal resources: 512–1024 MB RAM, 1 CPU core, small dynamic VDI disk (10–20 GB).  
  - Install minimal OS (e.g., lightweight Windows or Linux rescue ISO) + only required runtime.  
  - **Key practice**: Take clean snapshot after setup/updates → revert to snapshot after every risky session.  
  - Run VM offline by default; enable network only when monitored.  
  - Headless mode if no GUI needed (saves resources).  
  - Alternative: Use system rescue ISOs (e.g., Windows PE) booted in VM for ultra-minimal runtime.

- **Tier 3: Dedicated non-admin account for risky tasks**  
  - Create one standard account named e.g. "RiskyUse" or "TempSession".  
  - Run gray-area software only here.  
  - Keep files isolated to that profile.  
  - Delete profile contents (or entire profile) after session.  
  - No saved credentials, no browser history/sync.  
  - Combine with AppLocker denies in writable folders + outbound block.

- **Portable apps as default (lowest persistence risk)**  
  - Prefer PortableApps.com suite, AppImage (Linux), or standalone .exe portables.  
  - Store/run from encrypted VeraCrypt container on USB.  
  - Reduces infection surface (no registry writes, no system install).  
  - Mount USB only when needed; unmount after.

- **Recommended hybrid for your low-spec shared laptop**  
  - Primary: Encrypted USB with portables + Windows Sandbox (map folder or script rebuild).  
  - Secondary: Small VirtualBox VM with snapshots (revert always) for tasks needing persistence.  
  - Never install risky software system-wide if avoidable.

Say **"next"** when ready for Part 7 – Malware Detection & Persistence Monitoring.

## Part 7 – Malware Detection & Persistence Monitoring

(For ongoing vigilance on shared low-spec machine. Focus on low-overhead, periodic checks + real-time logging where feasible. Tools are Sysinternals-based — portable, no heavy install needed.)

- **Core monitoring tools (download once, store on encrypted USB)**  
  - Autoruns (Sysinternals): Weekly run to inspect all startup locations, Run keys, scheduled tasks, services, drivers, logon items.  
    - Look for: unknown entries, unsigned binaries, unusual paths.  
    - Disable/remove suspicious items immediately (as admin).  
  - Process Explorer: Real-time process viewer.  
    - Check: CPU/network spikes, unknown parent/child processes, suspicious command lines.  
    - Kill suspicious processes; investigate further if persistent.  
  - Process Monitor (ProcMon): Capture filesystem/registry/network activity (short bursts only — high volume).  
    - Filter by: process name, path (e.g., registry Run keys, %APPDATA%), operation (CreateFile, RegSetValue).  
    - Use for debugging suspicious behavior after first runs.  
  - TCPView: Quick network connection monitor (alternative to Resource Monitor).  
    - Watch for unexpected outbound connections from unknown processes.

- **Advanced logging (install once, low impact)**  
  - Sysmon (Sysinternals): Install with conservative config (XML file).  
    - Recommended events to log (minimal noise):  
      - Process Create  
      - File Create / CreateStream (in critical folders)  
      - RegistryEvent (SetValue on persistence keys like Run, Services)  
      - NetworkConnect  
      - DriverLoad  
    - Review logs in Event Viewer (Applications and Services Logs → Microsoft → Windows → Sysmon).  
    - Export periodically to encrypted USB for offline analysis.  
  - Windows Event Log auditing: Enable Object Access / Detailed Tracking for success/failure on critical registry keys and folders.

- **Integrity checking (filesystem / registry baseline)**  
  - Weekly PowerShell script (run as admin, save output to encrypted USB):  
    ```powershell
    $paths = 'C:\Windows', 'C:\Program Files', 'C:\Program Files (x86)', "$env:USERPROFILE\Desktop", "$env:USERPROFILE\Documents"
    Get-ChildItem -Path $paths -Recurse -File -ErrorAction SilentlyContinue | 
    Get-FileHash -Algorithm SHA256 | 
    Export-Csv -Path 'E:\integrity-check-$(Get-Date -Format yyyyMMdd).csv' -NoTypeInformation
    ```  
  - Compare new CSV to previous (manual diff or simple script) → flag new/ changed files in system folders or user writable areas.

- **Offline deep scans**  
  - Maintain rescue USB with Microsoft Defender Offline (download ISO, create bootable USB on another machine).  
  - Boot from it monthly or on suspicion → scans outside Windows, catches rootkits/persistent malware.

- **What to watch for & immediate actions**  
  - New/unknown entries in Autoruns (Run keys, tasks, services).  
  - Unexpected drivers or kernel modules.  
  - Repeated registry writes to persistence areas by unknown processes.  
  - New executables in %APPDATA%, %TEMP%, Startup folders.  
  - Suspicious network from non-system processes.  
  - If detected:  
    1. Disconnect network immediately.  
    2. Boot rescue USB → full scan.  
    3. Revert VM snapshot or restore from clean backup.  
    4. Change critical passwords from trusted separate device.

Say **"next"** when ready for Part 8 – Credential & 2FA Safety.

## Part 8 – Credential & 2FA Safety

(This is critical for shared machine where keyloggers / credential stealers are a realistic threat from gray-area software.)

- **Password manager choice**  
  - Use KeePass or KeePassXC (fully offline, portable, no cloud sync).  
  - Store vault file (.kdbx) in encrypted VeraCrypt container on external USB (never on shared laptop HDD).  
  - Never open vault on main daily account → open only from your non-admin account or directly from encrypted USB (in Sandbox if paranoid).

- **Strong protection for vault**  
  - Long master passphrase (20+ characters, memorable but high-entropy).  
  - Require **keyfile** in addition to passphrase:  
    - Generate random keyfile in KeePassXC settings.  
    - Store keyfile on **separate** encrypted USB or secure location (different physical place).  
    - Stealing vault file alone is useless without both passphrase + keyfile.  
  - Enable auto-lock on inactivity + clear clipboard after copy.  
  - Optional extra: Trigger words / pattern (e.g., add secret string only you know to passphrase; keep pattern hint printed/stored separately in safe place).

- **Splitting / avoiding single-point failure**  
  - Vault file → encrypted USB #1 (kept with laptop or hidden).  
  - Keyfile → encrypted USB #2 or printed QR code in locked safe place.  
  - One offline encrypted backup of vault → off-site (trusted neighbor/family, sealed envelope or another encrypted drive).  
  - Printed emergency hint: obscure reminder/phrase only you understand (e.g., "first pet + favorite number backwards") stored separately from devices.

- **2FA implementation**  
  - Enable wherever possible (email, important accounts, password manager itself if supported).  
  - Prefer:  
    - TOTP authenticator apps (Aegis, andOTP, Bitwarden Auth – exportable/backupable).  
    - Hardware tokens (YubiKey, Nitrokey – OATH-TOTP mode; keep primary + backup token separate).  
  - Avoid SMS as primary (SIM swap risk).  
  - Immediately after enabling:  
    - Generate and save recovery codes / backup methods.  
    - Store recovery codes: encrypted USB + one printed copy in secure physical location.  
    - Test recovery flow once (log out → recover with codes).  
  - Never do account recovery / reset on shared laptop (use trusted phone or separate device).

- **Shared machine hygiene rules**  
  - Never autofill passwords in browser on shared device.  
  - Close KeePass immediately after use; lock screen when away.  
  - If vault must be accessed during risky session: do it in Sandbox or VM only.

Say **"next"** when ready for Part 9 – Data Protection, Backups & Recovery.

## Part 9 – Data Protection, Backups & Recovery

(Focus on encryption at rest, offline backups, and fast recovery — essential when running untrusted software on shared hardware.)

- **Sensitive file storage**  
  - Never store important files unencrypted on the shared laptop drive.  
  - Use VeraCrypt:  
    - Create encrypted containers (hidden volume optional for extra plausible deniability).  
    - Mount only when needed; unmount immediately after.  
    - Store containers on external USB (encrypted filesystem or VeraCrypt).  
  - For very sensitive data: keep on separate encrypted USB, never copy to laptop unless in Sandbox/VM.

- **Regular offline backups**  
  - Before any risky session: copy critical files to encrypted external drive / USB.  
  - Use VeraCrypt container or BitLocker-To-Go on the backup drive.  
  - Maintain **at least two copies**:  
    - One local (with laptop, but encrypted).  
    - One off-site (trusted neighbor/family, sealed/locked location).  
  - Versioned backups if space allows (e.g., date-stamped folders inside container).  
  - Wipe free space securely after deleting sensitive files (use Cipher.exe /w or VeraCrypt wipe).

- **Rescue & recovery USB (critical for compromise scenarios)**  
  - Create bootable USB with:  
    - Microsoft Defender Offline (download ISO → Rufus or similar to make bootable).  
    - Windows Recovery Environment (WinRE) or lightweight PE ISO if needed.  
    - Sysinternals tools (Autoruns, Process Explorer, etc.) copied to USB.  
  - Keep USB encrypted or stored securely.  
  - Test boot & scan periodically.

- **System / VM recovery practices**  
  - For VMs: always take clean snapshot after setup → revert after every risky session.  
  - For full system: maintain periodic encrypted disk image (use Macrium Reflect Free or similar portable, store on external).  
  - Practice full restore: boot rescue USB → scan → restore from known-good backup/image → verify.

- **Incident response checklist (print & pin near laptop)**  
  1. Suspect infection / anomaly → immediately disconnect network (unplug cable / disable Wi-Fi).  
  2. Do not shut down normally (preserve memory if possible).  
  3. Boot from clean rescue USB.  
  4. Run full Defender Offline scan.  
  5. Inspect Autoruns / Sysmon logs from backup drive if copied.  
  6. Restore from known-good snapshot (VM) or encrypted backup (files/system).  
  7. If full wipe needed: reinstall fresh OS (keep offline initially).  
  8. Change all critical passwords from a trusted separate device (phone / another computer).  
  9. Re-apply hardening from Parts 2–5 before reconnecting.

## Part 10 – Physical & Operational Practices

(These are the non-technical layers that complement software controls — especially important for a shared laptop in a remote/village setting with multiple users and limited oversight.)

- **Physical device security**  
  - Use a Kensington cable lock (or adhesive anchor plate if no K-slot) to deter opportunistic theft.  
    - Loop cable around immovable object (table leg, metal rack, wall fixture).  
    - Choose keyed or combo lock; keep spare key/code in secure off-site location.  
  - Lock the room / cupboard where laptop is stored when not in use (if feasible).  
  - Remove battery (if removable) or power cable for long absences.

- **Camera & microphone controls**  
  - Physically cover webcam with non-residue sticker/tape (easy removal, no permanent mark).  
  - Disable/unplug microphone when not needed (tape over hole if built-in; use external only).  
  - In Windows: Settings → Privacy → Camera/Microphone → turn off access for all apps by default; allow only when actively using.

- **Access limitation & shared rules**  
  - Limit physical access to trusted people only when possible.  
  - Agree on minimal group rules (even if hard to enforce):  
    - No running unknown installers in main/daily account.  
    - Always scan USBs before opening.  
    - Use non-admin account or Sandbox/VM for anything risky.  
    - Backup important files before sessions.  
  - Print and pin a short one-line daily reminder near laptop:  
    "Scan USB → Use non-admin / Sandbox / VM → Backup first → Revert/delete after → Lock screen"

- **USB & removable media handling**  
  - Treat every USB as potentially malicious (common infection vector in shared setups).  
  - Always: Right-click → Scan with Defender before opening files.  
  - Mount read-only when possible (via third-party tool or copy to quarantine folder first).  
  - Keep "quarantine" folder on host for incoming files (scan → move if clean).

- **Operational routines (daily/weekly/monthly)**  
  - **Daily / before risky work**:  
    - Lock screen when away.  
    - Close KeePass/vaults immediately after use.  
    - Never run installers/executables in main account.  
  - **Weekly**:  
    - Full Windows Defender scan.  
    - Run Autoruns → review/disable suspicious entries.  
    - Verify scheduled tasks & services (taskschd.msc, services.msc).  
    - Quick Process Explorer check during typical use.  
    - Compare integrity hashes (PowerShell script from Part 7).  
  - **Monthly / before high-risk sessions**:  
    - Update rescue USB (new Defender Offline ISO).  
    - Update portable tools / AV signatures offline (transfer via USB).  
    - Test full recovery: boot rescue → scan → restore snapshot/backup.  
    - Create fresh system/VM snapshot or disk image backup.  
    - Review Sysmon logs for anomalies.

## Part 11 – Risky Software Ritual (Concise Daily/Weekly/Monthly Checklist)

(This is the practical, repeatable workflow distilled from the entire conversation — optimized for your shared low-spec laptop, gray-area usage, and power-user caution level. Print or note this near the laptop.)

**Daily / Before Any Risky Session Ritual** (3–5 minutes)
1. Switch to dedicated non-admin account (or create fresh session profile if needed).  
2. Disconnect network (unplug cable / disable Wi-Fi) if possible.  
3. Backup critical files to encrypted external USB (quick copy if changed).  
4. Scan incoming USB/installer with Defender (right-click → Scan).  
5. Run software only via:  
   - Windows Sandbox (map folder + provisioning script if needed) — preferred for one-off.  
   - Lightweight VM (revert to clean snapshot after).  
   - Portable .exe from encrypted USB (no install).  
6. Monitor first run: Process Explorer open → watch CPU/network/parent processes.  
7. After work:  
   - Close / discard Sandbox.  
   - Revert VM snapshot.  
   - Delete session files in risky account.  
   - Unmount USB / close vault.  
   - Lock screen.

**Weekly Routine** (15–30 minutes)
1. Full Windows Defender scan (or quick if time limited).  
2. Run Autoruns → review & disable/remove unknown/suspicious entries (Run keys, tasks, services, drivers).  
3. Check scheduled tasks (taskschd.msc) & services (services.msc) for anomalies.  
4. Quick Sysmon / Event Viewer review (filter recent ProcessCreate, RegistryEvent, NetworkConnect).  
5. Run integrity hash check (PowerShell script) → compare to previous CSV on USB.  
6. Process Explorer sweep: look for odd processes/connections during normal use.

**Monthly / Pre-High-Risk Session Routine** (30–60 minutes)
1. Update rescue USB: fresh Microsoft Defender Offline ISO + tools.  
2. Transfer latest portable tools / AV signatures / installers via trusted offline method.  
3. Test full recovery path: boot rescue USB → simulate scan → restore test backup / snapshot.  
4. Create new clean VM snapshot or full encrypted disk image backup.  
5. Review & tighten any loose firewall / AppLocker rules based on recent usage.  
6. Verify KeePass vault backup off-site + test 2FA recovery codes (on trusted device).

**One-Line Quick Reminder (to pin by laptop)**  
"Scan USB → Non-admin / Sandbox / VM only → Backup first → Monitor Process Explorer → Revert / Delete / Close → Lock screen"

## Part 12 – Quick Implementation Priority Order (Final Part)

(This is the realistic, phased rollout sequence right after your fresh Windows Pro install + all updates. Prioritized for maximum risk reduction with minimal effort/time on low-spec shared laptop. Do in this order; each step builds on the previous.)

**Phase 1: Foundation (Day 1 – 1–2 hours) – Do these before connecting to internet or installing anything**
1. Create admin account + daily non-admin accounts for each user.  
2. Enable full-disk encryption (BitLocker preferred). Backup key off-device.  
3. Set auto-lock screen (1–5 min idle) on all accounts.  
4. Block outbound firewall by default + allow only system essentials (Windows Update, DNS, time sync). Enable logging.  
5. Enable Controlled Folder Access + protect key user folders.  
6. Set up AppLocker/SRP in Audit mode: allow Microsoft signed + trusted paths; deny user-writable folders (Downloads, %APPDATA%, USB, etc.).  
7. Disable autorun/autoplay completely.  
8. Create baseline file hash snapshot (PowerShell on C:\Windows, Program Files, etc.) → save to encrypted USB.

**Phase 2: Detection & Recovery Setup (Day 1–2 – 30–60 min)**
9. Prepare encrypted VeraCrypt USB for tools/portables/backups.  
10. Download & create rescue USB (Microsoft Defender Offline + Sysinternals suite) on another machine if needed → test boot.  
11. Install Sysmon with conservative config (ProcessCreate, NetworkConnect, RegistryEvent on persistence keys, FileCreate in critical folders).  
12. Install KeePassXC portable → create vault with strong passphrase + keyfile (keyfile on separate USB). Make off-site encrypted backup.

**Phase 3: Isolation & Monitoring Tools (Day 2–3 – 1 hour)**
13. Enable 2FA on important accounts (TOTP app or hardware) + store recovery codes split/printed securely.  
14. Set up Windows Sandbox (.wsb config with mapped folder for provisioning if needed).  
15. Install lightweight VirtualBox → create minimal VM (512–1024 MB RAM) + take clean snapshot.  
16. Test portable apps from USB in Sandbox/VM.

**Phase 4: Operationalization & Testing (Day 3–7 – ongoing)**
17. Switch to non-admin daily use permanently.  
18. Practice risky ritual: backup → Sandbox/VM → monitor Process Explorer → revert/delete → lock.  
19. Weekly routine: Defender scan, Autoruns check, integrity compare, scheduled tasks/services review.  
20. Monthly: update rescue USB, test full restore/snapshot revert, review Sysmon logs.

**Phase 5: Ongoing Refinements**
- Add per-app firewall rules + AppLocker allows only after Sandbox/VM testing.  
- Tighten registry permissions on persistence keys if no breakage occurs.  
- Expand Controlled Folder Access whitelists gradually.  
- Print/pin the one-line reminder + risky ritual checklist.

This order gives you strong protection quickly while allowing gradual addition of gray-area tools in the safest way possible (offline install → Sandbox first → monitored run → strict rules).
