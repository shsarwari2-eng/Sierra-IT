# SierraLab Final — Part B: Decommissioning Plan

**Author:** shsarwari  
**Date:** 2026-10-10  
**Version:** 1.0  
**Status:** PLAN ONLY — no decommissioning actions executed  
**Purpose:** Student assignment plan describing how the SierraLab Windows Server 2025 environment would be safely decommissioned. Not a record of completed destruction.

---

## Statement of scope and limitations
This document is a **plan**, not evidence of completed decommissioning. No domain-controller demotion, VM removal, application uninstallation, file deletion, disk sanitization, account removal, or setting change has been performed. The existing Part A backup evidence is preserved (file backup partially successful; System State backup successful with version-database verification limitation; GPO backup successful). Existing X:\WindowsImageBackup is untouched.

---

## 1. Authorization and preparation
- Confirm ownership: student lab (shsarwari), assignment SierraLab Final — Part B.
- Confirm instructor requirements and approval to decommission; do not proceed without written/instructor confirmation.
- Confirm assignment scope: Windows Server 2025 domain controller, Windows 10 VM, Ubuntu Server VM, Hyper-V resources, lab applications/configurations/policies/files.
- Dependencies: verify no other lab users depend on this domain controller or VMs; confirm replication/standalone status before AD removal.
- Maintenance window: define and approve a window; stop if dependencies are discovered.

## 2. Asset inventory (planned — to confirm before action)
- VMs: Windows Server 2025 (domain controller), Windows 10, Ubuntu Server.
- Virtual disks / checkpoints / virtual switches (Hyper-V): inventory with Get-VM, Get-VHD, Checkpoint-VM, Get-VMSwitch.
- Lab applications, services, scheduled tasks, scripts.
- Lab-only accounts (not production/domain-wide), credentials (no secrets recorded here), temporary certificates/policies.
- Files: C:\Scripts, C:\AdminTools, C:\Users\Administrator\Documents (partially backed), plus lab outputs.
- Distinguish lab from unrelated systems (node1/2/3, Manager 1, other Hyper-V VMs, shared storage, X:\WindowsImageBackup existing backup) — these must NOT be touched.

## 3. Backup and retention gate (must complete before destruction)
- Confirm required Part A backups retained (Files partially successful — missing C:\Scripts, C:\AdminTools sources noted; System State ~65 MB verified physically with version-DB limitation documented; GPO successful).
- Confirm retention approvals and evidence preserved.
- Confirm X:\WindowsImageBackup intact; do not delete/rename/move.
- Stop condition: if required backups or approvals are missing, halt and request instructor approval — do not proceed with destructive steps.

## 4. Shutdown and dependencies
- Sequence: shut down dependent VMs first, then domain controller last (after confirming no replication/dependency needs).
- Confirm no other systems or users depend on this DC or VMs; document confirmation.

## 5. Active Directory decommissioning (requires instructor approval)
- Prerequisites: confirm standalone-lab / no replication partners (or handle replication first), prepare DSRM if needed, back up System State.
- Process: demote domain controller using dcpromo / AD DS removal with proper cleanup; handle DNS/dependency updates; remove domain-controller roles and metadata if applicable; verify removal.
- Do not execute without approval; document planned steps only.

## 6. VM and storage removal (planned — after approvals)
- Inventory approved lab VMs only; do not affect unrelated VMs.
- Shut down, unregister/remove approved VMs; handle virtual disks, checkpoints, installation media, associated files per inventory.
- Preserve existing X:\WindowsImageBackup.

## 7. Applications and configuration cleanup (planned — in scope only)
- Lab-installed applications, services, scheduled tasks, scripts.
- Lab-only accounts/credentials (no secrets in this document); temporary policies/files.
- Do not modify node1/2/3/Manager 1 or unrelated systems.

## 8. Secure data sanitization and remanence
- Select sanitization method appropriate to storage technology (SSD vs HDD vs virtual disk); do not claim ordinary deletion/formatting guarantees erasure.
- Verify result; document limitations.
- Handle encryption keys/credentials separately; do not erase shared/host storage without explicit authorization.
- Do not claim secure erasure from VM deletion or file deletion alone.

## 9. Validation and acceptance criteria (planned)
- Verify intended lab assets decommissioned; confirm no in-scope resources remain.
- Confirm access revoked; document retained evidence (Part A reports, this plan, inventory, approvals, any logs/exceptions).

## 10. Evidence and documentation
- Part A reports: Evidence_FileBackup_Only.md (partially successful), Evidence_AD_SystemState_Backup.md (success, version-DB limitation noted), Evidence_GPO_Backup.md (success), Evidence_SierraLab_PartA_All_Backups.md.
- Required future evidence (not produced — requires execution with approval): command outputs (Get-VM, wbadmin, Backup-GPO, robocopy logs), inventory records, approval records, sanitization verification, sign-off.
- No credentials/tokens included.

## 11. Risks, rollback, and exceptions
- Recovery depends on Part A backups (note partial file backup, System State verification limitation); irreversible steps after AD demotion/disk sanitization.
- Stop conditions: missing backups/approvals, undiscovered dependencies, failure of verification.
- Exceptions: document any incomplete or failed item; do not fabricate success.

## 12. Final checklist (by phase — planned, not completed)
- [ ] Authorization and approval obtained (instructor).
- [ ] Asset inventory completed and unrelated systems identified.
- [ ] Backup/retention gate passed (Part A limitations acknowledged; retention approved).
- [ ] Dependencies confirmed; shutdown sequence planned.
- [ ] AD decommission prerequisites verified; approval received before any demotion.
- [ ] VMs inventoried; unrelated VMs excluded; removal approved.
- [ ] App/config cleanup scoped; unrelated systems excluded.
- [ ] Sanitization method selected; verification method defined; shared storage excluded without authorization.
- [ ] Validation criteria defined; evidence list preserved.
- [ ] Risks/rollback documented; stop conditions stated.
- [ ] Sign-off recorded (after actual execution — not now).

---

## References (preserved, not altered)
- Part A backup plan: `C:\Users\shsarwari\Desktop\Sierra IT\Sierra IT\SierraLab_Final_PartA_BackupPlan.md`
- Part A consolidated evidence: `C:\Users\shsarwari\Desktop\Sierra IT\Sierra IT\Evidence_SierraLab_PartA_All_Backups.md`
- GitHub repo: shsarwari2-eng/Sierra-IT, branch master (not uploaded — pending review)
- Existing backup destination protected: `X:\WindowsImageBackup`

---
*No decommissioning actions have been executed. No VMs modified. No files deleted. No settings changed. No credentials included. Upload to GitHub deferred pending user review.*
