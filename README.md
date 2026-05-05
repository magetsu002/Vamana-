# Vamana — Workflow Automation Engine

Vamana is a Python workflow engine that runs JSON-based workflows step by step with validation, safety checks, and logging.

This is the project I’ve been using to understand how real systems are built. Not just making something work, but making sure it behaves correctly even when input is bad or unexpected.

## Overview

You define a workflow in JSON:

```json
{
  "name": "Example Workflow",
  "steps": [
    {
      "type": "print",
      "message": "Hello, World!"
    },
    {
      "type": "wait",
      "seconds": 2
    },
    {
      "type": "write_file",
      "filename": "output.txt",
      "content": "Done"
    }
  ]
}
```

Run it:

```bash
python main.py workflows/basic_flow.json
```

## Example output

```text
basic_demo
[1/3] Executing print...
Starting workflow
[2/3] Executing wait...
[3/3] Executing print...
Finished

[Vamana Summary]
Workflow: basic_demo
Steps Completed: 3/3
Status: Workflow ran successfully
```

## Example logs

Successful run:

```text
[2026-05-05 12:52:20] --- Starting Workflow: basic_demo ---
[2026-05-05 12:52:20] Step 1/3: print
[2026-05-05 12:52:21] Step 2/3: wait
[2026-05-05 12:52:21] Step 3/3: print
[2026-05-05 12:52:21] SUCCESS: Workflow 'basic_demo' finished all 3 steps.
```

Failed runs:

```text
[2026-05-04 21:16:32] --- Starting Workflow: Error Test Workflow ---
[2026-05-04 21:16:32] Step 1/2: print
[2026-05-04 21:16:32] CRASHED on Step 2: File not found!

[2026-05-04 21:17:45] --- Starting Workflow: Security Test - Path Traversal ---
[2026-05-04 21:17:45] CRASHED on Step 1: Path traversal is not allowed :<
```

## Capabilities

- Loads workflows from JSON  
- Validates everything before execution  
- Enforces limits to prevent misuse  
- Executes steps using a handler system  
- Restricts file access to a safe workspace  
- Logs execution with timestamps  
- Detects crashes and reports them  

## Supported steps

```text
print       → output a message
wait        → pause execution
write_file  → create or overwrite a file
append_file → append content to a file
read_file   → read file content
```

## How it works

```text
load → validate → enforce limits → execute → log → summarize
```

## Project structure

```text
engine/
  config.py
  validation.py
  handlers.py
  runner.py
  security.py
  logging_utils.py

main.py
workflows/
tests/
```

Each part has a clear role:

- validation checks structure, types, and limits  
- handlers execute each step  
- runner controls execution flow  
- security handles safe file access  
- logging records everything  
- config defines limits and schemas  

## How I built this

The first version of this project worked, but it was fragile.

It didn’t validate input properly, it could crash easily, and file operations weren’t safe.

So instead of adding more features, I focused on fixing those problems and making it reliable first.

## What went wrong and how I fixed it

### Execution limits

- Workflows could freeze the program  
  Large wait values would block execution  
  → added a maximum wait limit  

- Multiple waits could still cause long execution  
  → added a total wait limit across the workflow  

- Too many steps could overload execution  
  → added a step limit  

### Validation issues

- Validation was too weak  
  I was only checking if fields existed  
  → enforced both field names and types  

- Workflow files could be too large  
  → added a file size check before loading  

- Text fields could be abused  
  → added limits on string length  

### File safety

- File access was unsafe  
  Absolute paths and directory traversal were possible  
  → blocked both  

- Symlinks could bypass checks  
  → resolved paths and ensured they stay inside the workspace  

- There was still a race condition risk  
  → switched to safer file opening using OS-level flags  

- The engine could overwrite its own files  
  → blocked protected filenames  

### Logging issues

- Logs could be manipulated  
  → sanitized log messages and removed newlines  

- Logs could get too large  
  → limited and truncated long messages  

- OS errors were inconsistent  
  → converted them into clean error messages  

### Structure problems

- Validation was duplicated  
  → removed redundant checks and cleaned the flow  

- Everything was in one file  
  → split the project into modules  

- Type hints were unclear  
  → replaced generic dict usage with structured types  

- The project directory got messy  
  → cleaned it using ignore rules  

## Example invalid workflow

```json
{
  "name": "invalid_example",
  "steps": [
    {
      "type": "write_file",
      "filename": "../secret.txt",
      "content": "hack"
    }
  ]
}
```

This will be rejected due to path traversal protection.

## What changed

This started as a simple script and gradually became more structured.

- added proper validation before execution  
- introduced limits to prevent unexpected behavior  
- restricted file access to a safe workspace  
- improved handling of edge cases  
- split the code into separate modules  

## Current limitation

- No database yet  

## What I’m building next

Right now I’m adding SQL to Vamana.

The goal is to store workflows and track every run instead of just executing once.

Planned:

- Store workflow definitions  
- Store run history  
- Track each step execution  
- Save errors and crashes  
- Query previous runs  

## Goal

The goal isn’t to compete with other tools.

It’s to understand how systems behave, where they break, and how to build them properly.

## Author

Built as a personal systems engineering project.
