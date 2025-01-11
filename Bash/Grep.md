
# Delete lines in a file containing a string

```bash
grep -vi "pattern" file.json > file2.json

# Inline replace
grep -vi "pattern" file1.json > tmp && mv tmp file1.json
```

# Find pattern between two things

```bash
# find all microsoft.com_playwright.* between > and </a>
grep -oP '(?<=>)(microsoft.com_playwright)(.*)(?=</a>)' file.json
```

# Return only the matching portion

```bash
grep -o "pattern" file.json
```

# Exclude

```bash
grep -v "pattern" file.json
```