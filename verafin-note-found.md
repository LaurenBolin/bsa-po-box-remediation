# Verafin Note Template — Physical Address Found

Use this template when a physical address has been located through the automated search process.

---

## Template

```
Address remediation — [MM/DD/YYYY]. Checked Verafin and Bankway. 
Opened ImageCenter, actDataScout ([County] County), and ARCourts. 
Searched by customer name and DOB. 

Physical address located: [Street Address, City, State ZIP] 
Source: [ImageCenter / actDataScout / ARCourts]
AI confidence score: [High]

Bankway CIF updated: [Yes / No — reason if no]

— [Analyst Initials / AUTOMATED]
```

---

## Field Guidance

| Field | What to enter |
|-------|--------------|
| Date | MM/DD/YYYY format |
| County | Customer's county based on PO Box ZIP lookup |
| Physical address | Full street address as returned by source system |
| Source | Which system returned the address (ImageCenter, actDataScout, or ARCourts) |
| AI confidence score | High (auto-resolved), Low (analyst confirmed) |
| Bankway CIF updated | Yes if address was new; No with reason if not updated |
| Initials | Analyst initials for manual confirmations; "AUTOMATED" for auto-resolved records |

---

## Example (Auto-resolved)

```
Address remediation — 05/10/2026. Checked Verafin and Bankway. 
Opened ImageCenter, actDataScout (White County), and ARCourts. 
Searched by customer name and DOB. 

Physical address located: 123 Main Street, Searcy AR 72143
Source: actDataScout
AI confidence score: High — name and DOB exact match

Bankway CIF updated: Yes

— AUTOMATED
```

## Example (Analyst confirmed)

```
Address remediation — 05/10/2026. Checked Verafin and Bankway. 
Opened ImageCenter, actDataScout (White County), and ARCourts. 
Searched by customer name and DOB. 

Physical address located: 123 Main Street, Searcy AR 72143
Source: ARCourts
AI confidence score: Low — analyst confirmed match

Bankway CIF updated: Yes

— LB
```
