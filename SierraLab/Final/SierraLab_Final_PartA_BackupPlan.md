SierraLab Final — Part A: Backup Plan
Prepared by: shsarwari  |  Date: 2026-10-10  |  Environment: Windows Server 2025 (SierraLab)  |  Document Version: 1.0  |  Status: Plan prepared for review; execution and verification are separate tasks.

---

PURPOSE
This document defines the backup strategy for the SierraLab Windows Server 2025 environment. It ensures recoverability of critical system state (Active Directory), all Group Policy Objects, and important files, while preserving the existing backup on the secondary drive X:.

---

SCOPE — WHAT IS COVERED
The plan requires three backup categories, all written to the secondary hard drive X: (labeled "Hyper V"):

1. Back up important files (user data, scripts, configs) — Robocopy /E to X:\SierraLab_Backups\Files\<YYYYMMDD_HHMMSS>\.
2. Back up Active Directory / System State — Windows Server Backup (wbadmin) targeting X: volume; actual WSB storage under X:\WindowsImageBackup\<server>\<timestamp>.
3. Back up all Group Policy Objects — Backup-GPO PowerShell cmdlet to X:\SierraLab_Backups\GPOs\<YYYYMMDD_HHMMSS>\.

Out of scope: VM-level Hyper-V checkpoints; SQL/Exchange application-level backups (not present in this environment).

---

DESTINATION STRUCTURE (X: — secondary hard drive)

X:\
├── WindowsImageBackup\          ← EXISTING — DO NOT TOUCH, DELETE, RENAME, MOVE, OR OVERWRITE
└── SierraLab_Backups\           ← NEW — structured backups only
    ├── SystemState\             ← System State (WSB / wbadmin)
    ├── GPOs\                    ← Group Policy Objects (Backup-GPO, timestamped)
    └── Files\                   ← Important files (Robocopy /E, timestamped)

Protection of existing backup: The existing X:\WindowsImageBackup folder must never be deleted, renamed, moved, or overwritten. Before any backup job, confirm it remains intact. If it is missing or modified, stop and request approval for an alternative destination.

---

SPACE REQUIREMENTS (historical observation only)
- Previous observation (~9.28 GB free on X:) is historical; do not treat as current guarantee.
- Estimated per-backup footprint: System State ~2–4 GB; GPOs ~50–200 MB; Files = variable — assess before each run.
- Safety check: Verify sufficient free space before starting; use fresh measurement (Section 9) with realistic estimates for the planned backup types.
- Do not assume 5 GB is always sufficient without checking current availability.

---

BACKUP PROCEDURES

A. PREREQUISITES (run before every backup)
1. Verify X: mounted: Get-PSDrive X
2. Check free space (Section 9) — fresh measurement, not historical.
3. Confirm existing backup: Test-Path X:\WindowsImageBackup
4. Confirm feature installed: Get-WindowsFeature Windows-Server-Backup

B. SAFETY CONDITIONS (must all be true; if any fails, stop and request approval)
- Destination sufficiency: X: has enough free space for the expected backup (see Section 9).
- No overwrite risk: New backup creates a new subfolder under X:\WindowsImageBackup; existing content not replaced.
- Writable destination: New timestamped folders can be created in X:\SierraLab_Backups\GPOs\ and X:\SierraLab_Backups\Files\.
- Existing backup intact: X:\WindowsImageBackup contains the prior backup and has not been modified.

PowerShell check (before any backup):
if (-not (Get-PSDrive X -ErrorAction SilentlyContinue)) {
    Write-Error "X: is not mounted. Request approval for an alternative destination."
    exit 1
}

C. SYSTEM STATE BACKUP — ACTIVE DIRECTORY
Tool: wbadmin (command-line) or Windows Server Backup MMC
Destination: wbadmin targets the X: volume; WSB writes to default target X:\WindowsImageBackup (not the organizational SierraLab_Backups\SystemState folder).

Command:
wbadmin start systemstatebackup -backupTarget:X: -quiet

Stop condition before execution:
- Confirm available capacity on X: is sufficient for System State (estimate ~2–4 GB; verify with $drive = Get-PSDrive X).
- Confirm X:\WindowsImageBackup is intact and that the new backup will create a new subfolder rather than overwrite existing content.
- If either condition fails, stop and request approval for an alternative before running.

Protection note: wbadmin writes new versions under new subfolders in X:\WindowsImageBackup. This does not guarantee preservation of the existing backup if the drive is near full or if manual deletion occurs. Do not delete, rename, or move X:\WindowsImageBackup at any time.

D. GROUP POLICY OBJECTS BACKUP
Tool: Backup-GPO (Group Policy PowerShell module)
Destination: X:\SierraLab_Backups\GPOs\<YYYYMMDD_HHMMSS>\

PowerShell procedure (create directory first):
$timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
$gposDest = "X:\SierraLab_Backups\GPOs\$timestamp"
if (-not (Test-Path $gposDest)) {
    New-Item -ItemType Directory -Path $gposDest -Force | Out-Null
}
Backup-GPO -All -Path $gposDest -Comment "SierraLab scheduled GPO backup $timestamp"

Notes:
- Each run creates a new timestamped folder — no overwrites.
- Verify directory exists before calling Backup-GPO.
- Retention: keep last 7 runs (manual cleanup or scheduled task).

E. IMPORTANT FILES BACKUP
Tool: Robocopy (safe mirroring with /E — not /MIR)
Destination: X:\SierraLab_Backups\Files\<YYYYMMDD_HHMMSS>\
Sources: C:\Scripts, C:\AdminTools, C:\Users\Administrator\Documents, and other critical paths.

PowerShell procedure (create destination/log first, check sources, verify exit codes):
$timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
$filesDest = "X:\SierraLab_Backups\Files\$timestamp"
if (-not (Test-Path $filesDest)) {
    New-Item -ItemType Directory -Path $filesDest -Force | Out-Null
}
$logFile = Join-Path $filesDest "robocopy.log"
$sourcePaths = @("C:\Scripts", "C:\AdminTools", "C:\Users\Administrator\Documents")
$success = 0
foreach ($src in $sourcePaths) {
    if (Test-Path $src) {
        robocopy $src "$filesDest$(Split-Path $src -NoQualifier)" /E /R:2 /W:5 /LOG+:$logFile
        if ($LASTEXITCODE -eq 0) { $success += 1 }
        else { Write-Error "Robocopy failed for source '$src' (exit code: $LASTEXITCODE)" }
    } else { Write-Verbose "Source not found (skipping): $src" }
}
if ($success -eq $sourcePaths.Count) { Write-Host "File backup completed successfully." }
else { Write-Error "Some file backups failed. See $logFile for details."; exit 1 }

Notes:
- /E copies subdirectories recursively; safer than /MIR (which deletes destination files not present in source).
- Log each run to $filesDest\robocopy.log.
- Verify exit codes: 0 = success; non-zero = failure — do not ignore failures.

---

VERIFICATION PROCEDURES (after each component completes)

System State: wbadmin get versions -backupTarget:X: — confirm new version appears.
GPOs: Get-ChildItem X:\SierraLab_Backups\GPOs\<latest>\ — verify folder count matches domain count.
Files: Get-ChildItem X:\SierraLab_Backups\Files\<latest>\ -Recurse | Measure-Object.

Log all results: X:\SierraLab_Backups\backup_verification.log (append with timestamp).

---

RECOVERY CONSIDERATIONS
- System State / AD Restore: Plan for both authoritative and non-authoritative restore; boot into DSRM; test restore in isolated VM first.
- GPO Restore: Import-GPO with valid BackupId from timestamped folder; use Get-ChildItem to enumerate.
- File Restore: Robocopy /E in reverse, or manual copy from timestamped folder.

---

SECURITY PRECAUTIONS
- Access Control: Restrict X:\SierraLab_Backups\ to Administrators (Full Control) and Backup Operators (Read) via ACL; disable inheritance; apply explicit rules. Adjust per organizational policy.
- Encryption at Rest: Enable BitLocker only after confirming recovery-key storage (file, AD, or USB). Use Authenticated AES-256 with recovery-password protector. Document the recovery-key location — do not rely solely on system-managed keys.
- Integrity: Compute SHA256 hashes of critical outputs after each backup.
- Offsite: Replicate to USB/NAS/cloud separately; this plan covers local secondary-drive only.

---

FREE SPACE CHECK (before each backup — fresh measurement required)
Run this script; do not rely on historical 9.28 GB:

$drive = Get-PSDrive X
$freeGB = [math]::Round($drive.Free / 1GB, 2)
$systemStateEstGB = 4; $gposEstGB = 0.2; $filesEstGB = 1
$totalEstimatedGB = $systemStateEstGB + $gposEstGB + $filesEstGB
Write-Host "X: free space: $freeGB GB"
Write-Host "Estimated backup footprint this run: $totalEstimatedGB GB"
if ($freeGB -lt $totalEstimatedGB) {
    Write-Error "Insufficient free space ($freeGB GB < $totalEstimatedGB GB estimated). Clean up or expand X: before proceeding."
    exit 1
} else { Write-Host "Space check PASSED." }

---

CHECKLIST — WHAT THIS PLAN COVERS (assignment requirements)
- [x] All backups saved to secondary hard drive X:
- [x] Back up important files (Section 4.5)
- [x] Back up Active Directory / System State (Section 4.3)
- [x] Back up all Group Policy Objects separately (Section 4.4)
- [x] Destination folders defined (X:\SierraLab_Backups\SystemState, GPOs, Files)
- [x] Existing X:\WindowsImageBackup protected (do not delete/rename/move/overwrite)
- [x] Safety conditions and stop conditions included (Section 4.2)
- [x] Free-space check included with realistic estimates (Section 9)
- [x] Verification steps for all three categories (Section 5)
- [x] Recovery considerations documented (Section 6)
- [x] Security precautions (ACL, BitLocker with recovery-key storage, integrity) (Section 7)
- [x] No backup commands executed; no server settings changed; no VMs modified

---

STATUS STATEMENT
Backup plan prepared for review; actual backup execution and verification are separate tasks.
No backups have been executed. No files have been deleted. No server settings have been changed. No virtual machines have been modified. This document is evidence of backup planning for assignment Part A only.

---

Prepared by: shsarwari  |  File: SierraLab_Backup_Plan.md  |  Location: C:\Users\shsarwari\Desktop\Sierra IT\Sierra IT\  |  Date: 2026-10-10  |  Assignment: SierraLab Final — Part A: Backup Plan
