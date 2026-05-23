# Master Context

## Project Name
Restock Reconciliation Tool

## Business Problem
The current restock process faces several challenges:
- **Non-Real-Time Inquiry**: System stock snapshots (Inquiry) are taken at specific times (e.g., 08:00) and do not reflect subsequent movements.
- **Delayed Movement Data**: In-Out transaction data for the current day is only available the following day.
- **Physical Discrepancies**: Actual stock on the floor (Physical Count) often differs from system records due to location mismatches or unrecorded movements.
- **Complex Rules**: Specific products like "RE T1" or "Pure Sugar T1" cannot be substituted across different customers, requiring strict reconciliation logic.

## Goal
To create a lightweight, single-user web application that automates the analysis of stock discrepancies and generates actionable transfer plans and recount lists, significantly reducing the time spent on manual reconciliation.

## Core Workflow
### Day T: Baseline Analysis
1. **Upload**: User uploads a single Excel file containing Inquiry, In-Out (up to yesterday), and Physical Count.
2. **Reconcile**: System runs baseline reconciliation between Inquiry and Physical Count.
3. **Draft**: System identifies matches, location mismatches, and potential variances.
4. **Export**: User exports a draft result and a transfer plan for immediate actions.

### Day T+1: Movement Impact & Finalization
1. **Upload**: User uploads the In-Out data for Day T.
2. **Impact Analysis**: System identifies which items/locations were affected by Day T movements.
3. **Recount List**: System generates a targeted recount list for only the affected items/locations.
4. **Finalize**: After recount results are uploaded, the system generates the final action plan.

## Key Principles
- **Physical Count = Field Truth**: The actual count is the primary reference for what is physically present.
- **Inquiry = System Truth**: The snapshot represents the system's state at a specific point in time.
- **In-Out = Evidence**: Transactions explain why the system and physical counts might differ.
- **Draft vs. Confirmed**: Results are considered "Draft" on Day T and only "Confirmed" after T+1 movement data is processed.

## Non-goals
- Full ERP or WMS functionality.
- Multi-user roles or complex approval workflows.
- Real-time integration with external ERP systems.
- Capacity simulation or cross-warehouse custody management in version 1.
