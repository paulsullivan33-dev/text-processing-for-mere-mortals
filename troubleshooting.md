# Troubleshooting

Text tools fail quietly more often than
loudly: the commonest symptom is *nothing
printed, no error shown*, and the cause is
almost never the one you first suspect.
Five classic troubles, each reproduced
against the lab, in the order a mortal
meets them.

## Nothing printed, and no error

**Symptom:** a grep prints nothing and the
prompt just returns.

**First check:** the exit code, immediately
— `$?` holds only the *last* command's
result, so ask before running anything
else:

```
$ grep CRITICAL app.log; echo "exit=$?"
exit=1
```

Exit 1 means the search ran fine and found
nothing: no line contains CRITICAL (the lab
log's worst level is ERROR). That is an
answer, not a malfunction. Exit 2 is the
malfunction — the question itself failed:

```
$ grep ERROR nope.log; echo "exit=$?"
grep: nope.log: No such file or directory
exit=2
```

If the code is 1 and you expected matches,
the pattern and the file disagree about
spelling: case (`Error` versus `ERROR` —
try `-i`), an anchor asking for a position
the text doesn't have (next section), or a
pattern character meaning more than you
intended (chapter 2's dots). Shrink the
pattern until something matches, then build
it back up.

## The file is "binary"

**Symptom:** grep answers with a refusal
instead of lines:

```
$ grep ERROR bin.dat; echo "exit=$?"
grep: bin.dat: binary file matches
exit=0
```

(`bin.dat` is a lab-made file containing
zero bytes: `printf 'alpha\0beta ERROR
gamma\0\n'`.) grep found the match, decided
the file isn't text — NUL bytes are its
evidence — and summarised instead of
printing bytes that would scramble your
terminal. The exit code is 0: it *did*
match. Older and BSD greps phrase the same
message as "Binary file bin.dat matches."
`-c` still counts matching lines (here, 1),
and `-a` treats the file as text and prints:

```
$ grep -a ERROR bin.dat | cat -v
alpha^@beta ERROR gamma^@
```

(shown through `cat -v`, which renders each
NUL as `^@` so this page stays readable).
Real-world version of this trap: a log file
that picked up stray binary bytes stops
answering plain grep — `-a` is the fix, and
the stray bytes are a separate mystery
worth its own minute.

## Anchors that won't match: Windows line endings

**Symptom:** `grep 'two$'` finds nothing in
a file where `two` is plainly the last word
on its line.

**Cause:** the file has Windows (CRLF) line
endings. Every line secretly ends in a
carriage return — an invisible `\r` — so
the line's actual end is *after* the `\r`,
and `two` is not at the end of anything:

```
$ printf 'one\r\ntwo\r\n' > crlf.txt
$ cat -A crlf.txt
one^M$
two^M$
$ grep 'two$' crlf.txt; echo "exit=$?"
exit=1
$ grep two crlf.txt
two
```

`cat -A` is the diagnostic: `^M` before
each `$` is the carriage return, exposed.
Unanchored `two` matches fine, which is how
you know the file is healthy and only the
line endings are foreign. The fix, when the
file is yours to normalise, is a one-line
sed that deletes the `\r` at each line end:

```
$ sed 's/\r$//' crlf.txt | grep 'two$'
two
```

(The `\r` in the file-making `printf` and
in sed's pattern is the carriage return
written as an escape; files that arrive from
Windows editors and some Windows tools are
where you'll meet it.)

## awk prints empty columns

**Symptom:** awk prints the right number of
lines, but the field you asked for is
blank:

```
$ awk '{print $2}' orders.csv | head -3



```

**Cause:** the separator. With no `-F`,
awk splits on whitespace — and a CSV line
has no spaces, so the *whole line* is `$1`
and `$2` simply doesn't exist. Empty output
from awk almost always means the split
isn't happening where you think. Name the
separator and the same program works:

```
$ awk -F, '{print $2}' orders.csv | head -3
date
2026-10-15
2026-10-15
```

Relatives of this trouble: fields numbered
from 1, not 0 (`$1` is the first); a header
row counted as data (skip it with `NR>1`);
and string fields compared as text when you
meant numbers — `"10" < "9"` is true for
strings, false for numbers, and awk chooses
by the look of the values.

## The shell ate my $1

**Symptom:** an awk program in double
quotes prints whole lines when you asked
for one field.

```
$ awk "{print $1}" app.log | head -3
2026-10-17 06:00:01 INFO cron job nightly-backup started
2026-10-17 06:04:37 INFO cron job nightly-backup finished status=ok
2026-10-17 07:15:22 INFO auth login ok user=paul ip=192.168.1.10
```

**Cause:** double quotes let the *shell*
expand `$1` before awk ever sees it — `$1`
is the shell's first argument, usually
empty — so awk received the program
`{print }`, which prints the whole record.
The fix is the standing rule from chapter
2: **program text goes in single quotes**,
which the shell passes through untouched:

```
$ awk '{print $1}' app.log | head -3
2026-10-17
2026-10-17
2026-10-17
```

The same rule protects sed scripts and grep
patterns — anywhere a `$` is meant for the
tool, not the shell.

## sed replaced it strangely: the & trap

**Symptom:** a replacement contains the
thing you replaced, embedded like a typo:
you wrote `&amp;` and got `andamp;`.

```
$ printf 'salt and pepper\n' | sed 's/and/&amp;/'
salt andamp; pepper
```

**Cause:** in a replacement, `&` means
*the whole match* (chapter 5) — sed put the
matched word `and` where the `&` stood,
producing `and` + `amp;`. To write a
literal ampersand, escape it:

```
$ printf 'salt and pepper\n' | sed 's/and/\&/'
salt & pepper
```

Its mirror image is chapter 4's quiet
surprise: without the `g` flag, only the
first match *on each line* is replaced.
Between the two, most "sed did something
weird" reports are accounted for.

Next: the glossary — every term the guide
used, in one place.
