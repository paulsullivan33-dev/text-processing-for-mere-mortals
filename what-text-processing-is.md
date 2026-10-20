# What Text Processing Is

A text file, to the tools in this guide, is
a list of **lines**. Each line is a string of
characters ending in an invisible newline,
and most lines that mortals meet are secretly
a table in disguise: a log line is a date, a
time, a level, a component, and a message; a
config line is a name, an equals sign, and a
value; a CSV row is a list of values with
commas between them. Text processing is the
craft of working with that structure using
three ideas — **lines**, **fields** (the
pieces a line splits into), and **patterns**
(descriptions of the lines or pieces you
want).

The material is everywhere once you look:
application and system logs, configuration
files, account and inventory lists, data
exported as CSV, and the output of other
commands, which is just a text file that
hasn't landed yet. The questions are always
the same few. Which lines mention the
failure? How many times did it happen, and
on which day? What do the numbers in column
five add up to? Who keeps knocking on the
door? Three tools, unchanged in their
essentials since the 1970s, answer all of
them.

## The three tools

**grep** selects lines. You give it a
pattern and a file; it prints the lines that
match. It does not change anything, and it
thinks one line at a time. This morning's
question, in its whole form:

```
$ grep -c ERROR app.log
10
```

**sed** edits lines as they stream past —
its name is short for *stream editor*. You
give it an editing command; it prints every
line with the command applied, and the file
itself is untouched unless you explicitly
ask otherwise:

```
$ sed -n '10p' app.log
2026-10-17 09:58:47 ERROR app job import-orders failed reason=timeout
$ sed -n '10p' app.log | sed 's/ERROR/PROBLEM/'
2026-10-17 09:58:47 PROBLEM app job import-orders failed reason=timeout
```

**awk** splits lines into fields and
computes. Each line arrives as `$1`, `$2`,
`$3` … and you write small pattern–action
rules: when a line looks like *this*, do
*that* with its fields. Line 10 again, asked
for its third and fourth fields (the level
and the component):

```
$ awk 'NR==10 {print $3, $4}' app.log
ERROR app
```

And the same shape reads a column out of a
CSV file, skipping its header row:

```
$ awk -F, 'NR>1 {print $4}' orders.csv | head -4
widget
gadget
sprocket
cable
```

Select, edit, compute. Most real work is a
short pipeline of the three, which is where
this guide ends up — but each tool is taught
on its own first, because a pipeline is only
debuggable one stage at a time.

## Have you got them?

All three are standard issue on Linux and
macOS and have been for decades. Ask each
one its version:

```
$ grep --version | head -1
grep (GNU grep) 3.11
$ sed --version | head -1
sed (GNU sed) 4.9
$ awk --version | head -1
mawk 1.3.4 20240123
```

Those captures are the build machine's, and
yours may differ in a way that matters more
than the numbers: these tools have
*dialects*. GNU grep and GNU sed (Linux)
accept options and spellings that the BSD
versions on macOS don't, and "awk" is a
language with several implementations —
gawk, mawk, and the BSD awk among them. This
guide was verified on GNU grep 3.11, GNU
sed 4.9, and mawk 1.3.4, and it teaches the
common core that runs everywhere; wherever
a dialect difference bites — and a few do —
the chapter stops and says so, and
`criteria.md` lists which claims are
documented rather than captured.

## The lab

Five specimen files live in `examples/`.
All of them are invented, and every command
in the guide runs against them:

| File | What it is |
|---|---|
| `app.log` | 56 lines of a small server's log, three days of it — logins, web requests, jobs, disk warnings, errors |
| `passwd.txt` | 12 account lines in passwd format: name, password field, ids, real name, home, shell — colon-separated |
| `orders.csv` | 14 orders plus a header row: id, date, customer, item, quantity, price, status |
| `app.conf` | 10 lines of configuration, including one setting commented out and replaced — a trap for the unwary |
| `messy.txt` | 15 lines of a shopping list with mixed capitals, stray spaces, duplicates, and a blank line |

A first look at three of them:

```
$ wc -l app.log passwd.txt orders.csv app.conf messy.txt
  56 app.log
  12 passwd.txt
  15 orders.csv
  10 app.conf
  15 messy.txt
 108 total
```

```
$ head -4 app.log
2026-10-17 06:00:01 INFO cron job nightly-backup started
2026-10-17 06:04:37 INFO cron job nightly-backup finished status=ok
2026-10-17 07:15:22 INFO auth login ok user=paul ip=192.168.1.10
2026-10-17 07:15:40 INFO http GET /index.html 200 ip=192.168.1.10
```

```
$ head -4 passwd.txt
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
```

Copy the specimens to a scratch folder and
work there — chapter 5 deliberately edits a
file in place, and you want the originals
pristine:

```
$ mkdir textlab
$ cp examples/* textlab/
$ cd textlab
```

(`examples/commands.txt` is a pocket
reference card for the whole guide, and
`examples/lab-notes.txt` describes each
specimen; both come along for the ride.)

Check: the three `--version` commands answer
(one may name a different implementation
than the captures — that is fine, and the
chapters flag where it matters); `wc -l`
reports the five counts above; and
`grep -c ERROR app.log` says 10 in your
scratch folder.

Next: grep, the selector — every option a
beginner actually needs, on the log's real
lines.
