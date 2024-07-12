### Regex Cheat Sheet

**Basic Characters:**

* `.`  (Period): Matches any single character.
* `*`  (Asterisk): Matches zero or more occurrences of the preceding character.
* `+`  (Plus): Matches one or more occurrences of the preceding character.
* `?`  (Question Mark): Matches zero or one occurrence of the preceding character.
* `[]` (Square Brackets): Defines a character set. For example, `[abc]` matches any "a", "b", or "c".
* `[^]` (Caret inside Square Brackets): Defines a negated character set. For example, `[^abc]` matches any character *except* "a", "b", or "c".
* `\d` (Digit): Matches any digit (0-9).
* `\D` (Not a Digit): Matches any non-digit character.
* `\w` (Word): Matches any word character (a-z, A-Z, 0-9, _).
* `\W` (Not a Word): Matches any non-word character.
* `\s` (Whitespace): Matches any whitespace character (space, tab, newline).
* `\S` (Not Whitespace): Matches any non-whitespace character.

**Anchors:**

* `^` (Caret): Matches the beginning of a line.
* `$` (Dollar Sign): Matches the end of a line.

**Quantifiers:**

* `{n}`: Matches exactly *n* occurrences of the preceding character or group.
* `{n,}`: Matches at least *n* occurrences of the preceding character or group.
* `{n,m}`: Matches between *n* and *m* occurrences of the preceding character or group.

**Grouping and Alternation:**

* `()` (Parentheses): Groups characters together.
* `|` (Pipe): Matches either the expression before or after it.

**Example in Markdown:**

```
* To match any word character, use `\w`.
* To match zero or more spaces, use `\s*`.
* To match the beginning of a line followed by "hello", use `^hello`.
```



Let me know if you want to explore specific regex patterns or have any other questions!