# Linux Regular Expressions Cheat Sheet

## Regex Flavors on Linux

| Tool       | Flavor           | Notes                              |
|------------|------------------|------------------------------------|
| `grep`     | BRE (default)    | Use `-E` for ERE, `-P` for PCRE   |
| `egrep`    | ERE              | Same as `grep -E`                  |
| `sed`      | BRE (default)    | Use `-E` for ERE                   |
| `awk`      | ERE              |                                    |
| `perl`     | PCRE             | Most feature-rich                  |
| `python`   | PCRE-like        | `re` module                        |

---

## Anchors

| Pattern | Matches                              |
|---------|--------------------------------------|
| `^`     | Start of line                        |
| `$`     | End of line                          |
| `\b`    | Word boundary (PCRE/ERE)             |
| `\B`    | Non-word boundary                    |
| `\<`    | Start of word (BRE/ERE in GNU)       |
| `\>`    | End of word (BRE/ERE in GNU)         |

```bash
grep '^hello'    file.txt   # lines starting with "hello"
grep 'world$'    file.txt   # lines ending with "world"
grep '\bword\b'  file.txt   # exact word match (grep -P or -E)
```

---

## Character Classes

| Pattern    | Matches                                      |
|------------|----------------------------------------------|
| `.`        | Any single character (except newline)        |
| `[abc]`    | Any one of: a, b, or c                       |
| `[^abc]`   | Any character NOT a, b, or c                 |
| `[a-z]`    | Any lowercase letter                         |
| `[A-Z]`    | Any uppercase letter                         |
| `[0-9]`    | Any digit                                    |
| `[a-zA-Z]` | Any letter                                   |
| `[a-zA-Z0-9]` | Any alphanumeric character              |

### POSIX Character Classes (usable inside `[...]`)

| Class        | Equivalent     | Matches                       |
|--------------|----------------|-------------------------------|
| `[:alpha:]`  | `[a-zA-Z]`     | Letters                       |
| `[:digit:]`  | `[0-9]`        | Digits                        |
| `[:alnum:]`  | `[a-zA-Z0-9]`  | Letters and digits            |
| `[:space:]`  | `[ \t\n\r\f]`  | Whitespace characters         |
| `[:blank:]`  | `[ \t]`        | Space and tab                 |
| `[:lower:]`  | `[a-z]`        | Lowercase letters             |
| `[:upper:]`  | `[A-Z]`        | Uppercase letters             |
| `[:punct:]`  |                | Punctuation characters        |
| `[:print:]`  |                | Printable characters          |
| `[:graph:]`  |                | Printable non-space chars     |
| `[:cntrl:]`  |                | Control characters            |
| `[:xdigit:]` | `[0-9a-fA-F]`  | Hexadecimal digits            |

```bash
grep '[[:upper:]][[:lower:]]*'  file.txt   # capitalized word
grep '[[:digit:]]\{3\}'         file.txt   # 3 consecutive digits (BRE)
```

---

## Shorthand Character Classes (PCRE / grep -P)

| Pattern | Matches                           |
|---------|-----------------------------------|
| `\d`    | Digit `[0-9]`                     |
| `\D`    | Non-digit `[^0-9]`                |
| `\w`    | Word char `[a-zA-Z0-9_]`         |
| `\W`    | Non-word char                     |
| `\s`    | Whitespace `[ \t\n\r\f\v]`       |
| `\S`    | Non-whitespace                    |
| `\h`    | Horizontal whitespace             |
| `\H`    | Non-horizontal whitespace         |

```bash
grep -P '\d{4}-\d{2}-\d{2}' file.txt   # ISO date (PCRE)
```

---

## Quantifiers

### Basic (BRE — backslash required for `\{` `\}` `\+` `\?`)

| Pattern   | BRE       | ERE/PCRE  | Matches                     |
|-----------|-----------|-----------|-----------------------------|
| zero or one  | `\?`   | `?`       | 0 or 1 occurrence           |
| zero or more | `*`    | `*`       | 0 or more occurrences       |
| one or more  | `\+`   | `+`       | 1 or more occurrences       |
| exactly n    | `\{n\}`| `{n}`     | Exactly n occurrences       |
| n or more    | `\{n,\}`| `{n,}`   | n or more occurrences       |
| between n–m  | `\{n,m\}`| `{n,m}` | Between n and m occurrences |

```bash
grep -E 'ab+'        file.txt   # "a" followed by 1+ "b"s
grep -E 'colou?r'    file.txt   # "color" or "colour"
grep -E '[0-9]{2,4}' file.txt   # 2 to 4 digit number
```

### Lazy (Non-Greedy) Quantifiers — PCRE only (`grep -P`)

| Pattern | Matches                                   |
|---------|-------------------------------------------|
| `*?`    | 0 or more, as few as possible             |
| `+?`    | 1 or more, as few as possible             |
| `??`    | 0 or 1, as few as possible                |
| `{n,m}?`| Between n–m, as few as possible          |

```bash
grep -P '<.+?>'  file.txt   # shortest HTML tag match
```

---

## Groups and Alternation

| Pattern      | Flavor       | Description                          |
|--------------|--------------|--------------------------------------|
| `\(...\)`    | BRE          | Capturing group                      |
| `(...)`      | ERE/PCRE     | Capturing group                      |
| `(?:...)`    | PCRE         | Non-capturing group                  |
| `(?P<name>...)`| PCRE       | Named capturing group                |
| `\|`         | BRE (GNU)    | Alternation (OR)                     |
| `\|`         | ERE/PCRE     | Alternation (OR) — no backslash      |
| `\1` `\2`    | BRE/ERE/PCRE | Backreference to group 1, 2, …       |

```bash
grep -E '(cat|dog)'          file.txt   # "cat" or "dog"
grep -E '(ha)\1'             file.txt   # "haha" (backreference)
grep -P '(?P<year>\d{4})-(?P<month>\d{2})' file.txt
sed -E 's/(foo)(bar)/\2\1/'  file.txt   # swap "foo" and "bar"
```

---

## Lookahead and Lookbehind — PCRE only (`grep -P`)

| Pattern      | Type                  | Description                              |
|--------------|-----------------------|------------------------------------------|
| `(?=...)`    | Positive lookahead    | Followed by ...                          |
| `(?!...)`    | Negative lookahead    | NOT followed by ...                      |
| `(?<=...)`   | Positive lookbehind   | Preceded by ...                          |
| `(?<!...)`   | Negative lookbehind   | NOT preceded by ...                      |

```bash
grep -P 'foo(?=bar)'    file.txt   # "foo" only when followed by "bar"
grep -P 'foo(?!bar)'    file.txt   # "foo" NOT followed by "bar"
grep -P '(?<=\$)\d+'    file.txt   # digits preceded by "$"
grep -P '(?<!\d)\d{3}'  file.txt   # 3-digit number not preceded by digit
```

---

## Escaping Special Characters

Special characters that must be escaped with `\` when used literally:

```
. * + ? ^ $ { } [ ] ( ) | \
```

```bash
grep '\.'      file.txt   # literal dot
grep '\$'      file.txt   # literal dollar sign
grep -E '\(.*\)' file.txt  # literal parentheses with anything inside
```

---

## Common grep Options

| Flag        | Description                                      |
|-------------|--------------------------------------------------|
| `-E`        | Extended regex (ERE)                             |
| `-P`        | Perl-compatible regex (PCRE)                     |
| `-F`        | Fixed strings (no regex)                         |
| `-i`        | Case-insensitive match                           |
| `-v`        | Invert match (non-matching lines)                |
| `-n`        | Print line numbers                               |
| `-c`        | Count matching lines                             |
| `-l`        | List filenames with matches                      |
| `-L`        | List filenames WITHOUT matches                   |
| `-o`        | Print only the matched part                      |
| `-r` / `-R` | Recursive search                                 |
| `-w`        | Match whole words only                           |
| `-x`        | Match whole lines only                           |
| `-A n`      | Print n lines after match                        |
| `-B n`      | Print n lines before match                       |
| `-C n`      | Print n lines before and after match             |
| `--color`   | Highlight matches                                |

---

## sed — Stream Editor

```bash
# Basic substitution (first occurrence per line)
sed 's/old/new/'         file.txt

# Global substitution (all occurrences)
sed 's/old/new/g'        file.txt

# Case-insensitive substitution (GNU sed)
sed 's/old/new/gi'       file.txt

# Delete lines matching pattern
sed '/pattern/d'         file.txt

# Print only matching lines
sed -n '/pattern/p'      file.txt

# Substitute on lines matching a pattern
sed '/pattern/s/old/new/g'  file.txt

# Edit file in-place
sed -i 's/old/new/g'     file.txt

# Use ERE with -E
sed -E 's/(foo|bar)/baz/g'  file.txt

# Capture groups — swap first two words
sed -E 's/(\w+) (\w+)/\2 \1/' file.txt

# Delete empty lines
sed '/^[[:space:]]*$/d'  file.txt

# Print lines 5–10
sed -n '5,10p'           file.txt
```

---

## awk — Pattern Scanning

```bash
# Print lines matching pattern
awk '/pattern/'           file.txt

# Print specific field from matching lines
awk '/pattern/ { print $2 }'  file.txt

# Field separator (CSV-like)
awk -F',' '/pattern/ { print $1 }'  file.csv

# Negate match
awk '!/pattern/'          file.txt

# Regex on specific field
awk '$3 ~ /pattern/'      file.txt
awk '$3 !~ /pattern/'     file.txt

# BEGIN/END blocks
awk 'BEGIN { count=0 } /pattern/ { count++ } END { print count }' file.txt
```

---

## Practical Examples

### Validate formats
```bash
# IPv4 address (basic)
grep -E '^([0-9]{1,3}\.){3}[0-9]{1,3}$' file.txt

# Email address (simplified)
grep -E '^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$' file.txt

# URL (simplified)
grep -E '^https?://[a-zA-Z0-9.-]+(/[^[:space:]]*)?$' file.txt

# ISO date YYYY-MM-DD
grep -E '^[0-9]{4}-(0[1-9]|1[0-2])-(0[1-9]|[12][0-9]|3[01])$' file.txt

# MAC address
grep -E '^([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}$' file.txt
```

### Extract data
```bash
# Extract all IPs from a log
grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' access.log

# Extract all emails
grep -oE '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' file.txt

# Extract content inside quotes
grep -oP '(?<=")[^"]*(?=")' file.txt

# Extract HTTP status codes from nginx log
grep -oP '"(GET|POST|PUT|DELETE)[^"]*" \K\d{3}' access.log
```

### File operations
```bash
# Find files containing pattern recursively
grep -rl 'pattern' /path/to/dir

# Replace string in all .conf files
sed -i 's/old_value/new_value/g' /etc/*.conf

# Remove commented lines and blank lines
grep -Ev '^\s*(#|$)' config.conf

# Count occurrences of a word
grep -o 'word' file.txt | wc -l

# Find lines with 2+ consecutive spaces
grep -P '  +' file.txt
```

---

## BRE vs ERE Quick Reference

| Feature         | BRE             | ERE          |
|-----------------|-----------------|--------------|
| Grouping        | `\(...\)`       | `(...)`      |
| Alternation     | `\|`            | `\|`         |
| One or more     | `\+`            | `+`          |
| Zero or one     | `\?`            | `?`          |
| Repetition      | `\{n,m\}`       | `{n,m}`      |
| Backreference   | `\1`            | `\1`         |

---

## Special Sequences Summary (PCRE)

| Sequence | Meaning                               |
|----------|---------------------------------------|
| `\n`     | Newline                               |
| `\t`     | Tab                                   |
| `\r`     | Carriage return                       |
| `\f`     | Form feed                             |
| `\v`     | Vertical tab                          |
| `\0`     | Null character                        |
| `\xHH`   | Hex character (e.g. `\x41` = "A")    |
| `\uHHHH` | Unicode code point                    |

---

## Quick Reference Card

```
ANCHORS        ^  $  \b  \B  \<  \>
ANY CHAR       .
QUANTIFIERS    *  +  ?  {n}  {n,}  {n,m}
LAZY           *?  +?  ??  {n,m}?          (PCRE)
CLASSES        [abc]  [^abc]  [a-z]
POSIX          [:alpha:]  [:digit:]  [:space:]  etc.
SHORTHANDS     \d  \D  \w  \W  \s  \S     (PCRE)
GROUPS         (...)  (?:...)  (?P<name>...)
ALTERNATION    a|b
BACKREFS       \1  \2  ...
LOOKAHEAD      (?=...)  (?!...)            (PCRE)
LOOKBEHIND     (?<=...)  (?<!...)          (PCRE)
ESCAPE         \  before special chars
```
