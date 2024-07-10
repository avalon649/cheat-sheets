Regex Accelerated Course and Cheat Sheet
----------------------------------------

For easy navigation, here are some jumping points to various sections of the page:  
  
✽ [Characters](#chars)  
✽ [Quantifiers](#quantifiers)  
✽ [More Characters](#morechars)  
✽ [Logic](#logic)  
✽ [More White-Space](#whitespace)  
✽ [More Quantifiers](#morequants)  
✽ [Character Classes](#classes)  
✽ [Anchors and Boundaries](#anchors)  
✽ [POSIX Classes](#posix)  
✽ [Inline Modifiers](#modifiers)  
✽ [Lookarounds](#lookarounds)  
✽ [Character Class Operations](#classoperations)  
✽ [Other Syntax](#other)  
  
  
[(direct link)](#chars)  

Characters
----------

| Character | Legend | Example | Sample Match |
| --- | --- | --- | --- |
| `\\d` | `Most engines: one digit` | `from 0 to 9` | `file\_\\d\\d` |
| `\\d` | `.NET, Python 3: one Unicode digit in any script` | `file\_\\d\\d` | `file_9੩` |
| `\\w` | `Most engines: "word character": ASCII letter, digit or underscore` | `\\w-\\w\\w\\w` | `A-b\_1` |
| `\\w` | `.Python 3: "word character": Unicode letter, ideogram, digit, or underscore` | `\\w-\\w\\w\\w` | `字-ま\_۳` |
| `\\w` | `.NET: "word character": Unicode letter, ideogram, digit, or connector` | `\\w-\\w\\w\\w` | `字-ま‿۳`|
| `\\s` | `Most engines: "whitespace character": space, tab, newline, carriage return, vertical tab` | `a\\sb\\sc` | `a b` | `c` |
| `\\s` | `.NET, Python 3, JavaScript: "whitespace character": any Unicode separator` | `a\\sb\\sc` | `a b` | `c` |
| `\\D` | `One character that is not a _digit_ as defined by your engine's` | `_\\d_` | `\\D\\D\\D` | `ABC` |
| `\\W` | `One character that is not a _word character_ as defined by your engine's` | `_\\w_` | `\\W\\W\\W\\W\\W` | `\*-+=)` |
| `\\S` | `One character that is not a _whitespace character_ as defined by your engine's` | `_\\s_` | `\\S\\S\\S\\S` | `Yoyo` |  

Quantifiers
-----------

| Character | Legend | Example | Sample Match |
| --- | --- | --- | --- |
| `+` | `One or more Version` | `\\w-\\w+` | `Version A-b1\_1` |
| `{3}` | `Exactly three times` | `\\D{3}` | `ABC` |
| `{2,4}` | `Two to four times` | `\\d{2,4}` | `156` |
| `{3,}` | `Three or more times` | `\\w{3,}` | `regex\_tutorial` |
| `\*` | `Zero or more times` | `A\*B\*C\*` | `AAACC` |
| `?` | `Once or none` | `plurals?` | `plural` |

More Characters
---------------

| Character | Legend | Example | Sample Match |
| --- | --- | --- | --- |
| `.` | `any character except line break` | `a.c` | `abc` |
| `.` | `Any character except line break` | `.*` | `whatever, man.` |
| `\.` | `A period A period (special character: needs to be escaped by a \)` | `a\.c` | `a.c` |
| `\` |`Escapes a special character`	| `\.\*\+\?    \$\^\/\\` | `.*+? $^/\` |
| `\` | `Escapes a special character` | `\[\{\(\)\}\]` | `[{()}]` |

Logic
-----

| Character | Legend | Example | Sample Match |
| --- | --- | --- | --- |
| ``|`` | ``Alternation / OR operand 22|33`` | `33` | 
| `( … )` |	``Capturing group | A(nt|pple)`` | `Apple (captures "pple")` |
| `\1`| `Contents of Group 1` | `r(\w)g\1x` | `regex` |
| `\2` | `Contents of Group 2`| `(\d\d)\+(\d\d)=\2\+\1` | `12+65=65+12` |
| `(?: … )` | `Non-capturing group` | `A(?:nt|pple)` | `Apple` |
 
More White-Space
----------------

Character

Legend

Example

Sample Match

\\t

Tab

T\\t\\w{2}

T     ab

\\r

Carriage return character

see below

\\n

Line feed character

see below

\\r\\n

Line separator on Windows

AB\\r\\nCD

AB  
CD

\\N

Perl, PCRE (C, PHP, R…): one character that is not a line break

\\N+

ABC

\\h

Perl, PCRE (C, PHP, R…), Java: one horizontal whitespace character: tab or Unicode space separator

\\H

One character that is not a horizontal whitespace

\\v

.NET, JavaScript, Python, Ruby: vertical tab

\\v

Perl, PCRE (C, PHP, R…), Java: one vertical whitespace character: line feed, carriage return, vertical tab, form feed, paragraph or line separator

\\V

Perl, PCRE (C, PHP, R…), Java: any character that is not a vertical whitespace

\\R

Perl, PCRE (C, PHP, R…), Java: one line break (carriage return + line feed pair, and all the characters matched by \\v)

  
  
[(direct link)](#morequants)  

More Quantifiers
----------------

Quantifier

Legend

Example

Sample Match

+

The + (one or more) is "greedy"

\\d+

12345

?

Makes quantifiers "lazy"

\\d+?

1 in **1**2345

\*

The \* (zero or more) is "greedy"

A\*

AAA

?

Makes quantifiers "lazy"

A\*?

empty in AAA

{2,4}

Two to four times, "greedy"

\\w{2,4}

abcd

?

Makes quantifiers "lazy"

\\w{2,4}?

ab in **ab**cd

  
  
[(direct link)](#classes)  

Character Classes
-----------------

Character

Legend

Example

Sample Match

\[ … \]

One of the characters in the brackets

\[AEIOU\]

One uppercase vowel

\[ … \]

One of the characters in the brackets

T\[ao\]p

_Tap_ or _Top_

\-

Range indicator

\[a-z\]

One lowercase letter

\[x-y\]

One of the characters in the range from x to y

\[A-Z\]+

GREAT

\[ … \]

One of the characters in the brackets

\[AB1-5w-z\]

One of either: A,B,1,2,3,4,5,w,x,y,z

\[x-y\]

One of the characters in the range from x to y

\[ -~\]+

Characters in the printable section of the [ASCII table](http://www.asciitable.com/).

\[^x\]

One character that is not x

\[^a-z\]{3}

A1!

\[^x-y\]

One of the characters **not** in the range from x to y

\[^ -~\]+

Characters that are **not** in the printable section of the [ASCII table](http://www.asciitable.com/).

\[\\d\\D\]

One character that is a digit or a non-digit

\[\\d\\D\]+

Any characters, inc-  
luding new lines, which the regular dot doesn't match

\[\\x41\]

Matches the character at hexadecimal position 41 in the ASCII table, i.e. A

\[\\x41-\\x45\]{3}

ABE

  
  
[(direct link)](#anchors)  

[Anchors](regex-anchors.html) and [Boundaries](regex-boundaries.html)
---------------------------------------------------------------------

Anchor

Legend

Example

Sample Match

^

[Start of string](regex-anchors.html#caret) or [start of line](regex-anchors.html#carmulti) depending on multiline mode. (But when \[^inside brackets\], it means "not")

^abc .\*

abc (line start)

$

[End of string](regex-anchors.html#dollar) or [end of line](regex-anchors.html#eol) depending on multiline mode. Many engine-dependent subtleties.

.\*? the end$

this is the end

\\A

[Beginning of string](regex-anchors.html#A)  
(all major engines except JS)

\\Aabc\[\\d\\D\]\*

abc (string...  
...start)

\\z

[Very end of the string](regex-anchors.html#z)  
Not available in Python and JS

the end\\z

this is...\\n...**the end**

\\Z

[End of string](regex-anchors.html#Z) or (except Python) before final line break  
Not available in JS

the end\\Z

this is...\\n...**the end**\\n

\\G

[Beginning of String or End of Previous Match](regex-anchors.html#G)  
.NET, Java, PCRE (C, PHP, R…), Perl, Ruby

\\b

[Word boundary](regex-boundaries.html#wordboundary)  
Most engines: position where one side only is an ASCII letter, digit or underscore

Bob.\*\\bcat\\b

Bob ate the cat

\\b

[Word boundary](regex-boundaries.html#wordboundary)  
.NET, Java, Python 3, Ruby: position where one side only is a Unicode letter, digit or underscore

Bob.\*\\b\\кошка\\b

Bob ate the кошка

\\B

[Not a word boundary](regex-boundaries.html#notb)

c.\*\\Bcat\\B.\*

copycats

  
  
[(direct link)](#posix)  

POSIX Classes
-------------

Character

Legend

Example

Sample Match

\[:alpha:\]

PCRE (C, PHP, R…): ASCII letters A-Z and a-z

\[8\[:alpha:\]\]+

WellDone88

\[:alpha:\]

Ruby 2: Unicode letter or ideogram

\[\[:alpha:\]\\d\]+

кошка99

\[:alnum:\]

PCRE (C, PHP, R…): ASCII digits and letters A-Z and a-z

\[\[:alnum:\]\]{10}

ABCDE12345

\[:alnum:\]

Ruby 2: Unicode digit, letter or ideogram

\[\[:alnum:\]\]{10}

кошка90210

\[:punct:\]

PCRE (C, PHP, R…): ASCII punctuation mark

\[\[:punct:\]\]+

?!.,:;

\[:punct:\]

Ruby: Unicode punctuation mark

\[\[:punct:\]\]+

‽,:〽⁆

  
  
[(direct link)](#modifiers)  

[Inline Modifiers](regex-modifiers.html)
----------------------------------------

None of these are supported in JavaScript. In Ruby, beware of (?s) and (?m).  

Modifier

Legend

Example

Sample Match

(?i)

[Case-insensitive mode](regex-modifiers.html#i)  
(except JavaScript)

(?i)Monday

monDAY

(?s)

[DOTALL mode](regex-modifiers.html#dotall) (except JS and Ruby). The dot (.) matches new line characters (\\r\\n). Also known as "single-line mode" because the dot treats the entire input as a single line

(?s)From A.\*to Z

From A  
to Z

(?m)

[Multiline mode](regex-modifiers.html#multiline)  
(except Ruby and JS) ^ and $ match at the beginning and end of every line

(?m)1\\r\\n^2$\\r\\n^3$

1  
2  
3

(?m)

[In Ruby](regex-modifiers.html#rubym): the same as (?s) in other engines, i.e. DOTALL mode, i.e. dot matches line breaks

(?m)From A.\*to Z

From A  
to Z

(?x)

[Free-Spacing Mode mode](regex-modifiers.html#freespacing)  
(except JavaScript). Also known as comment mode or whitespace mode

(?x) # this is a  
\# comment  
abc # write on multiple  
\# lines  
\[ \]d # spaces must be  
\# in brackets

abc d

(?n)

[.NET, PCRE 10.30+: named capture only](regex-modifiers.html#n)

Turns all (parentheses) into non-capture groups. To capture, use [named groups](regex-capture.html#namedgroups).

(?d)

[Java: Unix linebreaks only](regex-modifiers.html#d)

The dot and the ^ and $ anchors are only affected by \\n

(?^)

[PCRE 10.32+: unset modifiers](regex-disambiguation.html#unset-all)

Unsets ismnx modifiers

  
  
[(direct link)](#lookarounds)  

[Lookarounds](regex-lookarounds.html)
-------------------------------------

Lookaround

Legend

Example

Sample Match

(?=…)

[Positive lookahead](regex-disambiguation.html#lookahead)

(?=\\d{10})\\d{5}

01234 in **01234**56789

(?<=…)

[Positive lookbehind](regex-disambiguation.html#lookbehind)

(?<=\\d)cat

cat in 1**cat**

(?!…)

[Negative lookahead](regex-disambiguation.html#negative-lookahead)

(?!theatre)the\\w+

theme

(?<!…)

[Negative lookbehind](regex-disambiguation.html#negative-lookbehind)

\\w{3}(?<!mon)ster

Munster

  
  
[(direct link)](#classoperations)  

[Character Class Operations](regex-class-operations.html)
---------------------------------------------------------

Class Operation

Legend

Example

Sample Match

\[…-\[…\]\]

.NET: character class subtraction. One character that is in those on the left, but not in the subtracted class.

\[a-z-\[aeiou\]\]

Any lowercase consonant

\[…-\[…\]\]

.NET: character class subtraction.

\[\\p{IsArabic}-\[\\D\]\]

An Arabic character that is not a non-digit, i.e., an Arabic digit

\[…&&\[…\]\]

Java, Ruby 2+: character class intersection. One character that is both in those on the left and in the && class.

\[\\S&&\[\\D\]\]

An non-whitespace character that is a non-digit.

\[…&&\[…\]\]

Java, Ruby 2+: character class intersection.

\[\\S&&\[\\D\]&&\[^a-zA-Z\]\]

An non-whitespace character that a non-digit and not a letter.

\[…&&\[^…\]\]

Java, Ruby 2+: character class subtraction is obtained by intersecting a class with a negated class

\[a-z&&\[^aeiou\]\]

An English lowercase letter that is not a vowel.

\[…&&\[^…\]\]

Java, Ruby 2+: character class subtraction

\[\\p{InArabic}&&\[^\\p{L}\\p{N}\]\]

An Arabic character that is not a letter or a number

  
  
[(direct link)](#other)  

Other Syntax
------------

Syntax

Legend

Example

Sample Match

\\K

[Keep Out](regex-best-trick.html#bsk)  
Perl, PCRE (C, PHP, R…), Python's alternate [_regex_](https://pypi.python.org/pypi/regex) engine, Ruby 2+: drop everything that was matched so far from the overall match to be returned

prefix\\K\\d+

12

\\Q…\\E

Perl, PCRE (C, PHP, R…), Java: treat anything between the delimiters as a literal string. Useful to escape metacharacters.

\\Q(C++ ?)\\E

(C++ ?)

  
  
  
  

Don't Miss The [**Regex Style Guide**](regex-style.html)  
  
and [**The Best Regex Trick Ever!!!**](regex-best-trick.html)  

 [![next](https://d1go27vtttaqyn.cloudfront.net/next_regex.png)  
 **The 1001 ways to use Regex**](regex-uses.html)  
  
  

[![Regex Rex](https://d1go27vtttaqyn.cloudfront.net/rightgraphic_rexegg3.png)  
**Ask Rex**](regex-consultant.html)  
  

[![Buy me a coffee](https://d1go27vtttaqyn.cloudfront.net/buy_me_a_coffee-250.gif)](https://www.buymeacoffee.com/lutfen)  

[![Buy me a coffee](https://d1go27vtttaqyn.cloudfront.net/buy_me_a_coffee-250.gif)](https://www.buymeacoffee.com/lutfen)  

  

[×](javascript:void(0)) **Fundamentals**  

*   [Regex Tutorial](.)
*   [Regex vs. Regex](regex-vs-regular-expression.html)
*   [Quick Reference](regex-quickstart.html)
*   [100 Uses for Regex](regex-uses.html)
*   [Regex Style Guide](regex-style.html)

  
**Black Belt Program**  

*   [All (? … ) Syntax](regex-disambiguation.html)
*   [Boundaries++](regex-boundaries.html)
*   [Anchors](regex-anchors.html)
*   [Capture & Back](regex-capture.html)
*   [Flags & Modifiers](regex-modifiers.html)
*   [Lookarounds](regex-lookarounds.html)
*   [Quantifiers](regex-quantifiers.html)
*   [Explosive Quantifiers](regex-explosive-quantifiers.html)
*   [Conditionals](regex-conditionals.html)
*   [Recursion](regex-recursion.html)
*   [Class Operations](regex-class-operations.html)
*   [Backtracking Control](backtracking-control-verbs.html)
*   [Regex _Gotchas_](regex-gotchas.html)
*   [Syntax Tricks](regex-tricks.html)
*   [PCRE Callouts](pcre-callouts.html)
*   [Quantifier capture](regex-quantifier-capture.html)

  
**Regex in Action**  

For awesome tricks:  
scroll down!

*   [Cookbook](regex-cookbook.html)
*   [Cool Regex Classes](regex-interesting-character-classes.html)
*   [Regex Optimizations](regex-optimizations.html)
*   [PCRE: Grep and Test](pcregrep-pcretest.html)
*   [Perl One-Liners](regex-perl-one-liners.html)
*   [Amazing Shortcuts](regex-firefox-shortcuts.html)

  
**Tools & More**  

*   [Regex Tools](regex-tools.html)
*   [RegexBuddy](regexbuddy-tutorial.html)
*   [Regex Humor](regex-humor.html)
*   [Regex Books & Links](regex-books.html)

  
**Tricks**  

*   [The Best Regex Trick](regex-best-trick.html)
*   [Conditional Sub](regex-trick-conditional-replacement.html)
*   [Line Numbers](regex-trick-line-numbers.html)
*   [Numbers in English](regex-trick-numbers-in-english.html)

  
**Languages**  

*   [PCRE Doc & Log](pcre-documentation.html)
*   [Regex with Perl](regex-perl.html)
*   [Regex with C#](regex-csharp.html)
*   [Regex with PHP](regex-php.html)
*   [Regex with Python](regex-python.html)
*   [Regex with Java](regex-java.html)
*   [Regex with JavaScript](regex-javascript.html)
*   [Regex with Ruby](regex-ruby.html)
*   [Regex with VB.NET](regex-vbnet.html)

  

[![Buy me a coffee](https://d1go27vtttaqyn.cloudfront.net/buy_me_a_coffee150.gif)](https://www.buymeacoffee.com/lutfen)

 

  

  
**© Copyright RexEgg.com**