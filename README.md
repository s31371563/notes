Here’s a clear, structured, and polished rewrite of your conversation, keeping all the important technical details while improving readability and flow.


---

Refined Conversation Summary: COBRA Trigger & Event Processing Design

1. COBRA Trigger Actions Analysis

The team reviewed a master list of 74 actions that trigger COBRA.

Duplicate entries (same action + region with different effective dates) were removed.

Final filtered list identified valid COBRA-triggering actions.

Some actions (e.g., TSC, IOM, CTN) were observed to:

Insert records into PS_COBRA_ACTIVITY

But do not generate actual COBRA events


Decision:

These can still be inserted into COBRA activity.

Their downstream inactivity is not a blocker.




---

2. PS_BAS_PARTIC Table Debate (Audit vs Simplicity)

Proposal (Speaker 1):

Insert records into PS_BAS_PARTIC for:

Audit trail

Traceability of COBRA activity source


Reason:

PS_COBRA_ACTIVITY is temporary

Data is deleted after COBRA Admin job runs

Without BAS_PARTIC, source tracking is lost



Counterpoint (Speaker 2):

Prefer NOT inserting into PS_BAS_PARTIC

Goal:

Keep implementation lean

Avoid unnecessary data writes



Decision:

Do NOT populate PS_BAS_PARTIC (for now)



---

3. COBRA Event ID Confusion

COBRA Event IDs:

1 → Initial event

2 → Secondary event


Issue:

Some records show Event ID = 2 without an initial event


Status:

Unclear why this occurs

Will be investigated later

Not blocking current work




---

4. Architectural Direction: Table-Driven Design

Current Problem:

Too many hardcoded handlers per event class

Logic is:

Complex

Difficult to maintain

Frequently changing



New Approach:

Move to a table-driven system


Plan:

Create a custom BAS Action Table (simplified version)

Remove unused event classes

Keep only relevant data


Add:

COBRA triggers

Additional filtering logic (stored as JSON)



JSON-Based Conditions Example:

Store dynamic filtering logic in a generic field

Allows flexible rule definition without code changes



---

5. COBRA Trigger Storage Design

New Table (DCP Side):

A COBRA Trigger Table (child of BE_EVENT)


Fields:

BE_EVENT_ID (FK)

COBRA_EVENT_DATE

EVENT_CLASS

SYNC_STATUS (Boolean: Yes/No)


Important Notes:

COBRA event date:

May differ from BAS event date

Usually based on coverage begin date




---

6. Sync Strategy: HIP vs Facade

Options Considered:

1. Facade Service

Better logging

Easier retry handling



2. HIP Job

Already used for PeopleSoft sync

Built-in retry mechanism




Final Decision:

Use HIP Job


Reasoning:

Consistency with existing sync architecture

Centralized retry handling



---

7. Critical Sync Dependency

Key Rule:

COBRA data must NOT be synced until:

Core election data is synced to PeopleSoft


Why?

Avoid data inconsistency

Elections may still be pending in HIP job queue


Constraints:

HIP job:

Runs periodically (~every 5 minutes)

Delayed during:

Payroll blackout

Processing checks





---

8. Final Sync Flow

1. Event is submitted


2. Core tables updated in DCP


3. HIP job runs:

Syncs:

Core tables

COBRA triggers (together)




4. COBRA trigger marked:

SYNC_STATUS = YES





---

9. Admin Portal Design (New System)

Purpose:

Replace dependency on PeopleSoft Job Data triggers


Capabilities:

Manual event creation

Include COBRA actions directly


Requirement:

Add COBRA Action field in UI



---

10. API Changes

Endpoint:

createBEEvent


Enhancement:

Add new field:

cobraAction



Flow:

1. API receives payload


2. Sends message to Kafka


3. Event is processed


4. COBRA trigger stored in new table




---

11. Important Behavior Clarification

COBRA trigger is:

NOT executed at event creation

Only executed after:

Event reaches Submitted status




Why?

Ensures:

Elections exist

Data is ready for sync




---

12. Event Creation Logic (Current vs Future)

Current (PeopleSoft):

1. Job data change


2. BAS Activity record created


3. BenAdmin creates event



Future (New System):

Direct event creation via:

Admin Portal

API


No dependency on job data triggers



---

13. Special Case: FSC Events

FSC (Family Status Change):

Always manually created

Not triggered via job data


Some existing triggers may be:

Obsolete




---

14. Event Classification Logic Fix

Existing Logic:

Based on:

Wait period

Employment status



Issue:

Hawaii employees:

Can be "Full-time" at 20 hours


Fix required:

Adjust full-time determination logic




---

15. Future Enhancements

Use AI (Copilot / Opus model) to:

Generate rule tables

Convert logic into database-driven format




---

16. Tooling Note

Developer frustration:

SQL Developer cannot view BLOBs effectively


Preferred tool:

Toad (not available on Mac)




---

17. Action Items

Task	Owner	Status

Create COBRA Trigger Table	Developer	In Progress
Modify createBEEvent API	Developer	Pending
Add COBRA field in Admin Portal	Team	Pending
Build HIP Sync Job	Future Work	Not Started
Fix Full-Time Logic (Hawaii)	Developer	Pending
Design Table-Driven Rules	Team	Next Sprint



---

18. Sprint Guidance

Current sprint focus:

Add COBRA action field to API

Design trigger table approach

Document:

API changes

HIP-based sync strategy

Table-driven design





---

19. Meeting Follow-up

Schedule discussion with team (including Navya):

Planned for Monday (before standup)




---

Final Takeaways

Move toward:

Table-driven architecture

Deferred COBRA processing

HIP-based synchronization


Avoid:

Premature COBRA triggering

Unnecessary table writes (like BAS_PARTIC)


Ensure:

Data consistency

Scalability

Maintainability




---

If you want, I can also convert this into:

Low-level design (LLD) document

Database schema (DDL)

API contract (request/response JSON)
