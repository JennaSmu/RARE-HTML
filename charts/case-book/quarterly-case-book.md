---
dg-publish: true
---

```dataview
TABLE
	Intake,
	Outtake,
	InjuryEvent,
	Injury,
	Status
FROM "RARE Birds" AND -"RARE Birds/Species" AND -"RARE Birds/Ed Birds" AND -"RARE Birds/2221 RLHA"
WHERE (intake.year = 2025 AND intake.month = 4) OR (intake.year = 2025 AND intake.month = 5) OR (intake.year = 2025 AND intake.month = 6) OR outtake.month = null AND !contains(status, "Unknown")
SORT file.name ASC
```
