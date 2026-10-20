# Recipes

Six questions a mortal actually asks a text
file, answered end to end with the tools
from the preceding chapters. Each recipe
names its ingredients; nothing new is
introduced, except one semicolon.

## 1. Who talks to this server most?

*Ingredients: grep, awk `$NF`, sed, the
counting pair — chapter 8's pipeline.*

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

Now read it like an operator. 192.168.1.10
is the home workstation — of course it's
busy. But the leader is 198.51.100.23, a
stranger's address, and chapter 3 already
showed its business: failed logins, in
bursts, for `admin`, `root`, `oracle`,
`support` — and one locked account. A
frequency table doesn't just count; it
tells you where to look next.

## 2. How many errors, per day?

*Ingredients: awk field test + counting
array (chapter 7), `sort` to put awk's
unordered END output into date order.*

```
$ awk '$3 == "ERROR" {c[$1]++} END {for (d in c) print d, c[d]}' app.log | sort
2026-10-17 3
2026-10-18 5
2026-10-19 2
```

The 18th was the bad day — which the log
backs up: a failed backup, a locked
account, two timed-out imports, a database
outage, and a disk error. Per-day counting
turns "it feels broken lately" into a date
you can investigate.

## 3. What is the order sheet worth?

*Ingredients: awk with the header skip,
a field condition, accumulation, `printf`
(chapters 6–7).*

```
$ awk -F, 'NR>1 && $7 == "shipped" {rev += $5 * $6; n++} END {printf "%d shipped orders, revenue %.2f, average %.2f\n", n, rev, rev/n}' orders.csv
9 shipped orders, revenue 248.66, average 27.63
```

One command, a sentence of an answer: of 14
orders, 9 shipped, worth 248.66 altogether,
27.63 on average. The cancelled and pending
five are excluded by the `$7` test — the
same program with `"pending"` answers the
follow-up question. This is the recipe that
justifies awk to most mortals; a
spreadsheet is fine until the CSV arrives
fresh every morning and the question is
always the same.

## 4. One setting out of a config file

*Ingredients: grep, anchors (chapter 2).*

The config says where the database lives.
Ask naively:

```
$ grep db_host app.conf
db_host = db.example.test
# db_host = old.example.test
```

Two lines — and one of them is the old
address, commented out when the setting
changed. A script that grabbed "the line
with db_host" could take either. Anchor the
pattern to the start of the line, where a
live setting stands and a comment can't:

```
$ grep '^db_host' app.conf
db_host = db.example.test
```

The other sound approach removes comments
first and asks second — chapter 4's `d` in
grep form:

```
$ grep -v '^#' app.conf | grep db_host
db_host = db.example.test
```

Either way, the lesson generalises: config
files keep their history in comments, and a
careful question names which lines count as
live.

## 5. Clean the messy list, end to end

*Ingredients: sed trimming, awk `NF` and
`tolower`, `sort -u` — and `cat -A` to see
the enemy first.*

The mess, with line-ends made visible:

```
$ cat -A messy.txt
apples$
Apples $
 APPLES$
bananas$
  bananas$
carrots $
Carrots$
dates$
$
elderberries $
ELDERBERRIES$
figs$
grapes  $
grapes$
honeydew$
```

Four faults: leading spaces, trailing
spaces, three capitalisations of everything,
one blank line, and duplicates that aren't
byte-identical. One stage per fault class:

```
$ sed 's/^ *//; s/ *$//' messy.txt | awk 'NF {print tolower($0)}' | sort -u
apples
bananas
carrots
dates
elderberries
figs
grapes
honeydew
```

- **sed** trims: `s/^ *//` deletes spaces at
  the start of the line, `s/ *$//` at the
  end. The semicolon runs two sed commands
  in one invocation — chapter 5's `-e`
  pair compressed; here, honestly, the
  semicolon is the readable form.
- **awk** `NF {…}` runs only on records
  that *have* fields — the blank line,
  trimmed to nothing, has none, and drops
  out. `tolower($0)` lowercases the whole
  record, so the three spellings converge.
- **`sort -u`** sorts and keeps one of
  each — now safe, because after the first
  two stages the duplicates really are
  identical.

Save it the mortal way, by redirecting the
stream to a new file and checking it:

```
$ sed 's/^ *//; s/ *$//' messy.txt | awk 'NF {print tolower($0)}' | sort -u > messy-clean.txt
$ cat messy-clean.txt
apples
bananas
carrots
dates
elderberries
figs
grapes
honeydew
```

15 lines of mess, 8 lines of list, and
`messy.txt` itself untouched — the cleanup
is re-runnable, which matters the day the
list arrives messy again from the same
source.

## 6. Who logs in, and how often?

*Ingredients: grep, awk field, sed strip,
counting pair — recipe 1's shape on a
different field.*

```
$ grep 'login ok' app.log | awk '{print $7}' | sed 's/^user=//' | sort | uniq -c
      3 katherine
      4 paul
```

Field 7 of a login line is its `user=…`
tail; sed strips the key, and the counting
pair does the rest. Seven successful
logins, all accounted for — against 11
failed ones (chapter 2), a ratio worth
knowing about any server you keep.

Next: when the tools answer with silence,
gibberish, or the wrong lines — the
troubleshooting chapter.
