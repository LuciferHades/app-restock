# T+1 Recount Logic

## Purpose and Strategy
The primary goal of the T+1 logic is to minimize warehouse labor by avoiding a full recount. Instead, the system generates a targeted **Recount List** focusing only on items and locations that had movements on the day of the count or remained unresolved after the initial analysis.

## Input Requirements
- **Day T Results**: The reconciliation output from the previous day.
- **Day T In-Out**: The transaction log for the day the count was performed.
- **Recount Results (Optional)**: Any physical counts performed specifically for the recount list.

## Movement Impact Identification
The system identifies the "Movement Impact Set" by analyzing Day T transactions. A location is considered impacted if it was involved in any of the following:

| Movement Type | Impacted Location(s) |
| :--- | :--- |
| **IN** | `ToLocation` |
| **OUT** | `FromLocation` |
| **TRANSFER** | Both `FromLocation` and `ToLocation` |
| **REJ** | `FromLocation` and the `W8-REJ` area |
| **ADJUST** | All locations specified in the adjustment |

## Recount List Generation
The final **Recount List** is a combination of two sets:
1. **Movement Impact Set**: All locations where transactions occurred on Day T.
2. **Unresolved Variance Set**: Any locations from Day T that had a variance that could not be explained by existing evidence or matched for transfer.

## Operational Rules
- **Missing Location Data**: If a movement record lacks a specific location, the system will flag all locations containing that `ItemKey` for a recount.
- **Time Ambiguity**: If In-Out records only contain dates without timestamps, the system assumes the movement could have happened at any time and marks the location as impacted.
- **Finalization**: Once the recount results are uploaded, the system re-runs the reconciliation to produce the **Final Action Plan**, converting "Draft" statuses into "Confirmed" Loss or Over.
