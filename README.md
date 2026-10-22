# Students Register Management

[![Python Version](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![Architecture](https://img.shields.io/badge/Architecture-Layered-orange.svg)]()
[![Pattern](https://img.shields.io/badge/Pattern-Command%20%7C%20Repository-lightgrey.svg)]()
[![Storage](https://img.shields.io/badge/Storage-Memory%20%7C%20Text%20%7C%20Pickle-green.svg)]()

A Python console application for managing students, disciplines, and grades. Built with a layered architecture, configurable repository backends (in-memory, text file, binary pickle), grade statistics, and an undo/redo system based on the command pattern.

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Features](#features)
- [Repository Structure](#repository-structure)
- [Configuration](#configuration)
- [Installation and Run](#installation-and-run)
- [CLI Usage](#cli-usage)
- [Implementation Details](#implementation-details)
- [License](#license)

---

## Overview

The application provides student enrollment and grade management through a text-based menu interface. It separates domain models, data access, business logic, and presentation into distinct layers.

Main components:
- **Storage options**: Supports in-memory storage, plain-text files, and serialized binary (pickle) files, configured via `settings.properties`.
- **Undo / Redo**: Uses the Command design pattern with support for cascaded operations (e.g. restoring a deleted student also restores their grades).
- **Statistics**: Queries for failing students, student ranking by overall average, and disciplines ordered by student performance.

---

## Architecture

```
 ┌──────────────────────────────────────────┐
 │         User Interface (Console)         │  ui/ui.py
 └────────────────────┬─────────────────────┘
                      │ calls
 ┌────────────────────▼─────────────────────┐
 │              Service Layer               │  service/service.py
 │   (Business logic, validation, undo)     │  service/undo_service.py
 └────────────────────┬─────────────────────┘
                      │ calls
 ┌────────────────────▼─────────────────────┐
 │             Repository Layer             │  repository/
 │    (MemoryRepo, TextfileRepo, PickleRepo)│  - in-memory, text, binary
 └────────────────────┬─────────────────────┘
                      │ creates / uses
 ┌────────────────────▼─────────────────────┐
 │               Domain Layer               │  domain/
 │       (Student, Discipline, Grade)       │  - entities
 └──────────────────────────────────────────┘
```

### Repositories
All storage implementations share the interface defined in `repository/memoryRepo.py`:
- **In-Memory (`Repository`)**: Stores entities in Python lists; includes test data generation.
- **Text Files (`TextRepository`)**: Reads from and writes to formatted `.txt` files.
- **Binary Files (`PickleRepository`)**: Serializes entities using Python's `pickle` module.

### Undo / Redo Mechanism
- State-changing actions create an `Operation` containing an undo `Command` and a redo `Command`.
- Actions affecting multiple records (such as deleting a student and their associated grades) use `CascadedOperation` to roll back or reapply all related operations together.

---

## Features

### 1. Student and Discipline Management
- CRUD operations (create, read, update name, delete) for students and disciplines.
- Validation for numeric IDs, unique IDs, and unique discipline names.

### 2. Grading
- Assign grades (range `(0, 10]`) linking a student ID to a discipline ID.
- Validates student existence, discipline existence, and grade range.

### 3. Search
- Search students or disciplines by ID or Name.
- Performs case-insensitive substring matching.

### 4. Statistics
- **Failing students**: Lists students with an average below 5.00 in at least one discipline.
- **Student leaderboard**: Ranks students by overall grade average across all disciplines in descending order.
- **Disciplines by average**: Ranks disciplines by the average of all grades assigned to that discipline in descending order.

### 5. Undo / Redo
- Tracks history for add, update, remove, and grading actions.
- Re-executes or reverses operations via internal command stacks.

---

## Repository Structure

```text
Students-Register-Management/
├── domain/                      # Domain entities
│   ├── discipline.py            # Discipline class (id, name)
│   ├── grade.py                 # Grade class (discipline_id, student_id, value)
│   └── student.py               # Student class (id, name)
│
├── repository/                  # Data access layer
│   ├── memoryRepo.py            # Base in-memory repository
│   ├── pickleRepo.py            # Pickle-based file repository
│   └── textfileRepo.py          # Text-based file repository
│
├── service/                     # Business logic and operations
│   ├── service.py               # Validation, statistics, search, data coordination
│   └── undo_service.py          # Command, Operation, CascadedOperation, UndoService
│
├── ui/                          # Presentation layer
│   └── ui.py                    # Menu-driven console interface
│
├── utils/                       # Common utilities
│   └── exceptions.py            # Custom exceptions
│
├── disciplines.txt              # Text data for disciplines
├── grades.txt                   # Text data for grades
├── students.txt                 # Text data for students
├── disciplines.pickle           # Binary data for disciplines
├── grades.pickle                # Binary data for grades
├── students.pickle              # Binary data for students
├── settings.properties          # Repository configuration file
├── main.py                      # Application entry point
└── README.md                    # Project documentation
```

---

## Configuration

Persistence is configured in `settings.properties`:

```properties
# Options: 'inmemory', 'textfiles', 'binaryfiles'
repository=textfiles

# File paths:
students=students.txt
disciplines=disciplines.txt
grades=grades.txt
```

To switch to binary mode:
```properties
repository=binaryfiles
students=students.pickle
disciplines=disciplines.pickle
grades=grades.pickle
```

To switch to in-memory mode:
```properties
repository=inmemory
```

---

## Installation and Run

### Requirements
- Python 3.8+
- `jproperties`

### 1. Clone
```bash
git clone https://github.com/andreiu77/Students-Register-Management.git
cd Students-Register-Management
```

### 2. Install dependencies
```bash
pip install jproperties
```

### 3. Run
Set `PYTHONPATH` to the project root:

**Windows (PowerShell):**
```powershell
$env:PYTHONPATH = ".;$env:PYTHONPATH"
python main.py
```

**Linux / macOS:**
```bash
PYTHONPATH=. python3 main.py
```

---

## CLI Usage

Main menu options:

```text
1. Manage students and disciplines
2. Grade a student
3. Search
4. Statistics
5. Exit
6. Undo
7. Redo
```

- **Option 1**: Submenu to add, remove, update, or list students or disciplines.
- **Option 2**: Prompts for `student_id`, `discipline_id`, and `grade`.
- **Option 3**: Search either entity by ID or name substring.
- **Option 4**: Run one of the 3 statistical queries.
- **Option 6 / 7**: Undo or redo the previous operation.

---

## Implementation Details

- **Validation and error handling**: Input validation raises domain-specific exceptions (`InvalidInput`, `InvalidId`, `InvalidGrade`, `InvalidDiscipline`, `UndoRedoException`) which are caught and displayed by the UI without terminating the program.
- **Command storage**: The undo service stores function references and arguments rather than copying state snapshots, reducing memory overhead.
- **Cascaded operations**: Deleting a student or discipline automatically removes related grades. Undo restores both the entity and the removed grades.

---


