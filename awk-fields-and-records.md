# awk: Fields and Records

**awk** is the third shape of text
processing: not selecting lines (grep) or
rewriting them (sed), but *taking them
apart*. awk reads a file one **record** at
a time — a record is a line, in everything
this guide does — and splits it into
**fields**. You write rules in the form
*pattern* `{` *action* `}`: when a record
matches the pattern, run the action. Fields
are named by position: `$1` is the first,
`$2` the second, and `$0` is the whole
record, unsplit.

The log, asked for fields 1 and 3 — the
date and the level:

```
$ awk '{print $1, $3}' app.log | head -4
2026-10-17 INFO
2026-10-17 INFO
2026-10-17 INFO
2026-10-17 INFO
```

That program has no pattern, so its action
runs on every record. `print $1, $3` prints
the two fields separated by a space — the
comma in a print statement means "join with
the output separator," which is a space
unless you change it (chapter 7 does).
There is no semicolon and no ceremony. Most
useful awk programs are one line long, and
they are built from these few pieces.

## Choosing the split: FS

By default awk splits on *whitespace* —
any run of spaces or tabs counts as one
separator, and leading whitespace is
ignored. The messy list shows what that
means: its ragged lines all yield the same
first field, cleanly:

```
$ awk '{print $1}' messy.txt
apples
Apples
APPLES
bananas
bananas
carrots
Carrots
dates

elderberries
ELDERBERRIES
figs
grapes
grapes
honeydew
```

(Even the blank line behaves — it has no
fields, so it prints an empty line.) But
passwd lines and CSV rows aren't
whitespace-split; they're split on a
character. `-F` names the **field
separator**:

```
$ awk -F: '{print $1, $7}' passwd.txt
root /bin/bash
daemon /usr/sbin/nologin
bin /usr/sbin/nologin
sys /usr/sbin/nologin
sync /bin/sync
games /usr/sbin/nologin
nobody /usr/sbin/nologin
paul /bin/bash
katherine /bin/zsh
guest /bin/sh
backupsvc /bin/false
oldacct /usr/sbin/nologin
```

Username and shell — a table the file was
always hiding, in one line of awk. The
separator can also be set inside the
program, in a `BEGIN` rule (chapter 7
explains `BEGIN`; the short version is
"run this first"):

```
$ awk 'BEGIN {FS=":"} {print $1, $7}' passwd.txt | head -3
root /bin/bash
daemon /usr/sbin/nologin
bin /usr/sbin/nologin
```

`-F` is the everyday spelling; the `BEGIN`
form earns its place inside script files,
and when the separator is more than one
character — `FS` accepts a small pattern,
which the patterns chapter of your memory
now supports. For CSV, the separator is a
comma: `-F,`. (Real-world CSV with quoted
commas inside values defeats a one-character
separator — awk's honest limit, stated now
and returned to in the recipes: the lab's
CSV is comma-clean, and files that aren't
want a real CSV parser.)

Two more field names carry their weight.
`$NF` is the **last** field, whatever the
count — `NF` is the number of fields in the
current record, so `$NF` means "the field
whose number is NF":

```
$ awk '{print $NF}' app.log | head -6
started
status=ok
ip=192.168.1.10
ip=192.168.1.10
ip=198.51.100.23
ip=198.51.100.23
```

And `NR` is the record's number — its line
number in the input — which makes printing
with numbers, or selecting by position, a
pattern like any other:

```
$ awk '{print NR, $0}' app.conf | head -4
1 # appserver configuration - specimen file
2 # Lines starting with # are comments.
3 
4 db_host = db.example.test
```

## Patterns: conditions on records

The pattern half of a rule can be a regular
expression between slashes (matched against
the whole record) or an ordinary condition
on fields. With no action written, the
default action is *print the record* — so
awk doubles as a grep that can test numbers:

```
$ awk '/failed/' app.log | wc -l
18
$ awk '$3 == "ERROR"' app.log | wc -l
10
```

18 lines mention *failed*; 10 have ERROR as
field 3 exactly. But the field conditions
are where awk leaves grep behind, because
they *compare*. "Accounts with a uid of
1000 or more" is a question grep can't
honestly ask — it would have to match digit
shapes — and awk asks it directly:

```
$ awk -F: '$3 >= 1000' passwd.txt
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
paul:x:1000:1000:Paul Sullivan:/home/paul:/bin/bash
katherine:x:1001:1001:Katherine Sullivan:/home/katherine:/bin/zsh
guest:x:1002:1002:Guest Account:/home/guest:/bin/sh
backupsvc:x:1003:1003:Backup Service:/srv/backup:/bin/false
oldacct:x:1004:1004:Old Account:/home/oldacct:/usr/sbin/nologin
```

Read it before you trust it: **`nobody` is
in the output**, and nobody is not a person
— its uid is 65534, the conventional
"no user" value, which is indeed ≥ 1000.
awk did exactly what it was told. The fix
is to say what you mean — human uids live
in a range:

```
$ awk -F: '$3 >= 1000 && $3 < 60000 {print $1, $3}' passwd.txt
paul 1000
katherine 1001
guest 1002
backupsvc 1003
oldacct 1004
```

`&&` is *and* (`||` is *or*, `!` is *not*),
and comparisons are numeric when the field
looks like a number, string otherwise. Keep
the nobody line in mind: it's the guide's
standing lesson that a field test is only
as honest as the range you give it.

One pattern idiom appears everywhere,
including this guide's first chapter: the
CSV header row is a record like any other,
so almost every CSV program starts
`NR>1` — "record number greater than 1,"
i.e. skip the header:

```
$ awk -F, 'NR>1 {print $4}' orders.csv
widget
gadget
sprocket
cable
widget
gadget
adapter
cable
widget
sprocket
adapter
cable
widget
gadget
```

That is the item column, all 14 orders.
What it's *for* — adding the quantities up,
counting by item — is computing, and that's
the next chapter.

Next: awk with arithmetic — sums, counts,
averages, and counting by category.
