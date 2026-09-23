# ExamConflictBot

> A UiPath RPA solution that detects examination scheduling conflicts and generates a structured Excel conflict report.

---

## Overview

**ExamConflictBot** automates the validation of examination schedules. Manual scheduling can silently produce:

- Room over-capacity
- Double-booked rooms
- Faculty assigned to overlapping exams
- Students assigned to overlapping exams
- Missing or invalid room assignments

The bot reads scheduling data from Excel, validates it against five conflict rules, logs every conflict to a centralized table, and outputs a structured `ConflictReport.xlsx`. Each validation rule is its own reusable workflow.

---

## Problem Statement

Checking a schedule manually means cross-referencing the timetable, room capacities, faculty assignments, and student enrollments simultaneously. At scale, this becomes slow and error-prone — a room can have enough capacity but already be booked at that time; a faculty member or student can be double-booked across two overlapping exams. ExamConflictBot automates that cross-referencing.

---

## Solution Workflow

```
Input Excel File → Read Input Data → Conflict Validation (R1–R5) → Central Conflict Table → ConflictReport.xlsx
```

---

## Conflict Detection Rules

| Rule | Checks | Example Output |
|---|---|---|
| **R1 — Room Capacity** | Enrolled students vs. room capacity | `R1｜EX09｜Overbooked: 35 students, capacity 30` |
| **R2 — Room Double Booking** | Same room, overlapping time | `R2｜EX04/EX05｜Room R04 has overlapping exams` |
| **R3 — Faculty Double Booking** | Same faculty, overlapping time | `R3｜EX07/EX08｜Faculty F07 has overlapping exams` |
| **R4 — Student Exam Clash** | Same student, overlapping time | `R4｜EX10/EX11｜Student S111 has overlapping exams` |
| **R5 — Assignment Integrity** | Missing or invalid room ID | `R5｜EX12｜Missing room assignment` |

R2–R4 share a reusable time-overlap check:

```vb
startDateTime1 < endDateTime2 AndAlso startDateTime2 < endDateTime1
```

Pair-based rules (R2–R4) use `j > i` when comparing exam pairs to avoid reporting the same conflict twice (e.g. both `EX04/EX05` and `EX05/EX04`).

---

## Project Architecture

```
ExamConflictBot/
│
├── Main.xaml
│
├── Workflows/
│   ├── ReadInputData.xaml
│   ├── ValidateRoomCapacity.xaml
│   ├── ValidateRoomDoubleBooking.xaml
│   ├── ValidateFacultyDoubleBooking.xaml
│   ├── ValidateStudentExamClash.xaml
│   ├── ValidateAssignmentIntegrity.xaml
│   └── CheckTimeOverlap.xaml
│
├── Data/
│   └── ExamSeating_MockData.xlsx
│
├── Output/
│   └── ConflictReport.xlsx
│
├── README.md
└── .gitignore
```

---

## Workflow Components

**`Main.xaml`** — Orchestrator. Initializes variables, creates the conflict report DataTable, invokes input reading and each validation workflow, generates the final Excel report, and logs execution info/errors.

**`ReadInputData.xaml`** — Loads four datasets (`TimeTable`, `Rooms`, `FacultyAllotment`, `StudentEnrollment`) from the input workbook, wrapped in `Try/Catch`.

**`ValidateRoomCapacity.xaml`** (R1) — For each exam, compares enrolled student count against the assigned room's capacity.

**`ValidateRoomDoubleBooking.xaml`** (R2) — Compares exam pairs for same room + overlapping time.

**`ValidateFacultyDoubleBooking.xaml`** (R3) — Matches exams to faculty, compares pairs for the same faculty across overlapping times.

**`ValidateStudentExamClash.xaml`** (R4) — Finds students enrolled in multiple exams and checks those exams for time overlap.

**`ValidateAssignmentIntegrity.xaml`** (R5) — Confirms every exam has a room assigned, and that the assigned room actually exists in the `Rooms` sheet.

**`CheckTimeOverlap.xaml`** — Shared overlap-check workflow, reused by R2, R3, and R4 to avoid duplicating logic.

---

## Input Data

The input workbook contains four sheets:

- **TimeTable** — `ExamID`, `Subject`, `SubjectCode`, `Date`, `StartTime`, `EndTime`, `RoomID`
- **Rooms** — `RoomID`, `Capacity`
- **FacultyAllotment** — `ExamID`, `FacultyID`
- **StudentEnrollment** — `StudentID`, `ExamID`

---

## Output

`Output/ConflictReport.xlsx` contains a single `Conflicts` sheet:

| RuleID | ExamID | Description |
|---|---|---|
| R1 | EX09 | Overbooked: 35 students, capacity 30 |
| R2 | EX04/EX05 | Room R04 has overlapping exams |
| R3 | EX07/EX08 | Faculty F07 has overlapping exams |
| R4 | EX10/EX11 | Student S111 has overlapping exams |
| R5 | EX12 | Missing room assignment |
| R5 | EX13 | Room R199 does not exist |

---

## Testing

Conflicts were seeded intentionally into the mock dataset to verify each rule fires correctly.

| Rule | Test Case | Result |
|---|---|---|
| R1 | EX09 exceeds room capacity | Detected |
| R2 | EX04 & EX05 share overlapping room | Detected |
| R3 | EX07 & EX08 share overlapping faculty | Detected |
| R4 | S111 has overlapping exams | Detected |
| R5 | EX12 has no room | Detected |
| R5 | EX13 has invalid room | Detected |

**Result: 6/6 conflicts detected — PASS ✅**

---

## Logging & Error Handling

- **Info logs**: execution start/end, input loading, validation completion, conflict count, report generation.
- **Warning logs**: fired per detected conflict (e.g. `R1 conflict detected for EX09: 35 students, capacity 30`).
- **Error logs**: fired on unexpected failures (e.g. `Failed to read input workbook: <exception details>`).

`ReadInputData.xaml` wraps reading in `Try/Catch` to handle a missing/invalid workbook path, missing sheets, or locked files. If input loading fails, `Main.xaml` halts instead of validating incomplete data.

---

## Key Design Decisions

- **Modular architecture** — each rule is an independent, reusable workflow, easier to debug, extend, and explain in a demo.
- **Centralized conflict table** (`dtConflictReport`) — all rules write to the same schema (`RuleID`, `ExamID`, `Description`), keeping output consistent.
- **Reusable time-overlap logic** — one workflow (`CheckTimeOverlap.xaml`) shared across R2–R4 instead of duplicated per rule.

---

## How to Run

1. **Clone the repo**: `git clone <YOUR-GITHUB-REPOSITORY-URL>`
2. **Open** `Main.xaml` in UiPath Studio.
3. **Confirm** `Data/ExamSeating_MockData.xlsx` exists and contains all four required sheets.
4. **Run** `Main.xaml`. It reads the Excel input, validates against R1–R5, builds the conflict table, and writes `ConflictReport.xlsx`.
5. **Open** `Output/ConflictReport.xlsx` → `Conflicts` sheet to view results.

**Prerequisites**: UiPath Studio, Microsoft Excel (or compatible), Windows.

---

## Current Status

**Working now:**
- [x] Excel input processing
- [x] R1–R5 conflict detection
- [x] Reusable time-overlap workflow
- [x] Centralized conflict report + Excel output
- [x] Error handling and structured logging

**Planned next phase — automatic resolution:**
- [ ] Automatic room reallocation
- [ ] Automatic faculty reassignment
- [ ] Automatic exam rescheduling
- [ ] Re-validation loop and final optimized schedule (`FinalExamSchedule.xlsx`)

Automatic resolution will require defining additional institutional constraints first (allowed time slots, faculty eligibility, room availability, max exams/day, department restrictions) — these aren't implemented yet, so resolution isn't attempted blindly.

---

## Skills Demonstrated

RPA development · Excel data processing · data validation · modular workflow design · exception handling · structured logging · reusable components · testing & debugging · Git & documentation.

---

## Technology Stack

| Technology | Purpose |
|---|---|
| UiPath Studio | RPA workflow development |
| Microsoft Excel | Input and output data |
| VB.NET Expressions | Data processing and validation logic |
| Windows | Execution environment |
| Git / GitHub | Version control and documentation |

---

## License

Developed for educational, learning, and portfolio purposes.

---

**ExamConflictBot** — Detect conflicts. Validate schedules. Build smarter automation.