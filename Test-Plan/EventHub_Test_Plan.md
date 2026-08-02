# EventHub — Test Plan

**Based on:** EventHub Requirement Analysis
**Testing window:** Sun Aug 2 – Sun Aug 9, 2026 (7 days)
**Owner:** QA

---

## 1. Objective
Verify that the EventHub MVP (Web + Android + iOS) meets the functional requirements, business rules, and acceptance criteria defined in the Requirement Analysis, within the remaining sprint window.

---

## 2. Scope

**In Scope:** Authentication, Event Creation, Vendor Search & Filter, Vendor Profile & Work Postings, Booking, Payment, Budget Manager, Favorites, Reviews, Vendor Dashboard, Admin Dashboard.

**Out of Scope:** Loyalty Program, AI Smart Recommendations, Corporate Events, Birthday Events, Baby Shower, Digital Contracts, Push Notifications, Live Chat, Event Timeline, Interactive Checklist.

---

## 3. Test Types
- **Functional testing** — each module against its FR / BR / AC.
- **UI testing** — responsive Web, Android, iOS.
- **Integration testing** — full Customer / Vendor / Admin journeys end-to-end (not module-by-module).
- **Regression testing** — one focused pass before final delivery.
- **Automation** — not in scope given the timeline; manual testing only for this sprint.

---

## 4. Test Environment
- Responsive Web (desktop + mobile browser)
- Android, iOS (native app)
- Note: Database, Authentication provider, File Storage, and Maps are still marked TBD in the tech stack. This is an infrastructure gap, separate from the functional Open Questions in the Requirement Analysis — ask the Dev team directly: which service/library was chosen for each, and whether QA needs any sandbox/test credentials for features that touch them (image upload, city/location picker, etc.).

---

## 5. Entry & Exit Criteria

**Entry (per module):** Module is code-complete and deployed to a testable environment; its Requirement Analysis section is finalized.

**Exit (per module):** All Acceptance Criteria pass · no open Blocker/High severity bugs · known Medium/Low bugs are logged and acknowledged by the dev team.

**Exit (overall):** All P0 modules pass their exit criteria · the three end-to-end journeys (Customer, Vendor, Admin) complete without a Blocker · final regression pass shows no new Blocker/High bugs.

---

## 6. Test Priorities (risk-based)

| Priority | Modules | Why |
|---|---|---|
| P0 | Authentication, Booking, Payment, Budget Manager | Core flow, money-handling, security |
| P1 | Vendor Profile & Work Postings, Vendor Dashboard, Admin Dashboard, Vendor Search & Filter | Approval workflows, cross-role visibility |
| P2 | Event Creation, Favorites, Reviews | Lower risk, simpler logic |

If time runs short, P2 gets reduced coverage before P0/P1 does — never the other way around.

---

## 7. Tools & Traceability
- **Test cases:** one Excel/CSV file per module, stored under `Modules/<module>/test-cases.xlsx`.
- **Traceability:** each test case row includes a `Requirement ID` column (FR-xx / BR-xx / US-xxx) — no separate RTM document; a summary can be compiled from these columns at the end if needed.
- **Bug tracking:** GitHub Issues, one issue per bug, linked to the relevant module.
- **Requirement source:** `Analysis/EventHub_Requirement_Analysis.md`

---

## 8. Schedule (suggested — adjust to actual dev delivery dates)

| Day | Focus |
|---|---|
| Mon Aug 3 – Tue Aug 4 | Resolve blocking Open Questions with the team · finish remaining module analyses · write test cases for modules already stable (Auth, Event Creation) |
| Wed Aug 5 – Fri Aug 7 | Execute functional testing as each module is delivered, in priority order (P0 → P1 → P2) · log bugs · retest fixes |
| Sat Aug 8 | Integration testing across the 3 end-to-end journeys · regression pass on P0 modules |
| Sun Aug 9 | Final regression · acceptance criteria checklist · close out bug list · sign-off |

*Note: the Requirement Analysis is a living document. As each Open Question is answered, update the relevant module's section in `Analysis/EventHub_Requirement_Analysis.md`, and revisit any test cases already written against the old assumption.*

---

## 9. Risks
- Payment gateway provider not yet chosen — blocks Payment module testing until resolved.
- Tech stack TBDs (Database, Auth, File Storage, Maps) may delay environment readiness.
- Spent Budget double-counting (Accepted vs Paid) is unresolved and could invalidate Budget Manager testing until clarified.
- No Admin review SLA defined — could delay Vendor/Work Posting approval testing.
- 7-day window is tight relative to Payment integration complexity; P2 modules are the planned buffer if schedule slips.

---

## 10. Roles & Responsibilities
Small team, no PO/Lead — QA (Requirement Analysis, Test Planning, Test Execution, Bug Reporting), Backend Dev, Mobile Dev, AI integration each own their module's fixes. Open Questions are resolved by quick team sync, not a formal PO sign-off.
