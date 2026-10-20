# awk: Computing

Chapter 6's awk selected and printed. This
one calculates. The trick is a single idea:
**variables persist from record to record.**
A variable you add to on every line holds,
at the end of the file, a total. awk
programs are a loop you don't have to write
— your rules run once per record, in order,
and the `END` rule runs once after the last
one. That is the whole machine, and it adds
up a column like this:

```
$ awk -F, 'NR>1 {sum += $5} END {print sum}' orders.csv
59
```

59 items across the 14 orders. `{sum += $5}`
ran 14 times, once per order; `END` printed
what accumulated. (A variable starts as 0 —
or as empty, which arithmetic treats as 0 —
the first time it's touched.) Conditions
narrow what accumulates; here is the money
the shipped orders represent, quantity
times price, shipped only:

```
$ awk -F, 'NR>1 && $7 == "shipped" {rev += $5 * $6} END {print rev}' orders.csv
248.66
```

Counting is summing where each match adds
one — `n++` is "add 1 to n":

```
$ awk '$3 == "WARN" {n++} END {print n}' app.log
14
```

And an average is a sum that remembers to
count itself:

```
$ awk -F, 'NR>1 {sum += $5; n++} END {printf "%.2f\n", sum/n}' orders.csv
4.21
```

That command introduces `printf`, awk's
formatting print: `printf "%.2f\n", value`
prints the value with exactly two decimal
places and a newline. Plain `print` of the
money above happened to land on 248.66, but
money and averages deserve stated precision
rather than luck — floating-point arithmetic
is not exact, and `%.2f` is how you say "to
the cent, rounded" out loud. `printf` takes
a format of literal text and slots:
`%d` an integer, `%s` a string, `%8s` a
string padded to width 8, `%-8s` the same
left-aligned. A small table of the orders,
formatted by those rules:

```
$ awk -F, 'NR>1 {printf "%-8s %2d x %.2f\n", $4, $5, $6}' orders.csv | head -4
widget    3 x 9.99
gadget    1 x 24.50
sprocket 12 x 1.75
cable     2 x 6.25
```

Note that `printf` adds no newline of its
own — the `\n` in every format above is
doing that job, and forgetting it is the
classic first printf bug.

## BEGIN and END, and the output separator

`BEGIN` rules run once, before the first
record; `END` once, after the last. `BEGIN`
sets separators (chapter 6), prints
headings, and initialises anything that
shouldn't start empty-by-accident. `END` is
where totals live. Together:

```
$ awk -F, 'BEGIN {print "counting orders"} NR>1 {n++} END {print n, "orders counted"}' orders.csv
counting orders
14 orders counted
```

Print joins its arguments with `OFS`, the
**output field separator** — a space by
default, settable like `FS`:

```
$ awk -F: 'BEGIN {OFS=" -> "} {print $1, $7}' passwd.txt | head -3
root -> /bin/bash
daemon -> /usr/sbin/nologin
bin -> /usr/sbin/nologin
```

## Counting by category: arrays

One total is a variable. A total *per
category* — errors per level, items per
kind — is a table indexed by the category's
name, and awk's one data structure is
exactly that: the **associative array**, an
array whose index is a string. `count[$3]++`
means "in the table `count`, add 1 to the
slot named by field 3":

```
$ awk '{count[$3]++} END {for (lvl in count) print lvl, count[lvl]}' app.log
INFO 30
ERROR 10
WARN 14
DEBUG 2
```

The whole log, summarised: 30 INFO, 14 WARN,
10 ERROR, 2 DEBUG. The `for (x in array)`
loop visits every slot. Same shape, a sum
per category instead of a count — total
quantity ordered, per item:

```
$ awk -F, 'NR>1 {q[$4] += $5} END {for (i in q) print i, q[i]}' orders.csv
widget 11
adapter 6
gadget 4
cable 6
sprocket 32
```

Read either output critically: **the order
is awk's, not yours.** Associative arrays
don't promise an order — the captures above
are the order this awk happened to use —
and "sorted by name" or "biggest first" is
a job for `sort`, which is the next
chapter's whole point. (sprockets, quietly,
are the business: 32 of the 59 items.)

## Several files at once

awk takes multiple files and reads them as
one stream, which makes two variables worth
knowing apart. `NR` is the record number
across *all* the input so far; `FNR` is the
record number within the *current* file;
`FILENAME` is the current file's name:

```
$ awk 'END {print NR, FNR}' app.log passwd.txt
68 12
```

68 records in total, and the last file
(passwd.txt) contributed 12 — so `NR>1`
skips the header of a *first* CSV file, but
in multi-file work the header test you want
is `FNR>1`, one per file. And with
`FILENAME` as an array index, per-file
counts are the level-count trick wearing a
different hat:

```
$ awk '{c[FILENAME]++} END {for (f in c) print f, c[f]}' app.log passwd.txt
passwd.txt 12
app.log 56
```

## Dialects, stated once

Everything in these two chapters is
standard awk — it runs in the build
machine's mawk, in gawk, and in the BSD awk
alike, because it is the common language
they share. The implementations have
private extensions (gawk's `gensub()`,
`BEGINFILE`, and others), which this guide
names only to say it doesn't use them: a
program learned here travels. If you meet
an awk error naming a function you've never
heard of in someone else's script, an
extension is the usual explanation.

Next: the three tools in harness together —
pipelines, with `sort` and `uniq` keeping
count.
