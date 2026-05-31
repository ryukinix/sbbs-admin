# SchemeBBS Administration Tool (`sbbs-admin.ros`)

`sbbs-admin.ros` is a standalone Common Lisp CLI script powered by [Roswell](https://github.com/roswell/roswell) to manage [SchemeBBS](https://github.com/alyssa-p-hacker/SchemeBBS) instances. It operates directly on the underlying S-Expression (`sexp`) database, bypassing the need for a running Scheme environment and naturally handling Scheme's idiosyncratic data formats.

## Features

- **No external dependencies:** Pure Common Lisp via Roswell.
- **Robust UTF-8 and S-Expression Parsing:** Capable of reading MIT Scheme's octal escape sequences (e.g., `\237\215` for emojis like 🍻) and parsing `#f`/`#t` boolean values correctly.
- **Index Generation:** Completely reconstructs the board's `list` and `index` files based on raw post data, fixing any desync issues.
- **Content Moderation:** Safely delete entire threads or specific comments within a thread, automatically regenerating the index and clearing HTML caches.
- **Backup & Restore:** Easily backup the entire `sexp` database to a tarball and restore it when needed.
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

### 1. `generate-index`
Regenerates the `list` and `index` files for a specific board by scanning all available threads. It accurately sorts threads by the date of their latest post and calculates the correct message counts. It also removes the cached HTML index files to ensure the BBS serves the updated data.

**Usage:**
```sh
sbbs-admin.ros generate-index <board>
```
**Example:**
```sh
sbbs-admin.ros generate-index prog
```

### 2. `remove-post`
Deletes an entire thread (post) and its associated HTML cache, then automatically regenerates the board's index.

**Usage:**
```sh
sbbs-admin.ros remove-post <board> <post-id>
```
**Example:**
```sh
sbbs-admin.ros remove-post prog 42
```

### 3. `remove-comment`
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

### 4. `backup`
Creates a compressed tarball (`.tar.gz`) backup of the entire `sexp` directory.

**Usage:**
```sh
sbbs-admin.ros backup <archive-name.tar.gz>
```
**Example:**
```sh
sbbs-admin.ros backup my-board-backup.tar.gz
```

### 5. `restore`
Restores the `sexp` directory from a previously created backup tarball.

**Usage:**
```sh
sbbs-admin.ros restore <archive-name.tar.gz>
```
**Example:**
```sh
sbbs-admin.ros restore my-board-backup.tar.gz
```
