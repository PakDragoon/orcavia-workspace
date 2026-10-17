# 03 — Time & Attendance

## 1. Concepts

- **Work schedule**: expected working pattern (fixed hours, flexible core hours, shift-based, or no tracking).
- **Shift**: start/end, break rules, grace periods, night-shift spanning midnight (attendance date = shift start date).
- **Roster**: assignment of shifts to employees per day/week.
- **Punch**: raw clock event (`timestamptz`, source, device, location, photo/selfie optional).
- **Attendance day**: computed per employee per calendar `date`: status (Present, Absent, Late, Early-out, Half-day,
  On leave, Holiday, Weekend, Remote, On duty/travel), worked minutes, overtime minutes, exceptions.

## 2. Requirements

| ID | Requirement | Ver |
|---|---|---|
| ATT-001 | Clock-in/out sources: web, mobile (GPS + optional selfie), kiosk mode (shared tablet with PIN/QR), biometric devices via integration (ZKTeco and similar — import or push API), CSV import. | V1 (devices V1 via import, live push V2) |
| ATT-002 | Geofencing per location (radius or polygon) with policy: block, allow-and-flag, or allow. Mock-location detection on mobile. | V1 |
| ATT-003 | Face recognition / liveness for mobile and kiosk punches, opt-in per tenant, explicit employee consent, templates deleted on exit. | V2 |
| ATT-004 | Shift management: shift templates, rotating patterns, roster planner (drag-drop calendar), publish roster to employees, shift swap requests with manager approval, open-shift bidding [V2]. | V1 (swaps V2) |
| ATT-005 | Rules engine per policy: grace minutes, late/early thresholds, half-day thresholds, minimum hours, break deduction, rounding, max hours/day. | V1 |
| ATT-006 | Daily processing job computes attendance days; recompute when leave/holiday/roster/punch changes (event-driven). | V1 |
| ATT-007 | Exceptions: missed punch, late, early-out, absent without leave → employee regularization request (reason + evidence) → workflow. Monthly cap on regularizations configurable. | V1 |
| ATT-008 | Late/absent penalties policy (e.g. 3 lates = half-day deduction) feeding payroll as LOP inputs — configurable, explainable. | V1 |
| ATT-009 | Overtime: eligibility per grade/employment type, pre-approval or post-approval, rate multipliers (normal day, rest day, public holiday, night), caps, compensatory off instead of pay option. Feeds payroll. | V1 |
| ATT-010 | Remote work / WFH: requests by date range with approval, attendance status "Remote", optional check-in location capture. | V1 |
| ATT-011 | On-duty / business travel / field visit requests; field staff visit logging with GPS trail [V3]. | V1 (trail V3) |
| ATT-012 | Holiday calendars per location; PK federal + provincial holidays and Islamic holidays with tentative dates confirmed later (moon-sighting) — HR can shift date and system recomputes. | V1 |
| ATT-013 | Ramadan timings: alternate schedule for a date range per location/employee group. | V1 |
| ATT-014 | Manager team view: who's in/out/late now, calendar heatmap, pending exceptions. | V1 |
| ATT-015 | Monthly attendance lock per pay period before payroll; post-lock corrections flow as payroll adjustments in the next period. | V1 |
| ATT-016 | Timesheets (project/task time) for billable or project-tracked staff, with approval and project cost reports. | V2 |
| ATT-017 | AI anomaly flags: buddy-punching patterns, impossible travel, chronic missed punches — shown to HR as insights, never automatic penalties. | V2 |

## 3. Explainability

Every attendance day shows: expected schedule, punches, rules applied, resulting status, and which payroll
input it produced. Employees see the same breakdown for their own days.
