# sed: Doing More

One substitution is a party trick. This
chapter makes sed a tool: several commands
in one pass, scripts saved to files,
replacements that reuse what the pattern
found, and — with its guardrails stated
first — editing the file itself.

## Several commands at once

Repeat `-e` to queue more than one edit.
sed applies them in order, line by line,
each command seeing the result of the one
before:

```
$ sed -n '1p;5p' app.log | sed -e 's/INFO/I/' -e 's/WARN/W/'
2026-10-17 06:00:01 I cron job nightly-backup started
2026-10-17 08:41:03 W auth login failed user=admin ip=198.51.100.23
```

When the edits grow past two or three, they
belong in a **script file** — one command
per line, run with `-f`. A file that
normalises the log's auth vocabulary:

```
$ cat fix.sed
s/login failed/LOGIN FAILED/
s/login ok/LOGIN OK/
s/logout/LOGOUT/
$ sed -f fix.sed app.log | sed -n '3p;15p'
2026-10-17 07:15:22 INFO auth LOGIN OK user=paul ip=192.168.1.10
2026-10-17 16:05:26 INFO auth LOGOUT user=paul ip=192.168.1.10
$ sed -f fix.sed app.log | grep -c LOGIN
18
```

18 lines carry a normalised marker (11
failed logins + 7 good ones). A sed script
is a document: it can be kept, reviewed,
and re-run next month — which the command
line history is not. (`fix.sed` is yours to
make, like chapter 3's `patterns.txt`.)
Two commands may also be separated by a
semicolon inside one set of quotes —
`s/a/b/; s/c/d/` — which the recipes
chapter uses; `-e` and `-f` are the forms
that stay readable.

## Remembering what matched: groups and &

A substitution can reuse pieces of what it
matched. Wrap part of the pattern in
`\(` … `\)` — a **capture group** — and the
replacement can refer to it as `\1` (the
first group), `\2`, and so on. Pull the
username out of the log's key–value tail
and bracket it:

```
$ grep 'login ok' app.log | sed 's/user=\([a-z]*\)/[\1]/'
2026-10-17 07:15:22 INFO auth login ok [paul] ip=192.168.1.10
2026-10-17 10:22:08 INFO auth login ok [katherine] ip=192.168.1.25
```

(and five more). The group captured `paul`
or `katherine`, and the replacement rebuilt
that stretch of line around it — `user=`
and the value are gone, the brackets are
new, `\1` is the rescued text. Two groups
can even change places, which is how you
reformat a date stamp without a program:

```
$ sed 's/^\([0-9-]*\) \([0-9:]*\)/\2 \1/' app.log | head -3
06:00:01 2026-10-17 INFO cron job nightly-backup started
06:04:37 2026-10-17 INFO cron job nightly-backup finished status=ok
07:15:22 2026-10-17 INFO auth login ok user=paul ip=192.168.1.10
```

Group 1 is the date, group 2 is the time,
and `\2 \1` prints them swapped. The
backslashes are the basic dialect again —
sed's default, like grep's. GNU sed's `-E`
switches patterns to the extended dialect,
where groups are plain parentheses:

```
$ grep heartbeat app.log | sed -E 's/ip=([0-9.]+)/address=\1/'
2026-10-17 22:03:19 INFO app heartbeat ok address=10.0.0.5
2026-10-18 22:03:19 INFO app heartbeat ok address=10.0.0.5
2026-10-19 22:03:19 INFO app heartbeat ok address=10.0.0.5
```

One more replacement tool needs no groups
at all: `&` in a replacement stands for
**the whole match**. Marking every error
line is one short command:

```
$ grep ERROR app.log | sed 's/ERROR/[&]/'
2026-10-17 09:58:47 [ERROR] app job import-orders failed reason=timeout
2026-10-17 15:44:10 [ERROR] http POST /orders 500 ip=203.0.113.7
```

(and eight more). `&` is also a trap when
you *don't* mean it — chapter 10 shows a
replacement going wrong by exactly one
ampersand, and its escape, `\&`.

## Editing the file: -i and the backup habit

Everything so far printed a stream. `-i`
(*in-place*) tells sed to write the result
back over the input file. It is the most
useful option in this guide and the only
one that can destroy data, so it gets a
worked example with the safety habit built
in: **give `-i` a suffix**. GNU sed takes
the suffix attached, `-i.bak`, and keeps
the pre-edit file under that name:

```
$ cp messy.txt scratch.txt
$ sed -i.bak 's/ *$//' scratch.txt
$ ls scratch.txt*
scratch.txt
scratch.txt.bak
```

The command trims trailing spaces — ` *$`
is "any run of spaces at the end of the
line," a tidy-up worth the whole chapter —
and the proof that it worked, and *only*
that, is one `diff` away:

```
$ diff scratch.txt.bak scratch.txt
2c2
< Apples 
---
> Apples
6c6
< carrots 
---
> carrots
10c10
< elderberries 
---
> elderberries
13c13
< grapes  
---
> grapes
```

Four lines changed, each by exactly the
trailing spaces (`diff` exits 1 when files
differ — that is its way of saying "found
some," not an error). `cat -A`, which shows
line-ends as `$`, confirms the spaces are
gone:

```
$ cat -A scratch.txt | head -4
apples$
Apples$
 APPLES$
bananas$
```

The habit to keep: while an edit is new,
always use the suffix, check with `diff`,
and delete the `.bak` when you're satisfied.
Two honest warnings. First, `sed -i` works
by writing a fresh file and renaming it
over the original — permissions and
ownership can change, and a second hard
link to the old file keeps the *old* text.
Second, the spelling is a dialect trap:
**BSD sed (macOS) requires the suffix
argument**, so the same command there is
`sed -i '' 's/…/…/' file` for no backup, and
GNU's attached `-i.bak` reads differently
to it. That difference is stated here from
the tools' documentation, not from a
capture — `criteria.md` records it as
documented-only, as it does for the other
BSD spellings in this guide.

## Named, not taught: the hold space

sed carries a second storage area, the
**hold space**, with commands (`h`, `H`,
`g`, `G`, `x`) for parking a line and
bringing it back later — enough machinery
to join, reorder, and accumulate lines
across a file. It is real, it is used, and
it is deliberately out of this guide's
scope: hold-space programs are the least
readable text processing there is, and
nearly every mortal job that reaches for it
is clearer in awk, which holds whole tables
in arrays (next chapter but one). Knowing
the name exists — and that a sed incantation
you meet in the wild with `h` and `x` in it
is doing cross-line state — is the mortal
share of it.

Next: awk, where lines become fields and
fields become arithmetic.
