# The Trio Together

Everything so far was one tool at a time.
Real text work is a **pipeline**: the output
of one tool becomes the input of the next,
joined by the shell's `|`, and each stage
does the one thing it's best at. grep
selects the lines; awk picks fields out of
them or computes over them; sed rewrites
what remains. Two companions complete the
counting toolkit — `sort` and `uniq` —
because the question pipelines answer most
often is *how many of each?*

## sort and uniq: the counting pair

`sort` puts lines in order. Alphabetical by
default; the options that matter are
`-n` (numeric order), `-r` (reverse), `-t`
(the field separator), and `-k` (which
field to sort by). Accounts by uid, smallest
first — numeric, because as text "1000"
would sort before "60":

```
$ sort -t: -k3 -n passwd.txt
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
```

(and the rest, ending with `nobody` at
65534 — sort, like awk, takes numbers at
face value). Sorting is blind to structure:
it treats a header row as data. On the order
sheet sorted by quantity, largest first,
the header's `quantity` reads as 0 and sinks
out of sight — convenient here, and worth
remembering whenever a header seems to have
vanished:

```
$ sort -t, -k5 -nr orders.csv | head -3
1010,2026-10-19,guest,sprocket,20,1.75,shipped
1003,2026-10-16,paul,sprocket,12,1.75,shipped
1005,2026-10-17,katherine,widget,5,9.99,shipped
```

`uniq` removes duplicate lines — but only
**adjacent** ones. It reads a stream and
collapses each run of identical neighbours
into one. Fed the log's levels in file
order, it produces a diary of runs, not a
summary:

```
$ awk '{print $3}' app.log | uniq -c | head -8
      4 INFO
      3 WARN
      2 INFO
      1 ERROR
      2 INFO
      1 WARN
      1 ERROR
      1 INFO
```

That's `uniq` working exactly as designed,
and it's why `uniq` is nearly always fed by
`sort`: sorting gathers every like line next
to its kin, and then `-c` counts the runs —
a frequency table:

```
$ awk '{print $3}' app.log | sort | uniq -c
      2 DEBUG
     10 ERROR
     30 INFO
     14 WARN
```

Append one more sort to rank it, biggest
first — `sort -rn`, reverse numeric:

```
$ awk '{print $3}' app.log | sort | uniq -c | sort -rn
     30 INFO
     14 WARN
     10 ERROR
      2 DEBUG
```

`sort | uniq -c | sort -rn` is the single
most useful pipeline shape in this guide:
*list the values, gather them, count them,
rank them.* Which shells the accounts use
is the same three beats — the values come
from awk's field 7 this time:

```
$ awk -F: '{print $7}' passwd.txt | sort | uniq -c
      2 /bin/bash
      1 /bin/false
      1 /bin/sh
      1 /bin/sync
      1 /bin/zsh
      6 /usr/sbin/nologin
```

Half the accounts are nologin — the account
file's story in six lines. And `sort -u`
(same as `sort | uniq`) removes duplicates
while sorting — but *duplicates* means
byte-for-byte, which the messy list defeats
without even trying:

```
$ sort -u messy.txt

  bananas
 APPLES
Apples 
Carrots
ELDERBERRIES
apples
bananas
carrots 
dates
elderberries 
figs
grapes
grapes  
honeydew
```

15 lines in, 15 out: `apples`, `Apples ` and
` APPLES` are three *different* lines to a
computer, exact sorting included. Ordering
is also locale-sensitive — the captures are
the build machine's order, where spaces and
capitals sort before lowercase; yours may
differ in the details. Cleaning the list for
real needs the trio, and it's the recipes
chapter's closing act.

## One pipeline, taken apart

Here is the three tools plus the counting
pair in a single line, answering *which
addresses appear most in the log?* Read it
stage by stage, right to left in your head
but left to right on the screen:

```
$ grep ' ip=' app.log | awk '{print $NF}' | sed 's/^ip=//' | sort | uniq -c | sort -rn
     11 198.51.100.23
      9 192.168.1.10
      6 203.0.113.7
      5 192.168.1.25
      3 10.0.0.5
      1 203.0.113.99
      1 198.51.100.99
```

1. `grep ' ip='` selects the 36 lines that
   carry an address.
2. `awk '{print $NF}'` keeps only the last
   field of each — `ip=198.51.100.23` and
   friends. (Chapter 3's `-oP` trick does
   this with one tool on GNU systems; this
   way is portable.)
3. `sed 's/^ip=//'` strips the `ip=` prefix
   — an anchored substitution, replacing
   three characters with nothing.
4. `sort | uniq -c` counts each address.
5. `sort -rn` ranks them.

When a pipeline misbehaves, debug it the
same way: run it up to the first `|`, look,
add a stage, look again. Every stage's
output is printable, checkable text — that
is the design, and it's why these tools
compose so well.

A smaller specimen, which the recipes reuse
— failed logins per day is grep selecting,
awk taking field 1, and the counting pair:

```
$ grep 'login failed' app.log | awk '{print $1}' | sort | uniq -c
      4 2026-10-17
      5 2026-10-18
      2 2026-10-19
```

Next: the recipes — six mortal questions,
answered end to end with what the guide has
built.
