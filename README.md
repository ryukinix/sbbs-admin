# SchemeBBS Administration Tool (`sbbs-admin.ros`)

`sbbs-admin.ros` is a standalone Common Lisp CLI script powered by [Roswell](https://github.com/roswell/roswell) to manage [SchemeBBS](https://github.com/alyssa-p-hacker/SchemeBBS) instances. It operates directly on the underlying S-Expression (`sexp`) database, bypassing the need for a running Scheme environment and naturally handling Scheme's idiosyncratic data formats.

## Features

- **No external dependencies:** Pure Common Lisp via Roswell.
- **Robust UTF-8 and S-Expression Parsing:** Capable of reading MIT Scheme's octal escape sequences (e.g., `\237\215` for emojis like 🍻) and parsing `#f`/`#t` boolean values correctly.
- **Index Generation:** Completely reconstructs the board's `list` and `index` files based on raw post data, fixing any desync issues.
- **Content Moderation:** Safely delete entire threads, specific comments, or edit comment content using your favorite `$EDITOR`. Automatically removes sequential duplicate comments across all boards. Regenerates indices and clears HTML caches.
- **Thread Management:** Move threads between different boards seamlessly.
- **Backup & Restore:** Easily backup the entire `sexp` directory to a tarball with optional descriptions and restore them with ease.
- **Timezone Adjustment:** Bulk update the timestamps of all posts across all boards.
- **Environment Driven:** Uses `SBBS_DATADIR` to locate the data directory.

## Installation

Ensure you have [Roswell](https://github.com/roswell/roswell) installed on your system.

Make the script executable:
```sh
chmod +x sbbs-admin.ros
```

## Configuration

The script relies on the `SBBS_DATADIR` environment variable to locate the root of your SchemeBBS data directory (which contains the `sexp/` and `html/` subdirectories).

- **Default:** If `SBBS_DATADIR` is not set, it defaults to `~/bbs`.
- **Custom Path:** You can pass it inline:
  ```sh
  SBBS_DATADIR=/path/to/schemebbs/data ./sbbs-admin.ros <command>
  ```

*Note: If your SchemeBBS data files are owned by `root`, you will need to run the script with `sudo`.*

## Commands

### 1. `list`
A versatile command to explore the BBS data at different levels:
- **No arguments:** Lists all available boards.
- **`<board>`:** Lists all threads in the specified board, showing their ID, last update date, and headline (sorted by latest update).
- **`<board> <post-id>`:** Lists all comments in a specific thread, including metadata (date, author) and content.

**Usage:**
```sh
# List all boards
sbbs-admin.ros list

# List threads in the 'prog' board
sbbs-admin.ros list prog

# List comments in thread #42 on the 'prog' board
sbbs-admin.ros list prog 42
```

### 2. `generate-index`
Regenerates the `list` and `index` files for a specific board by scanning all available threads. It accurately sorts threads by the date of their latest post and calculates the correct message counts. It also removes the cached HTML index files to ensure the BBS serves the updated data.

**Usage:**
```sh
sbbs-admin.ros generate-index <board>
```
**Example:**
```sh
sbbs-admin.ros generate-index prog
```

### 3. `remove-post`
Deletes an entire thread (post) and its associated HTML cache, then automatically regenerates the board's index.

**Usage:**
```sh
sbbs-admin.ros remove-post <board> <post-id>
```
**Example:**
```sh
sbbs-admin.ros remove-post prog 42
```

### 4. `remove-comment`
Deletes a specific comment from within a thread. It rewrites the thread's S-Expression file without the specified comment, deletes the thread's HTML cache, and regenerates the board's index to reflect the updated message count.

**Usage:**
```sh
sbbs-admin.ros remove-comment <board> <post-id> <comment-id>
```
**Example:**
```sh
# Removes comment #5 from thread #42 on the 'prog' board
sbbs-admin.ros remove-comment prog 42 5
```

### 5. `remove-duplicates`
Scans all boards and threads for sequential comments that have identical content. It displays the found duplicates and prompts for confirmation before deleting the second comment in each duplicate pair. A backup is automatically created before any deletions occur.

**Usage:**
```sh
sbbs-admin.ros remove-duplicates
```

### 6. `edit`
Opens a specific comment's content in your system's `$EDITOR` (defaults to `vi`). After saving and exiting the editor, the thread's S-Expression file is updated, a backup is created, and the board's index is regenerated.

**Usage:**
```sh
sbbs-admin.ros edit <board> <post-id> <comment-id>
```
**Example:**
```sh
sbbs-admin.ros edit prog 42 5
```

### 7. `move`
Moves an entire thread from a source board to a target board. It assigns a new post ID in the target board, removes the old files, and regenerates indices for both boards.

**Usage:**
```sh
sbbs-admin.ros move <source-board> <post-id> <target-board>
```
**Example:**
```sh
# Moves thread #42 from 'tmp' board to 'prog'
sbbs-admin.ros move tmp 42 prog
```

### 8. `backup`
Creates a compressed tarball (`.tar.gz`) backup of the entire `sexp` directory. You can optionally provide a description message which will be stored alongside the backup, or specify a custom filename.

**Usage:**
```sh
# Create a backup with a description
sbbs-admin.ros backup "Maintenance before update"

# Create a backup with a specific filename
sbbs-admin.ros backup manual-backup-2023.tar.gz
```

### 9. `restore`
Restores the `sexp` directory from a backup tarball. If no argument is provided, it lists all available backups in the `backup/` directory.

**Usage:**
```sh
# List all available backups
sbbs-admin.ros restore

# Restore from a specific backup file
sbbs-admin.ros restore sbbs-2023-06-01.tar.gz
```

### 10. `set-timezone`
Adjusts the timestamps of ALL posts across ALL boards by a specified number of hours. This is useful if your system clock was misconfigured or you're migrating between servers with different timezone settings. It automatically regenerates all board indices after the update.

**Usage:**
```sh
# Move all timestamps forward by 2 hours
sbbs-admin.ros set-timezone 2

# Move all timestamps back by 3 hours
sbbs-admin.ros set-timezone -3
```
