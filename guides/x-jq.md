# Guide to jq

A high-level overview of `jq`, a lightweight command-line JSON processor.

## What is jq?

`jq` is a command-line tool for parsing, filtering, and transforming JSON data.
It works like `sed` or `awk` but specifically for JSON. You pipe JSON into it,
apply a filter expression, and get formatted output.

## Installation

```bash
# macOS
brew install jq

# Ubuntu/Debian
sudo apt install jq

# Windows (scoop)
scoop install jq
```

## Basic Usage

### Pretty-print JSON

```bash
echo '{"name":"Alice","age":30}' | jq '.'
```

Output:
```json
{
  "name": "Alice",
  "age": 30
}
```

### Extract a field

```bash
echo '{"name":"Alice","age":30}' | jq '.name'
# "Alice"
```

### Raw output (no quotes)

```bash
echo '{"name":"Alice"}' | jq -r '.name'
# Alice
```

## Working with Arrays

### Access elements

```bash
echo '[1,2,3]' | jq '.[0]'
# 1
```

### Iterate over elements

```bash
echo '[1,2,3]' | jq '.[]'
# 1
# 2
# 3
```

### Get array length

```bash
echo '[1,2,3]' | jq 'length'
# 3
```

## Filtering and Selecting

### Select from array of objects

```bash
echo '[{"name":"Alice","age":30},{"name":"Bob","age":25}]' | jq '.[] | select(.age > 27)'
# {"name":"Alice","age":30}
```

### Map over elements

```bash
echo '[1,2,3]' | jq '[.[] | . * 2]'
# [2,4,6]
```

## Constructing New Objects

```bash
echo '{"first":"Alice","last":"Smith","age":30}' | jq '{full_name: (.first + " " + .last), age}'
# {"full_name":"Alice Smith","age":30}
```

## Pipes and Chaining

`jq` expressions can be chained with `|`, similar to shell pipes:

```bash
echo '{"users":[{"name":"Alice","role":"admin"},{"name":"Bob","role":"user"}]}' | \
  jq '.users[] | select(.role == "admin") | .name'
# "Alice"
```

## Common Patterns

### Read from a file

```bash
jq '.key' data.json
```

### Combine with curl

```bash
curl -s https://api.example.com/data | jq '.results[]'
```

### Compact output

```bash
echo '{"a": 1}' | jq -c '.'
# {"a":1}
```

### Keys of an object

```bash
echo '{"a":1,"b":2}' | jq 'keys'
# ["a","b"]
```

### Check if a key exists

```bash
echo '{"a":1}' | jq 'has("a")'
# true
```

## Summary

| Feature       | Syntax                        |
|---------------|-------------------------------|
| Identity      | `.`                           |
| Field access  | `.field`                      |
| Array index   | `.[N]`                        |
| Iterate       | `.[]`                         |
| Pipe           | `\|`                         |
| Select        | `select(condition)`           |
| Map           | `[.[] \| expr]`              |
| Raw strings   | `-r` flag                     |
| Compact       | `-c` flag                     |
