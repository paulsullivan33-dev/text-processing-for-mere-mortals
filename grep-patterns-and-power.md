# grep: Patterns and Power

Chapter 2's patterns were plain words. A
pattern can also *describe*: "a digit, then
another digit," "ERROR or WARN," "an address
beginning 192.168." The describing language
is **regular expressions** — regex — and it
is a subject of its own: the sibling guide
[*Regular Expressions for Mere
Mortals*](https://paulsullivan33-dev.github.io/regex-for-mere-mortals/)
teaches it properly. This chapter borrows
only the pieces grep work needs daily, and
spends its length on the grep-specific
machinery around them: which dialect you're
speaking, several patterns at once, whole
trees, and exit codes.

## Classes and repetition, briefly

Square brackets name a *set of acceptable
characters* for one position: `[0-9]` is
"any one digit." One — that word does more
damage than any other in pattern work:

```
$ grep 'usage at [0-9]%' app.log
```

Nothing. The log's figures are two digits
(`81%`), and `[0-9]` covers exactly one
character position, so the pattern demands a
digit immediately followed by `%`. Say it
twice and it matches:

```
$ grep 'usage at [0-9][0-9]%' app.log
2026-10-17 13:14:59 WARN disk usage at 81% on /data
2026-10-18 13:37:48 WARN disk usage at 84% on /data
2026-10-19 13:45:21 WARN disk usage at 79% on /data
```

A dot, remember, means *any character* —
which is why an address pattern wants its
dots escaped with backslashes, each `\.`
meaning one literal dot:

```
$ grep -c 'ip=192\.168\.1\.' app.log
14
```

(The count is 14 — nine lines from
192.168.1.10 and five from 192.168.1.25.
Unescaped, the same command also answers 14
on this file, because the addresses really
do contain dots. Chapter 2's nonsense
address, `192x168y1z10`, is what the escape
protects you from.)

## Two dialects: basic and extended

grep's default dialect is the **basic**
regular expression (BRE), a historical
dialect in which the powerful characters
`+`, `?`, `|`, `(`, `)` are *literal* unless
escaped with a backslash. The **extended**
dialect (`-E`, once a separate program
called `egrep`) makes them operators. The
two-digit problem above, in extended form,
uses `+` — "one or more of the thing
before":

```
$ grep -E 'usage at [0-9]+%' app.log
2026-10-17 13:14:59 WARN disk usage at 81% on /data
2026-10-18 13:37:48 WARN disk usage at 84% on /data
2026-10-19 13:45:21 WARN disk usage at 79% on /data
```

The same pattern in basic grep needs the
backslash spelling, `\+`:

```
$ grep 'usage at [0-9]\+%' app.log
2026-10-17 13:14:59 WARN disk usage at 81% on /data
2026-10-18 13:37:48 WARN disk usage at 84% on /data
2026-10-19 13:45:21 WARN disk usage at 79% on /data
```

Same answers, two spellings. Mortals
generally pick `-E` and stay there. With
`-E`, `|` means *or*, and parentheses group:

```
$ grep -Ec 'ERROR|WARN' app.log
24
$ grep -E 'user=(paul|katherine)' app.log
2026-10-17 07:15:22 INFO auth login ok user=paul ip=192.168.1.10
2026-10-17 10:22:08 INFO auth login ok user=katherine ip=192.168.1.25
```

(and eight more lines — every line naming
either user, logins and logouts alike).
That one pattern replaces two greps and a
merge, and it's the shape of most real-world
searching: *any of these words, in this
position*.

## Perl patterns, named honestly

GNU grep has a third dialect, `-P`, which
speaks the Perl dialect of regex: `\d` for a
digit, `{4}` for "exactly four," and more.
Two of its tricks earn a showing. `-o`
prints *only the matched part* of each line
instead of the whole line, and `\K` means
"the match starts keeping from here," which
strips a prefix in one motion:

```
$ grep -oP 'ip=\K[0-9.]+' app.log | head -4
192.168.1.10
192.168.1.10
198.51.100.23
198.51.100.23
```

```
$ grep -oP '\d{4}-\d{2}-\d{2}' app.log | head -3
2026-10-17
2026-10-17
2026-10-17
```

Now the honesty: `-P` is a GNU feature.
**macOS and the BSDs don't have it** — their
grep answers "invalid option" — and even
among GNU greps it can be missing from
minimal builds. Everything in this guide
except this section runs on any grep; treat
`-P` as a local convenience, not a portable
habit, and reach for it when you're working
interactively on a GNU system (chapter 8
extracts the same addresses with awk, the
portable way).

## Many patterns at once

`-e` may be repeated to give grep several
patterns — a line matching *any* of them
prints:

```
$ grep -e timeout -e disk-full app.log
2026-10-17 09:58:47 ERROR app job import-orders failed reason=timeout
2026-10-17 20:47:33 ERROR app job export-report failed reason=disk-full
2026-10-18 11:42:29 ERROR app job import-orders failed reason=timeout
```

And `-f` reads the patterns from a file,
one per line — the form that scales to a
watchlist you maintain:

```
$ cat patterns.txt
locked
heartbeat
$ grep -f patterns.txt app.log
2026-10-17 22:03:19 INFO app heartbeat ok ip=10.0.0.5
2026-10-18 08:14:55 ERROR auth account locked user=admin ip=198.51.100.23
2026-10-18 22:03:19 INFO app heartbeat ok ip=10.0.0.5
2026-10-19 22:03:19 INFO app heartbeat ok ip=10.0.0.5
```

(`patterns.txt` is yours to make — the lab
doesn't ship it, because making it *is* the
lesson.)

## Searching a tree

`-r` (recursive) searches a directory and
everything under it, prefixing each match
with its path. With a small specimen tree —
`site/a.log` is a copy of the lab log, and
`site/readme.txt` is a note saying all's
quiet:

```
$ grep -r 'account locked' site
site/a.log:2026-10-18 08:14:55 ERROR auth account locked user=admin ip=198.51.100.23
```

```
$ grep -rl 'ERROR' site
site/a.log
```

The readme doesn't appear: it says
"errors," lowercase, and grep is
case-sensitive unless `-i` says otherwise.
`--include` narrows the search to files
matching a name pattern:

```
$ grep -rl --include='*.txt' 'errors' site
site/readme.txt
```

Recursive search over a big tree is the one
grep in this guide that can take a while or
say a lot; `-l` and `-c` are how you keep
it civilised — survey first, read second.

## Exit codes: grep's second answer

Every grep gives two answers: the lines it
prints, and an **exit code** — a number the
shell keeps in `$?` — reporting how it went.
grep's are:

| Code | Meaning |
|---|---|
| 0 | Lines were selected (at least one match) |
| 1 | No lines selected |
| 2 | An error occurred (a file missing, unreadable, …) |

All three, captured:

```
$ grep ERROR app.log > /dev/null; echo $?
0
$ grep CRITICAL app.log; echo "grep exit was $?"
grep exit was 1
$ grep ERROR missing.log; echo "grep exit was $?"
grep: missing.log: No such file or directory
grep exit was 2
```

The distinctions matter because **1 is not
an error** — "nobody knocked today" is a
finding, not a failure — while 2 means the
question itself went wrong. Scripts lean on
this constantly, usually with `-q` (quiet:
print nothing, the exit code is the whole
answer):

```
$ grep -q ERROR app.log; echo $?
0
$ grep -q CRITICAL app.log; echo $?
1
```

`if grep -q 'failed' app.log; then …; fi`
is the shape. Chapter 10 returns to these
codes, because "grep printed nothing and I
don't know why" is the most common text
trouble there is — and the answer is almost
always sitting in `$?`.

Next: sed, which doesn't just find lines —
it changes them.
