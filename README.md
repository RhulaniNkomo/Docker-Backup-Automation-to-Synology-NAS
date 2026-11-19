# Docker-Backup-Automation-to-Synology-NAS
Infrastructure / Backup Automation Platform: Windows, Docker, Synology DSM Automation: PowerShell and Windows Task Scheduler Status: Core backup workflow implemented and test# Docker Backup Automation to Synology NAS

**Project type:** Infrastructure / Backup Automation  
**Platform:** Windows, Docker, Synology DSM  
**Automation:** PowerShell and Windows Task Scheduler  
**Status:** Core backup workflow implemented and tested

## 1. Project Overview

This project implements an automated Docker backup workflow using a Synology NAS as a centralised backup destination.

The objective was to create a repeatable process for backing up Docker database data and volume contents, transferring the backup files to network storage, and automatically removing older backups according to a defined retention period.

The solution uses a PowerShell script generated with ChatGPT based on my requirements. I defined the backup requirements, configured the environment, implemented the scheduled workflow and tested the resulting backups.

## 2. Infrastructure Environment

### NAS configuration

| Component | Configuration |
|---|---|
| NAS model | Synology DS925+ |
| Storage drives | 2 × Samsung 2TB SATA SSDs |
| RAID configuration | RAID 1 |
| Filesystem | Btrfs |
| Approximate usable capacity | 2TB |
| Backup destination | Dedicated shared folder |

RAID 1 mirrors data across both SSDs, providing protection against a single drive failure when the storage pool is healthy.

**Important:** RAID redundancy is not a replacement for an independent backup.

### Backup source

A Windows PC running Docker workloads is used as the backup source.

## 3. Backup Architecture

The backup process follows this workflow:

```text
        WINDOWS PC
             |
             v
    PowerShell Backup Script
             |
       +-----+------+
       |            |
       v            v
  SQL Database   Docker Volumes
     Dumps        (Read-only)
       |            |
       v            v
    .sql files    .tar files
       |            |
       +-----+------+
             |
             v
      Local Backup Folder
        D: or C:
             |
             v
     Windows Task Scheduler
             |
             v
       Network Transfer
         (UNC Path)
             |
             v
       SYNOLOGY DS925+
             |
             v
      Docker Backup Share
             |
             v
      14-Day Retention
```

## 4. Backup Implementation

### 4.1 Database backups

The PowerShell automation generates SQL database dump files.

The database exports are stored as `.sql` files.

### 4.2 Docker volume backups

Docker volume contents are backed up using a temporary container with the volumes mounted in read-only mode.

The resulting volume backup files use the `.tar` format.

The read-only mounting approach is intended to avoid modifying the source volume contents during the backup operation.

### 4.3 Local backup staging

The script uses a local backup directory.

- Drive D: is preferred when available.
- Drive C: is used as a fallback.
- Backups are organised into date-based folders.

Example directory structure:

```text
Docker-Backups/
│
├── 2026-11-17/
│   ├── Database-Dumps/
│   │   └── database.sql
│   │
│   └── Docker-Volumes/
│       └── volume.tar
│
├── 2026-11-18/
│   ├── Database-Dumps/
│   └── Docker-Volumes/
│
└── 2026-11-19/
    ├── Database-Dumps/
    └── Docker-Volumes/
```

*Illustrative directory structure.*

## 5. NAS Backup Transfer

After creating the local backup files, the script copies the day's backup contents to a shared folder on the Synology NAS.

The destination is configured using a Windows UNC network path.

Windows Credential Manager stores the credentials used to access the NAS.

The current implementation uses the main NAS administrator account. A dedicated backup account with restricted folder permissions is identified as a future security improvement.

## 6. Automation and Scheduling

Windows Task Scheduler is used to execute the PowerShell backup script automatically every night.

The script also contains functionality to register a scheduled task for 4:00 AM.

The purpose of scheduling is to run backups without requiring manual execution each night.

## 7. Backup Retention

The automation implements a 14-day retention policy.

Two retention processes are included:

**NAS retention**
- Identifies date-named backup folders.
- Removes folders older than the configured retention period.
- Only targets folders matching the `yyyy-MM-dd` naming convention.

**Local retention**
- Removes old date-named backup folders.
- Removes older loose backup files according to the retention period.

This helps control storage consumption on both the Windows backup source and the NAS.

## 8. Testing and Verification

The core backup workflow has been run and checked.

| Test | Result |
|---|---|
| SQL database dump generation | Tested |
| Docker volume backup generation | Tested |
| Date-based backup organisation | Tested |
| Transfer to NAS shared folder | Tested |
| 14-day retention process | Tested |
| Scheduled execution | Configured |
| Manual NAS backup verification | Performed |

### Current verification method

After the scheduled backup, I access the NAS shared folder and check that the expected dated backup folder and files are present.

Automated email notifications and a dedicated logging mechanism have not yet been configured.

## 9. Known Limitations

The following areas require further investigation or improvement:

1. **Restore validation:** A restoration attempt was made, but errors were encountered. Successful recovery has not yet been demonstrated.
2. **Backup logging:** No dedicated logging mechanism has been identified.
3. **Email notifications:** Backup success and failure notifications are not configured.
4. **NAS availability:** Behaviour when the NAS is unavailable during a scheduled backup has not yet been tested.
5. **Access control:** The workflow currently uses the main NAS administrator account instead of a dedicated backup account.

These limitations are recorded to distinguish backup creation from proven recovery capability.

## 10. Future Improvements

- Complete a successful database restoration test.
- Complete a Docker volume restoration test.
- Implement structured backup logs.
- Configure email notifications for backup failures.
- Test behaviour when the NAS is offline.
- Introduce a dedicated NAS backup user with restricted permissions.
- Review backup integrity verification.
- Consider an additional off-site backup destination.

## 11. AI-Assisted Development

The PowerShell backup script was generated with ChatGPT using requirements I provided.

My contribution included:

- Identifying the backup requirements.
- Defining the required backup contents and retention period.
- Configuring the Synology NAS storage destination.
- Setting up Windows Task Scheduler.
- Implementing the backup workflow.
- Running and checking the backup process.
- Identifying limitations and areas requiring further testing.

The use of AI-assisted code generation is disclosed to accurately represent the development process.

## 12. Project Outcome

The project established a working automated process for creating Docker database and volume backups, staging the files locally, transferring them to a Synology NAS and applying a 14-day retention policy.

The next major milestone is successful restoration testing to demonstrate that the backups can be used to recover the Docker workloads.

---

**Project category:** IT Infrastructure | Systems Administration | Backup and Recovery | PowerShell Automation | Docker | Synology NAS
