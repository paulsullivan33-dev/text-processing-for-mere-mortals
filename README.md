# Text Processing for Mere Mortals

Most of the text a mortal ever has to wrestle
is not prose. It is **lines**: log files,
config files, account lists, CSV exports, and
the output of other commands — thousands of
nearly identical lines, with the three you
need buried in the middle. Three old tools
exist for exactly this material. **grep**
selects the lines that match a pattern.
**sed** edits lines as they stream past.
**awk** splits each line into fields and
computes with them. Between them they answer
most questions a text file can be asked:
what failed, how often, how much, and who.

This guide teaches the trio the way it is
actually used: one specimen log read six
ways, one account file taken apart field by
field, one order sheet totalled, one config
setting extracted from under a decoy, and
one messy list cleaned end to end.

Almost everything happens in a small lab
built for the purpose: five specimen files
in `examples/` — a three-day application log
(`app.log`), a passwd-style account file
(`passwd.txt`), an order sheet (`orders.csv`),
a config file with a commented-out trap
(`app.conf`), and a deliberately messy list
(`messy.txt`). Every name, address, and
number in them is invented. The external
addresses use the documentation ranges
(198.51.100.x, 203.0.113.x) reserved for
examples, and the specimens are worth
nothing.

A note on how this guide was verified.
GNU grep 3.11, GNU sed 4.9, and mawk 1.3.4
(the build machine's awk) were on the build
machine, and every command quoted was run
against the lab files above and captured:
every match count, every total, every exit
code. Where another system's tools behave
differently — macOS and the BSDs mostly —
the chapters say so in place and
`criteria.md` keeps the ledger of what is
documented-only.

## What this guide covers

- [**What text processing is**](what-text-processing-is.md) —
  lines, fields, and patterns; the three
  tools and their three shapes; checking
  your versions; the lab
- [**grep: finding lines**](grep-finding-lines.md) —
  literal search, counting, line numbers,
  ignoring case, inverting, context, anchors,
  whole words, and fixed strings
- [**grep: patterns and power**](grep-patterns-and-power.md) —
  character classes, basic versus extended
  patterns, Perl patterns named honestly,
  many patterns at once, searching trees,
  and grep's exit codes
- [**sed: substituting**](sed-substituting.md) —
  the `s///` command, first match versus
  global, addresses, printing only what you
  ask for, other delimiters, and deleting
  lines
- [**sed: doing more**](sed-doing-more.md) —
  several commands at once, script files,
  capture groups and the whole-match `&`,
  and editing files in place — with a backup
- [**awk: fields and records**](awk-fields-and-records.md) —
  the pattern–action shape, `$0`, `$1`, `$NF`,
  field separators, record numbers, and
  conditions on fields
- [**awk: computing**](awk-computing.md) —
  `BEGIN` and `END`, sums, counts, and
  averages, formatted output, counting by
  category with arrays, and working across
  several files
- [**The trio together**](the-trio-together.md) —
  pipelines, and `sort` and `uniq` as the
  counting companions: frequency tables and
  rankings
- [**Recipes**](recipes.md) — the log's top
  talkers, errors per day, the order sheet's
  money, one config setting, the messy list
  cleaned, and who logs in
- [**Troubleshooting**](troubleshooting.md) —
  silence that means "no match," binary
  files, Windows line endings, the field
  separator you forgot, and quotes that eat
  your `$1`
- [**Glossary**](glossary.md) — the terms,
  collected in one place

## What it doesn't cover

Regular expressions as a subject in
themselves — the sibling guide *Regular
Expressions for Mere Mortals* teaches them
properly, and chapter 3 borrows only what
grep needs. JSON and other structured data
(the sibling *jq for Mere Mortals* owns
that), perl and Python as text processors
(named, not taught), and full programming in
awk — this guide stops at the one-program
size mortals actually write. Where a tool
has a dialect difference that bites —
GNU versus BSD sed, one awk versus another —
the chapter says so in place.

## How to use this guide

Read chapter 1, copy the lab files to a
scratch folder as it directs, and keep a
terminal open from chapter 2 on. Work in
the scratch copy: chapter 5 edits a file in
place on purpose, and the specimens should
survive your experiments so you can start a
chapter over. Keep the lab afterward — a log
with known answers is a standing rehearsal
for the day a real one asks the same
questions.
