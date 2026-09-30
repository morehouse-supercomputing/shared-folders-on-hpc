---
layout: default
title: Shared Folders on HPC
---

# Shared Folders on HPC

How to create directories on HPC that your students, collaborators, or research team can access. Useful for distributing datasets, sharing code, or collecting results.

---

## How Sharing Works on HPC

HPC systems use standard Unix file permissions. Every file and folder has three levels of access:

| Who | Meaning |
|-----|---------|
| **Owner** (u) | You — the person who created it |
| **Group** (g) | Everyone on your HPC allocation |
| **Others** (o) | Everyone else on the system |

When users are added to the same HPC allocation (project), they share a Unix group. This means you can give group-level access to files without opening them to the entire system.

---

## Quick Setup

If you just need the commands:

```bash
# Create the shared folder
mkdir $WORK/shared

# Let your group read from it
chmod 755 $WORK/shared

# Give students your full path
echo $WORK/shared
```

Give students the full path that gets printed (e.g., `/work/12345/your_username/system_name/shared`). They access it with that path — `$WORK` only points to their own directory.

> Replace `system_name` with the HPC system you're on: `vista`, `ls6`, `stampede3`, `frontera`, etc. This works the same on all systems.

---

## Step-by-Step

### Step 1: Check Your Group

```bash
groups
```

You'll see something like:

```
G-12345 your_username
```

That `G-12345` is your allocation group. Everyone on the same allocation shares this group.

### Step 2: Create the Shared Directory

```bash
mkdir $WORK/shared
```

Call it whatever makes sense: `shared`, `class_data`, `dataset`, `team_files`, etc.

### Step 3: Set Permissions

Choose the access level you want:

**Read-only (students can view and copy, not modify):**

```bash
chmod 755 $WORK/shared
```

| Permission | Owner | Group | Others |
|-----------|-------|-------|--------|
| Read | Yes | Yes | Yes |
| Write | Yes | No | No |
| Execute (enter directory) | Yes | Yes | Yes |

This is the safest option for distributing datasets or starter code.

**Read and write (students can add and modify files):**

```bash
chmod 775 $WORK/shared
```

| Permission | Owner | Group | Others |
|-----------|-------|-------|--------|
| Read | Yes | Yes | Yes |
| Write | Yes | Yes | No |
| Execute | Yes | Yes | Yes |

Use this when students need to submit files to a shared location.

**Group-only (only your allocation members, not the whole system):**

```bash
chmod 770 $WORK/shared
```

| Permission | Owner | Group | Others |
|-----------|-------|-------|--------|
| Read | Yes | Yes | No |
| Write | Yes | Yes | No |
| Execute | Yes | Yes | No |

Use this for research teams that need privacy.

### Step 4: Get the Full Path

```bash
echo $WORK/shared
```

This prints something like:

```
/work/12345/your_username/system_name/shared
```

**Give students this full path.** They cannot use `$WORK/shared` because `$WORK` expands to their own directory, not yours.

### Step 5: Students Access the Folder

Students run:

```bash
ls /work/12345/your_username/system_name/shared
```

Or copy files to their own space:

```bash
cp -r /work/12345/your_username/system_name/shared/dataset $WORK/my_copy
```

---

## Common Scenarios

### Distribute a Dataset to a Class

You want every student to have access to the same large dataset without each student downloading it separately.

```bash
# You (the instructor) set up once:
mkdir $WORK/class_data
chmod 755 $WORK/class_data

# Download the dataset
cd $WORK/class_data
wget https://example.com/large_dataset.csv

# Tell students:
# "The data is at /work/12345/your_username/system_name/class_data"
# "Copy it to your own $WORK or read from it directly."
```

### Collect Student Submissions

You want students to drop files into a shared folder.

```bash
# Create a submissions folder with write access
mkdir $WORK/submissions
chmod 777 $WORK/submissions

# Tell students:
# "Copy your file to /work/12345/your_username/system_name/submissions/"
```

Students submit with:

```bash
cp my_results.csv /work/12345/your_username/system_name/submissions/my_name_results.csv
```

> **Note:** With 777, any user on the system can write to this directory. Use 775 if you only want allocation members to write.

### Share Code with a Research Team

Everyone on the team needs to read and edit the same codebase.

```bash
mkdir $WORK/project
chmod 770 $WORK/project
cd $WORK/project
git clone https://github.com/your-repo.git
```

Team members access it at your full path. For a better workflow, have everyone clone the repo to their own `$WORK` and use Git for collaboration instead of shared folders.

### Pre-Stage a Database for a Class

You don't want 30 students each downloading 9 GB. Download once, share to all.

```bash
mkdir $WORK/shared_db
chmod 755 $WORK/shared_db
cd $WORK/shared_db

# Run the setup script once
bash /path/to/setup_data.sh

# Students read from your path:
# sqlite3 /work/12345/your_username/system_name/shared_db/nyc_taxi.db
```

---

## Setting Permissions on Files Inside the Folder

Creating a folder with `chmod 775` doesn't automatically apply those permissions to files inside it. New files inherit the creator's default permissions.

To set permissions on everything inside:

```bash
# Set folder permissions
chmod 775 $WORK/shared

# Set permissions on all existing files inside
chmod -R 775 $WORK/shared/*
```

To make new files automatically inherit group permissions, set the **setgid** bit:

```bash
chmod g+s $WORK/shared
```

This means any new file created inside `shared/` will belong to the same group as the folder, not the individual user's default group. This is important for team write access.

---

## Checking Permissions

```bash
# See permissions on a directory
ls -la $WORK/shared

# See who you are and what groups you're in
id

# See permissions explained
stat $WORK/shared
```

The output of `ls -la` looks like:

```
drwxrwxr-x  2 your_username G-12345  4096 Mar 24 10:00 shared
```

Reading left to right:
- `d` — it's a directory
- `rwx` — owner can read, write, execute
- `rwx` — group can read, write, execute
- `r-x` — others can read and execute (enter) but not write

---

## Which Filesystem to Use

| Filesystem | Good for Sharing? | Why |
|-----------|------------------|-----|
| `$WORK` | Best choice | Persistent, 1 TB, not purged |
| `$SCRATCH` | Temporary only | Files purged after 10 days of no access |
| `$HOME` | No | Too small (10 GB), meant for personal config |

Always use `$WORK` for shared directories.

---

## Troubleshooting

**"Permission denied" when a student tries to access**
- Check that the folder permissions allow group or others to read: `ls -la $WORK/shared`
- Make sure the student is on the same allocation: have them run `groups` and compare

**"No such file or directory"**
- The student is using the wrong path. Give them the output of `echo $WORK/shared` — the full absolute path, not a variable

**Files inside the folder aren't accessible even though the folder is**
- The files have different permissions than the folder. Run `chmod -R 755 $WORK/shared/*`

**Students can read but not write**
- The folder is 755 (read-only for group). Change to 775: `chmod 775 $WORK/shared`

**Students on a different allocation can't access**
- They're not in your Unix group. Either add them to your allocation, or use 755/777 permissions (opens to all users on the system)

---

## Quick Reference

| Task | Command |
|------|---------|
| Create shared folder | `mkdir $WORK/shared` |
| Read-only for everyone | `chmod 755 $WORK/shared` |
| Read/write for your group | `chmod 775 $WORK/shared` |
| Private to your group | `chmod 770 $WORK/shared` |
| Open to all (careful) | `chmod 777 $WORK/shared` |
| Apply to all files inside | `chmod -R 755 $WORK/shared/*` |
| Auto-inherit group on new files | `chmod g+s $WORK/shared` |
| Get your full path | `echo $WORK/shared` |
| Check permissions | `ls -la $WORK/shared` |
| Check your groups | `groups` |

---

## Related Guides

- [Getting Started with MSF](https://morehouse-supercomputing.github.io/mscf-getting-started/) — HPC account setup, SSH, first job
- [Jupyter on HPC](https://morehouse-supercomputing.github.io/jupyter-on-hpc/) — Running Jupyter notebooks on compute nodes
- [SQL on HPC](https://morehouse-supercomputing.github.io/sql-on-hpc/) — Querying large databases on HPC
