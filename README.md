Overview of the Discussion
The meeting focused on addressing system failures and data inconsistencies during benefit defaulting (auto-defaulting or manual submission) when an employee has a pending or existing termination record in the system. The primary goal is ensuring that earlier effective-dated events (such as a New Hire event) can still process correctly even if a future-dated termination record is present.
Core Problem Statement & Business Requirement
 * The Business Context: A scenario arises when a new hire event exists (e.g., effective August 1st) without an initial submission, but a termination record is subsequently entered into the system for a later date (e.g., effective August 31st).
 * The Business Goal: The system must support "New Hire defaulting" when a termination exists. Whether initiated via automated ELG (Eligibility) defaulting or manual submission by an admin, the system needs to allow processing the August 1st event prior to executing the August 31st termination.
 * Current Failure Mode: When trying to run defaulting (WSE_defaulting) while a termination record exists, the process fails. This occurs because shared facades and queries perform checks that fail due to active-status filtering and missing effective-date parameters.
Technical Root Cause Analysis
1. Inconsistent Plan Availability Caching (Instance-Level vs. Multi-Pod)
 * Observed Issue: While testing plan availability calls with a termination record, results were inconsistent—sometimes returning valid data, sometimes returning null, even when passing refresh_cache = true.
 * Root Cause: The existing refresh_cache parameter only clears cache at the instance/pod level. In a multi-pod setup (e.g., 3 pods), clearing the cache on one pod leaves stale cached data on the remaining pods. Subsequent API requests routed to a different pod yield stale, cached results.
 * Work in Progress (Fix): Harsha is currently refactoring the caching mechanism during this sprint to store and clear cache centrally using Redis. This ensures cache invalidation occurs across all pods simultaneously.
2. Status-Filtering Limitations in get_employee_details
 * Observed Issue: The facade/query (get_employee_details) used during defaulting checks whether an employee is active.
 * Root Cause: The current facade query filters hardcoded active statuses without taking an effective date parameter into account. As a result, when an event is processed on August 1st for a user who has a termination record on August 31st, the query fails to properly retrieve the job data row for August 1st.
Proposed Solution & Workflow Architecture
[ New Hire Event (Aug 1) Pending ]  --->  [ Termination Entered (Aug 31) ]
                                                    |
                                                    v
                                   [ Pre-Action Check: Pending ELG Events? ]
                                                    |
                                                    v
                                   [ Execute WSE_Defaulting (Aug 1 Date) ]
                                                    |
                                                    v
                                   [ Process & Finalize Termination (Aug 31) ]

1. Refactoring Facade Queries (get_employee_details)
 * Effective Date Parameter: Update the query/facade to accept an explicit effective_date parameter.
 * Date-Specific Lookups: Instead of relying on standard system/active status checks, pass the event's effective date (e.g., August 1st). Because the employee is still active on August 1st, the query will correctly return the required job data and pay group information without failing.
2. Sequence Adjustment for Termination Events (ELG Pre-Action)
 * Default Order: Normally, when a termination event is created, post-event actions immediately load temporary tables and process core benefit table terminations.
 * Revised Order: Introduce a pre-action check prior to finalizing termination:
   * Create the termination record.
   * Check if open ELG/New Hire events exist prior to the termination date.
   * If ELG events exist, trigger WSE_defaulting for the earlier effective date using the updated date-aware facade.
   * Once ELG defaulting completes, proceed to load temp tables and process the termination event.
Action Items & Next Steps
 * Facade Query Testing (Speaker 2): Test passing an effective_date into the get_employee_details facade/query to confirm it retrieves active job data rows as of that specific date.
 * POC for ELG Defaulting (Speaker 2): Set up a Proof of Concept (POC) simulating a separation event to test executing ELG defaulting as a pre-action step before processing termination tables.
