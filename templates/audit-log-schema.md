# Audit Log Schema

This document defines the fields that must be captured in the BSA automation audit log for every record processed — both resolved and escalated. This log is the primary evidence of internal controls operating as designed.

---

## Azure SQL Table Definition

```sql
CREATE TABLE BSA_Audit_Log (
    LogID               INT IDENTITY(1,1) PRIMARY KEY,
    RunDate             DATE NOT NULL,
    RunTimestamp        DATETIME NOT NULL,
    AccountNumber       VARCHAR(20) NOT NULL,
    CustomerName        VARCHAR(100) NOT NULL,
    DateOfBirth         DATE,
    OriginalAddress     VARCHAR(200),        -- The PO Box or blank address
    SystemsSearched     VARCHAR(500),        -- Comma-separated list of systems searched
    AddressFound        BIT NOT NULL,        -- 1 = found, 0 = not found
    PhysicalAddress     VARCHAR(200),        -- NULL if not found
    AddressSource       VARCHAR(50),         -- ImageCenter / actDataScout / ARCourts / Bankway
    AIConfidenceScore   VARCHAR(10),         -- high / low / none
    AIModel             VARCHAR(100),        -- Model used (e.g. claude-sonnet-4-6)
    ActionTaken         VARCHAR(50),         -- auto-resolved / analyst-queue / officer-escalated
    VerafinNoteSubmitted BIT,
    BankwayCIFUpdated   BIT,
    AnalystInitials     VARCHAR(10),         -- NULL if fully automated
    OfficerInitials     VARCHAR(10),         -- NULL if not escalated
    Notes               VARCHAR(1000)
);
```

---

## SharePoint List Equivalent (Power Automate option)

| Column Name | Type | Description |
|-------------|------|-------------|
| RunDate | Date | Date the workflow ran |
| RunTimestamp | Date/Time | Exact timestamp of record processing |
| AccountNumber | Single line of text | Customer account number |
| CustomerName | Single line of text | Full legal name searched |
| DateOfBirth | Date | DOB used in search |
| OriginalAddress | Single line of text | PO Box or blank address on file |
| SystemsSearched | Single line of text | Comma-separated: ImageCenter, actDataScout, ARCourts |
| AddressFound | Yes/No | Whether a physical address was located |
| PhysicalAddress | Single line of text | Address found (blank if none) |
| AddressSource | Choice | ImageCenter / actDataScout / ARCourts / Bankway |
| AIConfidenceScore | Choice | high / low / none |
| AIModel | Single line of text | Model name from Azure AI Foundry |
| ActionTaken | Choice | auto-resolved / analyst-queue / officer-escalated |
| VerafinNoteSubmitted | Yes/No | Whether the Verafin note was submitted |
| BankwayCIFUpdated | Yes/No | Whether Bankway was updated |
| AnalystInitials | Single line of text | Blank if automated |
| OfficerInitials | Single line of text | Blank if not escalated |
| Notes | Multiple lines of text | Any additional context |

---

## Retention

Per 31 CFR 1010.430, BSA records must be retained for a minimum of **five years**. This audit log must be:

- Retained for at least 5 years from the date of each entry
- Available for examiner review without manual reconstruction
- Exportable to CSV or Excel for regulatory requests

---

## Examiner Query Examples

Common examiner requests this log satisfies:

```sql
-- All records processed in a date range
SELECT * FROM BSA_Audit_Log 
WHERE RunDate BETWEEN '2026-01-01' AND '2026-12-31';

-- All auto-resolved records (no analyst involvement)
SELECT * FROM BSA_Audit_Log 
WHERE ActionTaken = 'auto-resolved';

-- All records escalated to BSA Officer
SELECT * FROM BSA_Audit_Log 
WHERE ActionTaken = 'officer-escalated';

-- Records where no address was found
SELECT * FROM BSA_Audit_Log 
WHERE AddressFound = 0;

-- Summary counts by action taken
SELECT ActionTaken, COUNT(*) AS RecordCount 
FROM BSA_Audit_Log 
GROUP BY ActionTaken;
```
