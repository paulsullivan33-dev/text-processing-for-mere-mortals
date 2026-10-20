# Glossary

**Anchor** — a pattern element that matches
a *position* rather than characters: `^`
(start of line) and `$` (end of line).
Chapter 2.

**Associative array** — awk's table type:
an array indexed by strings, used to keep a
count or total per category:
`count[$3]++`. Chapter 7.

**Back-reference** — a reuse of text a
pattern captured: `\1` in a sed replacement
stands for the first capture group.
Chapter 5.

**Basic regular expression (BRE)** — the
default pattern dialect of grep and sed,
in which `+ ? | ( )` are literal unless
escaped with a backslash. The extended
dialect (ERE, grep's `-E`, sed's `-E`) makes
them operators. Chapter 3.

**Capture group** — a part of a pattern
wrapped so its matched text is remembered:
`\(` … `\)` in the basic dialect, `( … )`
in the extended. Chapter 5.

**Delimiter** — the character separating
the parts of a sed substitution. A slash by
habit, but whatever character follows the
`s` — `s|a|b|` uses bars, which keeps paths
readable. Chapter 4.

**Exit code** — the number a command leaves
in `$?` reporting how it went. grep's are
0 (lines selected), 1 (none selected), 2
(an error). Chapter 3.

**Field** — one piece of a record after
splitting: `$1`, `$2`, … in awk. `$0` is the
whole record and `$NF` the last field.
Chapter 6.

**Field separator (FS)** — the character or
pattern awk splits records on: whitespace
by default, set with `-F` or in `BEGIN`.
`OFS` is its output twin, used when
printing. Chapters 6–7.

**Fixed string** — a pattern treated as
plain text with no pattern characters at
all: grep's `-F`. The right tool for text
paved with dots. Chapter 2.

**Global flag** — the `g` after a sed
substitution, meaning replace *every* match
on the line instead of the first.
Chapter 4.

**Hold space** — sed's spare storage for
parking a line and retrieving it later.
Named in chapter 5, deliberately not
taught: the programs that need it read
better in awk.

**In-place editing** — sed's `-i`, which
writes the edited stream back over the
input file. Travels with a backup suffix
(`-i.bak`) in this guide. Chapter 5.

**Line** — the unit grep and sed work in:
characters up to a newline. A file whose
lines end in an invisible carriage return
as well (Windows endings) is chapter 10's
trap.

**Pattern** — a description of text to
match: a plain word, or a regular
expression. Also, in awk, the condition
half of a pattern–action rule.

**Pipeline** — commands joined by `|`, each
stage's output feeding the next stage's
input. Chapter 8.

**Record** — awk's unit of input: one line,
in everything this guide does. `NR` counts
records overall, `FNR` within the current
file, `FILENAME` names it. Chapters 6–7.

**Regular expression** — the pattern
language: classes like `[0-9]`, repetition,
alternation, anchors. Borrowed lightly in
chapter 3; taught fully by the sibling
guide *Regular Expressions for Mere
Mortals*.

**Stream editor** — sed's full name and job
description: an editor that works on a
stream of lines as they pass, printing the
result. Chapter 4.

**Whole-word match** — grep's `-w`: the
match must be bounded by non-word
characters, so *login* doesn't match inside
*nologin*. Chapter 2.
