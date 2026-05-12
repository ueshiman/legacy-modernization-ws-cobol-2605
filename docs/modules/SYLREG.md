# SYLREG — Syllabus Registration Module

## Purpose

Registers new syllabus records into the indexed `syllabus.dat` file by collecting course metadata and a 15-week lesson plan from the operator via full-screen input forms. Input is validated by delegating to the shared `SYLCOM` module before the record is written.

## Per-paragraph documentation

### MAIN-PROCESS

- **Purpose**: Orchestrates the full syllabus registration session by invoking all sub-procedures in sequence inside a loop until the user chooses to stop.
- **Reads**: WS-EXIT (condition on WS-CONTINUE-FLAG)
- **Writes**: (none)
- **Calls**: OPEN-FILE, INITIALIZE-SYLLABUS-RECORD, INPUT-SYLLABUS-DATA, INPUT-WEEK-PLAN-DATA, WRITE-SYLLABUS-RECORD, CHECK-CONTINUE, CLOSE-FILE
- **Notes**: Terminates with GOBACK; the PERFORM UNTIL WS-EXIT loop drives the multi-registration flow.

### OPEN-FILE

- **Purpose**: Opens SYLLABUS-FILE in I-O mode, creating an empty file first if it does not yet exist.
- **Reads**: WS-FILE-STATUS (WS-FILE-NOT-FOUND condition on code "23")
- **Writes**: SYLLABUS-FILE (creates when absent)
- **Calls**: (none)
- **Notes**: Implements a create-on-open pattern — opens OUTPUT then immediately closes before reopening I-O; relies on file-status "23" to detect a missing file.

### CLOSE-FILE

- **Purpose**: Closes SYLLABUS-FILE to flush all pending I/O at the end of the session.
- **Reads**: SYLLABUS-FILE
- **Writes**: SYLLABUS-FILE
- **Calls**: (none)
- **Notes**: None

### INITIALIZE-SYLLABUS-RECORD

- **Purpose**: Clears the SYLLABUS-FILE-REC buffer to spaces/zeros before each new registration cycle.
- **Reads**: (none)
- **Writes**: SYLLABUS-FILE-REC
- **Calls**: (none)
- **Notes**: None

### INPUT-SYLLABUS-DATA

- **Purpose**: Displays the syllabus entry screen, accepts operator input, and calls SYLCOM with function code "C" to validate the course ID; re-invokes itself recursively on validation failure.
- **Reads**: SYLLABUS-INPUT-SCREEN, SYL-COURSE-ID, WS-RETURN-CODE, WS-RESULT
- **Writes**: WS-FUNCTION-CODE, WS-PARAM-1, WS-PARAM-2
- **Calls**: SYLCOM, INPUT-SYLLABUS-DATA (recursive PERFORM when WS-RETURN-CODE = 1)
- **Notes**: Magic constant "C" is passed as the function code to SYLCOM; recursive self-PERFORM on error creates an implicit retry loop with no iteration limit.

### INPUT-WEEK-PLAN-DATA

- **Purpose**: Displays the weekly lesson-plan screen and accepts input for all 15 weeks of the course.
- **Reads**: WEEK-PLAN-SCREEN, SYL-COURSE-ID, SYL-COURSE-NAME
- **Writes**: SYL-WEEK-PLAN(1) through SYL-WEEK-PLAN(15)
- **Calls**: (none)
- **Notes**: None

### WRITE-SYLLABUS-RECORD

- **Purpose**: Writes the populated SYLLABUS-FILE-REC to the indexed file and displays an error message if the course ID already exists.
- **Reads**: SYLLABUS-FILE-REC, SYL-COURSE-ID
- **Writes**: SYLLABUS-FILE, WS-ERROR-MSG
- **Calls**: (none)
- **Notes**: INVALID KEY branch constructs the error string via STRING; duplicate-key collision corresponds to file-status "22" (WS-FILE-DUP condition); no retry or abort logic is present after the error display.

### CHECK-CONTINUE

- **Purpose**: Prompts the operator to decide whether to register another syllabus entry and records the response.
- **Reads**: WS-MSG-CONTINUE
- **Writes**: WS-CONTINUE-FLAG
- **Calls**: (none)
- **Notes**: Accepts a single character (Y/N/y/n); the value written to WS-CONTINUE-FLAG is evaluated by the PERFORM UNTIL WS-EXIT condition in MAIN-PROCESS.

## Module dependencies

```mermaid
flowchart LR
    SYLREG --> SYLCOM
```

## Open questions

- (none — every paragraph fully understood)
