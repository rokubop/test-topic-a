# Guide to rg (ripgrep)

A high-level overview of `rg`, a fast recursive search tool.

## What is ripgrep?

`rg` (ripgrep) is a line-oriented search tool that recursively searches directories
for a regex pattern. It is similar to `grep -r` but significantly faster, respects
`.gitignore` by default, and has sensible defaults for searching codebases.

## Installation

```bash
# macOS
brew install ripgrep

# Ubuntu/Debian
sudo apt install ripgrep

# Windows (scoop)
scoop install ripgrep
```

## Basic Usage

### Search for a pattern in current directory

```bash
rg "TODO"
```

This recursively searches all files for "TODO" and prints matching lines with
file paths and line numbers.

### Search a specific file

```bash
rg "pattern" path/to/file.txt
```

### Search a specific directory

```bash
rg "pattern" src/
```

## Common Flags

### Case-insensitive search

```bash
rg -i "error"
```

### Show context lines

```bash
# 2 lines before and after each match
rg -C 2 "pattern"

# 3 lines before
rg -B 3 "pattern"

# 3 lines after
rg -A 3 "pattern"
```

### List only file names

```bash
rg -l "pattern"
```

### Count matches per file

```bash
rg -c "pattern"
```

### Show line numbers (on by default in terminals)

```bash
rg -n "pattern"
```

## Filtering Files

### Search specific file types

```bash
rg -t py "import"       # Python files
rg -t js "require"      # JavaScript files
rg -t rust "fn main"    # Rust files
```

### List available types

```bash
rg --type-list
```

### Glob patterns

```bash
rg -g "*.md" "TODO"           # Only markdown files
rg -g "!*.min.js" "function"  # Exclude minified JS
```

### Include hidden files

```bash
rg --hidden "pattern"
```

### Don't respect .gitignore

```bash
rg --no-ignore "pattern"
```

## Regex Support

`rg` uses Rust's regex engine, which supports most common patterns:

```bash
rg "fn \w+\("              # Function definitions
rg "^\s*import"             # Lines starting with import
rg "TODO|FIXME|HACK"       # Multiple patterns
rg "v\d+\.\d+\.\d+"       # Version strings like v1.2.3
```

## Replacement

### Preview replacements (does not modify files)

```bash
rg "old_name" -r "new_name"
```

## Advanced Usage

### Search only specific lines

```bash
rg --max-count 1 "pattern"   # Only first match per file
```

### Fixed strings (no regex)

```bash
rg -F "array[0]"             # Treat pattern as literal
```

### Multiline matching

```bash
rg -U "struct \w+ \{"
```

### JSON output

```bash
rg --json "pattern"
```

## Comparison with grep

| Feature              | rg             | grep -r         |
|----------------------|----------------|-----------------|
| Speed                | Very fast      | Slower          |
| .gitignore support   | Yes (default)  | No              |
| Unicode              | Yes            | Varies          |
| Binary file skip     | Yes (default)  | No              |
| Colored output       | Yes (default)  | With `--color`  |
| Regex engine         | Rust regex     | POSIX/PCRE      |

## Summary

| Task                  | Command                      |
|-----------------------|------------------------------|
| Basic search          | `rg "pattern"`               |
| Case insensitive      | `rg -i "pattern"`            |
| File names only       | `rg -l "pattern"`            |
| Specific file type    | `rg -t py "pattern"`         |
| With context          | `rg -C 3 "pattern"`         |
| Fixed string          | `rg -F "literal"`            |
| Glob filter           | `rg -g "*.rs" "pattern"`    |
| Count matches         | `rg -c "pattern"`            |
