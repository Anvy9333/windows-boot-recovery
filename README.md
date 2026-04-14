# 🛠️ Incident Report: Windows 11 Boot Failure (INACCESSIBLE_BOOT_DEVICE)

**Type:** Real situation troubleshooting / Boot recovery  
**OS:** Windows 11  
**Hardware:** MSI motherboard and Crucial CT1000P3SSD8 NVMe 1TB    
**Outcome:**  Full recovery, no data loss

---

## Summary

A PC running Windows 11 became completely unbootable after being physically moved. The system displayed a Blue Screen of Death (BSOD) with stop code `INACCESSIBLE_BOOT_DEVICE (0x7B)` on every boot attempt. Through systematic diagnosis and manual boot record reconstruction, the system was fully recovered without reinstalling Windows or losing any data.

---

## Diagnostic Process

### Step 1 — BIOS Investigation

1) The NVMe SSD was physically detected by the BIOS, ruling out a dead drive or completely loose connection.

2) The SATA controller was set to AHCI mode: this only affects SATA ports and has no impact on NVMe M.2 drives, which always operate in PCIe mode regardless of this setting.

### Step 2 — Physical Inspection

Since the PC had been recently moved, reseated all internal components:
- Removed and reinserted the NVMe SSD in the M.2 slot
- Reseated both RAM sticks

**Result:** No change, the system still unbootable. Confirmed the issue was software/boot record related, not hardware.

### Step 3 — Root Cause Hypothesis

With the drive healthy and detectable, the most likely cause was a corrupted or missing Boot Configuration Data or damaged EFI System Partition boot files, possibly caused by vibration during transport corrupting an in-progress write, or a previous Windows Update that left the boot environment inconsistent.

---

## Recovery Procedure

### Tools Used
- Second PC running Windows
- Rufus 4.13 (bootable USB creation)
- Windows 11 25H2 ISO (official Microsoft source)
- 32GB USB drive

### Bootable USB Creation
- Partition scheme: **GPT**
- Target system: **UEFI (non CSM)**

### Boot from USB
- Accessed boot menu via **F11** at POST on the MSI board
- Selected the USB drive
- On the Windows Setup screen, selected **"Repair your computer"** instead of installing

### Command Prompt Recovery

Navigated to: ` Troubleshoot  > Command Prompt`

**Step 1: Fix the Master Boot Record:**
```
bootrec /fixmbr
```
> Result: operation successfull 


**Step 2 — Fix the EFI partition access issue:**

Assigne a drive letter to the EFI System Partition 

```
diskpart
list vol
```

**Volume list output:**

| Volume | Letter | Name | FS | Type | Size | Status |
|--------|--------|------|----|------|------|--------|
| 0 | C | | NTFS | Partition | 930 GB | Healthy |
| 1 | | | FAT32 | Partition | 100 MB | Hidden |
| 2 | | | NTFS | Partition | 892 MB | Hidden |
| 3 | D | CCCOMA_X64F | NTFS | Removable | 28 GB | Healthy |
| 4 | E | UEFI_NTFS | FAT | Removable | 1039 KB | Healthy |

**Volume 1** (FAT32, 100MB, Hidden) = the EFI System Partition.

```
select vol 1
assign letter=Z
exit
```

**Step 5 — Rebuild boot files targeting the EFI partition:**
```
bcdboot C:\Windows /s Z: /f UEFI
```
> Result: worked

### Final Step
- Closed Command Prompt
- Removed USB drive
- Rebooted

**Windows 11 booted successfully. All data intact. **

---

##  Commands Reference

```bash
# Rebuild Master Boot Record
bootrec /fixmbr

# Fix boot sector
bootrec /fixboot

# Scan and rebuild BCD
bootrec /rebuildbcd

# Open disk partition tool
diskpart

# List all volumes
list vol

# Select and assign letter to EFI partition
select vol <number>
assign letter=Z

# Rebuild boot files for UEFI
bcdboot C:\Windows /s Z: /f UEFI
```

---

*Documented as part of a cybersecurity portfolio in a real-world incident, fully resolved.*
