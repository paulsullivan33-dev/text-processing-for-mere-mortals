# grep: Finding Lines

**grep** prints the lines of a file that
match a pattern. That is the whole job, and
this chapter is the everyday form of it:
plain word searches, the handful of options
that shape the answer, and the two or three
surprises that bite every beginner once.
Chapter 3 adds pattern power; everything
here is literal text and options.

The shape is `grep [options] pattern file`.
The pattern in this chapter is just words:
grep looks for the pattern *anywhere in the
line*, and the line is the unit — a match
anywhere prints the whole line:

```
$ grep ERROR app.log
2026-10-17 09:58:47 ERROR app job import-orders failed reason=timeout
2026-10-17 15:44:10 ERROR http POST /orders 500 ip=203.0.113.7
2026-10-17 20:47:33 ERROR app job export-report failed reason=disk-full
2026-10-18 06:31:52 ERROR cron job nightly-backup failed reason=rsync-exit-12
2026-10-18 08:14:55 ERROR auth account locked user=admin ip=198.51.100.23
2026-10-18 11:42:29 ERROR app job import-orders failed reason=timeout
2026-10-18 15:51:03 ERROR app job billing-run failed reason=db-unreachable
2026-10-18 17:29:40 ERROR disk write failed on /data reason=io-error
2026-10-19 09:31:27 ERROR app job import-orders failed reason=bad-row-441
2026-10-19 14:26:07 ERROR http POST /orders 500 ip=192.168.1.25
```

Quote any pattern that contains spaces or
shell characters — `"login failed"` below is
one pattern, not two:

```
$ grep -n "login failed" app.log
5:2026-10-17 08:41:03 WARN auth login failed user=admin ip=198.51.100.23
6:2026-10-17 08:41:09 WARN auth login failed user=admin ip=198.51.100.23
7:2026-10-17 08:41:31 WARN auth login failed user=root ip=198.51.100.23
16:2026-10-17 18:20:41 WARN auth login failed user=guest ip=198.51.100.23
23:2026-10-18 08:12:44 WARN auth login failed user=admin ip=198.51.100.23
```

(and six more after those). The `-n` in
that command prefixes each line with its
line number — the difference between "it
happened" and "it's line 23, look there."
Two more shapers earn their keep early.
`-c` counts instead of printing, for when
the question is *how many*:

```
$ grep -c ERROR app.log
10
$ grep -c "login ok" app.log
7
```

And `-i` ignores case, because the file's
spelling and yours will eventually disagree:

```
$ grep -i 'Login Failed' app.log | head -3
2026-10-17 08:41:03 WARN auth login failed user=admin ip=198.51.100.23
2026-10-17 08:41:09 WARN auth login failed user=admin ip=198.51.100.23
2026-10-17 08:41:31 WARN auth login failed user=root ip=198.51.100.23
```

## Inverting: everything except

`-v` inverts the match — print the lines
that do *not* contain the pattern. On the
account file, "which accounts are *not*
marked nologin" is a security question in
one line:

```
$ grep -v nologin passwd.txt
root:x:0:0:root:/root:/bin/bash
sync:x:4:65534:sync:/bin:/bin/sync
paul:x:1000:1000:Paul Sullivan:/home/paul:/bin/bash
katherine:x:1001:1001:Katherine Sullivan:/home/katherine:/bin/zsh
guest:x:1002:1002:Guest Account:/home/guest:/bin/sh
backupsvc:x:1003:1003:Backup Service:/srv/backup:/bin/false
```

Read that output like an auditor, not a
fan: `sync` and `backupsvc` end in other
shells (`/bin/sync`, `/bin/false`) — they
can't start an interactive session either,
but grep doesn't know that. grep answers
the question you asked, literally. Asking
the question precisely is most of the craft
in this guide.

## Context: the lines around

A match often needs its neighbours — what
happened just before the failure? `-A n`
adds n lines *after* each match, `-B n`
n lines *before*, and `-C n` both. The log's
one account lockout, with its surroundings:

```
$ grep -B 1 "account locked" app.log
2026-10-18 08:13:19 WARN auth login failed user=oracle ip=198.51.100.23
2026-10-18 08:14:55 ERROR auth account locked user=admin ip=198.51.100.23
```

```
$ grep -A 1 "account locked" app.log
2026-10-18 08:14:55 ERROR auth account locked user=admin ip=198.51.100.23
2026-10-18 09:26:37 INFO http GET /guide/openssl.html 200 ip=203.0.113.7
```

Before: another failed login from the same
address. After: ordinary traffic resumes.
Context is how a log line becomes a story.
(When matches sit close together, GNU grep
separates the groups with a `--` line.)

## Anchors: beginning and end of line

Two pattern characters are positions, not
letters: `^` means *start of the line* and
`$` means *end of the line*. They turn
"contains" into "starts with" and "ends
with":

```
$ grep -c '^2026-10-19' app.log
17
```

```
$ grep 'status=ok$' app.log
2026-10-17 06:04:37 INFO cron job nightly-backup finished status=ok
2026-10-17 11:30:00 INFO cron job cert-check finished status=ok
2026-10-18 12:15:00 INFO cron job cert-check finished status=ok
2026-10-19 06:05:13 INFO cron job nightly-backup finished status=ok
2026-10-19 12:15:00 INFO cron job cert-check finished status=ok
```

```
$ grep '^root' passwd.txt
root:x:0:0:root:/root:/bin/bash
$ grep 'bash$' passwd.txt
root:x:0:0:root:/root:/bin/bash
paul:x:1000:1000:Paul Sullivan:/home/paul:/bin/bash
```

Note the quotes in `'bash$'`: the `$` is
for grep, and single quotes keep the shell
from spending it first. That reflex — *the
pattern goes in single quotes* — prevents
half the surprises in chapter 10.

## Whole words, fixed strings

Remember grep matches *anywhere* in the
line — including inside other words. The
account file demonstrates why that matters:

```
$ grep login passwd.txt
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
games:x:5:60:games:/usr/games:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
oldacct:x:1004:1004:Old Account:/home/oldacct:/usr/sbin/nologin
```

Six lines, all false friends: every match
is the *login* inside *nologin*. `-w`
requires the match to be a whole word —
bounded by non-letter characters or the
line's ends:

```
$ grep -w login passwd.txt
```

and prints nothing at all, because the file
contains no standalone word *login*. Empty
output is an answer too: it means "no line
matched," and grep reports it with an exit
code rather than a message — chapter 3 ends
with those codes, because scripts live on
them.

Finally, `-F` treats the pattern as a
*fixed string*: plain text, no pattern
characters at all. It matters because some
characters in ordinary text — `.` above
all — are pattern characters in disguise.
A dot in a pattern matches *any* character:

```
$ echo 'id 192x168y1z10 here' | grep '192.168.1.10'
id 192x168y1z10 here
$ echo 'id 192x168y1z10 here' | grep -F '192.168.1.10'
```

The first command matches a nonsense
address, its dots happily standing in for
`x`, `y`, and `z`; the second, with `-F`,
correctly finds nothing. Searching for IP
addresses, file names, and version numbers
— text paved with dots — is `-F`'s home
ground:

```
$ grep -F "db.example.test" app.conf
db_host = db.example.test
```

## Several files at once

Give grep more than one file and it tells
you which file each line came from:

```
$ grep -c paul app.log passwd.txt orders.csv
app.log:6
passwd.txt:1
orders.csv:5
```

And when the question is only *which* files
contain a match, `-l` lists names instead
of lines:

```
$ grep -l shipped orders.csv app.log passwd.txt
orders.csv
```

That pair — search widely with `-l`, then
look closely at the survivors — is how you
triage a directory you don't know yet.
Chapter 3's recursive search puts it to
work on whole trees.

Next: patterns with real machinery —
classes, repetition, alternation — and the
exit codes that make grep scriptable.
