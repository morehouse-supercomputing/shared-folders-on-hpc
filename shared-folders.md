---
name: Shared Folders on HPC
project_type: workshop
domain:
  - career
status: active
started:
target: ongoing
entities:
  - "[[morehouse]]"
  - "[[mscf]]"
people:
  - "[[scruse-ashley]]"
program: "[[mscf]]"
tags:
  - workshop
  - module
  - template
  - mscf
  - morehouse
  - hpc
  - reusable
---
# Shared Folders on HPC

Reusable guide for setting up shared directories on HPC for classes, labs, and research teams. Maintained by MSCF.

**Status:** Active (reusable template; periodic maintenance)

## Quick links
- Owner: [[mscf]]
- Public guide: https://morehouse-supercomputing.github.io/shared-folders-on-hpc/

## Tasks
- [ ] 

## Overview

### Goal
Document how to set up shared HPC directories so that classes, labs, and research teams can collaborate on shared compute resources without each user needing to manage permissions individually.

### Why this matters
Shared directories are a recurring need that comes up every time an instructor wants their class to use HPC, or a research group wants a shared workspace. Centralizing the instructions as a public guide removes the friction of explaining the same setup process repeatedly. Like other [[modules|modules]] in this folder, it's built once and adopted on demand.

### Public guide
The canonical guide lives at the GitHub Pages URL above. The local `docs/` folder is the source.

## Live dashboard

### Open tasks
```dataview
TASK
FROM "morehouse/workshops/modules/shared-folders"
WHERE !completed AND (
  contains(file.path, "shared-folders.md") OR
  contains(string(tags), "#task")
)
GROUP BY file.link
SORT due ASC
```

### Recently completed
```dataview
TASK
FROM "morehouse/workshops/modules/shared-folders"
WHERE completed AND (
  contains(file.path, "shared-folders.md") OR
  contains(string(tags), "#task")
)
GROUP BY file.link
SORT completion DESC
LIMIT 5
```
