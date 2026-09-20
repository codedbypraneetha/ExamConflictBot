# ExamConflictBot 🤖

### Automated Examination Conflict Detection using UiPath

**ExamConflictBot** is a modular **UiPath RPA** project designed to automate the validation of college examination schedules before finalization.

The bot reads examination-related data from Excel and performs rule-based validations to identify scheduling and resource conflicts. The project is being developed with a focus on **modularity, reusable workflows, structured logging, exception handling, and maintainability**.

> 🚧 **Project Status:** In Development  
> Current implementation: **Input Data Processing + R1 + R2**  
> Remaining validations will be implemented in the next development stage.

---

## 🎯 Problem Statement

Manually validating examination schedules can be time-consuming and error-prone, especially when multiple exams share rooms, faculty members, and students.

ExamConflictBot aims to automate this process by checking the examination data against predefined validation rules and generating a centralized conflict report.

The bot is designed to detect issues such as:

- Examination rooms exceeding their capacity
- Two exams being scheduled in the same room at overlapping times
- Faculty members being assigned to overlapping exams
- Students having overlapping examinations
- Missing or invalid room assignments

---

## 🏗️ Current Architecture

```text
ExamConflictBot
│
├── Main.xaml
│   └── Orchestrates the complete automation
│
├── Workflows
│   ├── ReadInputData.xaml
│   ├── ValidateRoomCapacity.xaml
│   ├── ValidateRoomDoubleBooking.xaml
│   ├── CheckTimeOverlap.xaml
│   ├── ValidateFacultyDoubleBooking.xaml      [Planned]
│   ├── ValidateStudentExamClash.xaml          [Planned]
│   └── ValidateAssignmentIntegrity.xaml       [Planned]
│
├── Data
│   └── ExamSeating_MockData.xlsx
│
├── Output
│   └── ConflictReport.xlsx                    [Final stage]
│
└── README.md
```

---

## ⚙️ Technology Stack

- **UiPath Studio**
- **Microsoft Excel**
- **VB.NET Expressions**
- **DataTables**
- **UiPath Excel Activities**
- **Git & GitHub**

---

# 📊 Input Data

The automation currently works with four Excel sheets.

### 1. TimeTable

Contains examination scheduling information:

| Column | Description |
|---|---|
| ExamID | Unique examination identifier |
| Subject | Examination subject |
| SubjectCode | Subject code |
| Date | Examination date |
| StartTime | Examination start time |
| EndTime | Examination end time |
| RoomID | Assigned examination room |

### 2. Rooms

Contains room capacity information:

| Column | Description |
|---|---|
| RoomID | Unique room identifier |
| Capacity | Maximum number of students |

### 3. FacultyAllotment

Maps faculty members to examinations:

| Column | Description |
|---|---|
| ExamID | Examination identifier |
| FacultyID | Assigned faculty member |

### 4. StudentEnrollment

Maps students to their examinations:

| Column | Description |
|---|---|
| StudentID | Student identifier |
| ExamID | Examination identifier |

---

# ✅ Implemented Validations

## R1 — Room Capacity Validation

Checks whether the number of students enrolled for an examination exceeds the capacity of the assigned room.

### Example

```text
Exam: EX09
Students: 35
Room: R08
Capacity: 30
```

Result:

```text
R1 | EX09 | Overbooked: 35 students, capacity 30
```

The validation also safely handles rooms that do not exist in the Rooms sheet, allowing invalid room assignments to be handled by a separate validation rule.

---

## R2 — Room Double-Booking Validation

Checks whether two examinations are assigned to the **same room at overlapping times**.

The project uses a reusable `CheckTimeOverlap.xaml` workflow to determine whether two examination time intervals overlap.

### Example

```text
EX04 → Room R04 → 09:00–12:00
EX05 → Room R04 → 10:00–13:00
```

Result:

```text
R2 | EX04/EX05 | Room R04 has overlapping exams
```

The comparison logic uses unique row pairs to avoid reporting the same conflict twice.

For example:

```text
EX04 + EX05
```

is reported once instead of:

```text
EX04 + EX05
EX05 + EX04
```

---

# 🧩 Reusable Time-Overlap Workflow

`CheckTimeOverlap.xaml` was created as a reusable component instead of duplicating date/time comparison logic across every validation workflow.

It accepts:

```text
Date 1
Start Time 1
End Time 1
Date 2
Start Time 2
End Time 2
```

and returns:

```text
IsOverlap → Boolean
```

The core overlap condition is:

```vb
startDateTime1 < endDateTime2 AndAlso
startDateTime2 < endDateTime1
```

This workflow can be reused for:

- Room conflicts
- Faculty conflicts
- Student conflicts

---

# 📝 Current Execution Result

The current version successfully reads:

```text
Timetable rows:          13
Rooms rows:               8
Faculty rows:            13
Student enrollment rows: 132
```

Current detected conflicts:

```text
R1 → EX09 overbooked
R2 → EX04/EX05 room overlap
```

Current conflict count:

```text
2
```

Example execution log:

```text
[Information] Input data loaded successfully.
[Warning] R1 conflict detected for EX09: 35 students, capacity 30
[Information] R1 validation completed. Conflicts found so far: 1
[Warning] R2 conflict detected: Room R04 has overlapping exams EX04 and EX05
[Information] R2 validation completed. Conflicts found so far: 2
[Information] EXAMCONFLICTBOT execution ended
```

---

# 🚧 Development Roadmap

The following validations are planned for the next development stage:

### R3 — Faculty Double-Booking

Detect when the same faculty member is assigned to two overlapping examinations.

Expected test case:

```text
Faculty: F07
EX07 → 09:00–12:00
EX08 → 10:00–13:00
```

---

### R4 — Student Exam Clash

Detect when the same student is enrolled in two examinations that overlap.

Expected test case:

```text
Student: S111
EX10 → 09:00–12:00
EX11 → 10:00–13:00
```

---

### R5 — Assignment Integrity

Validate examination room assignments and identify:

- Missing RoomID
- RoomID that does not exist in the Rooms sheet

Example test cases:

```text
EX12 → Missing RoomID
EX13 → Invalid RoomID: R199
```

---

### Final Reporting

The completed project will generate a centralized conflict report using the following structure:

| RuleID | ExamID | Description |
|---|---|---|
| R1 | EX09 | Overbooked room |
| R2 | EX04/EX05 | Room overlap |
| R3 | EX07/EX08 | Faculty overlap |
| R4 | EX10/EX11 | Student clash |
| R5 | EX12 | Missing room |
| R5 | EX13 | Invalid room |

---

# 🛡️ Error Handling & Logging

The project is being developed with reliability and traceability in mind.

Current implementation includes:

- `Try/Catch` around input data processing
- Validation of successful input loading
- Structured `Information`, `Warning`, and `Error` logs
- Centralized conflict reporting
- Reusable validation workflows

Planned improvements include additional exception handling around individual validation stages and final report generation.

---

# 📁 Project Structure

```text
ExamConflictBot/
│
├── Main.xaml
│
├── Workflows/
│   ├── ReadInputData.xaml
│   ├── ValidateRoomCapacity.xaml
│   ├── ValidateRoomDoubleBooking.xaml
│   ├── CheckTimeOverlap.xaml
│   ├── ValidateFacultyDoubleBooking.xaml
│   ├── ValidateStudentExamClash.xaml
│   └── ValidateAssignmentIntegrity.xaml
│
├── Data/
│   └── ExamSeating_MockData.xlsx
│
├── Output/
│   └── ConflictReport.xlsx
│
└── README.md
```

> Some workflows and output files shown above are part of the planned final architecture and may not yet be implemented in the current version.

---

# ▶️ How to Run

### Prerequisites

- UiPath Studio
- Microsoft Excel
- UiPath Excel activities
- Windows environment

### Steps

1. Clone or download the repository.
2. Open the project in **UiPath Studio**.
3. Ensure the input Excel workbook is available in the `Data` folder.
4. Open `Main.xaml`.
5. Verify the workbook path used by the project.
6. Run `Main.xaml`.
7. Monitor the execution logs for detected conflicts.

---

# 🧪 Testing Strategy

The mock dataset intentionally contains known conflicts so that each validation rule can be tested independently.

Current test cases:

| Test Case | Expected Result | Status |
|---|---|---|
| EX09 exceeds room capacity | R1 conflict | ✅ |
| EX04 & EX05 overlap in R04 | R2 conflict | ✅ |
| EX07 & EX08 overlap for F07 | R3 conflict | 🚧 |
| S111 has overlapping exams | R4 conflict | 🚧 |
| EX12 has missing room | R5 conflict | 🚧 |
| EX13 has invalid room R199 | R5 conflict | 🚧 |

---

# 📌 Future Improvements

Planned improvements include:

- Complete all validation modules
- Generate the final Excel conflict report
- Add more comprehensive test cases
- Improve exception handling
- Add configuration-based input/output paths
- Add detailed execution summary
- Add screenshots/demo recording
- Document Git development stages and commits

---

## 👩‍💻 Author

**Praneetha V.**

Computer Science Engineering Student  
Interested in **RPA, Automation, Software Development, and Problem Solving**

---

## 📜 Project Status

**Currently in active development.**

The repository represents the development progress of the project. The current version demonstrates successful Excel data ingestion, modular workflow execution, room-capacity validation, room double-booking detection, reusable time-overlap logic, and structured execution logging.

More validation modules and final reporting functionality will be added in subsequent development stages.
