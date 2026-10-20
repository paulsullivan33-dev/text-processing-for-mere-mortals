# Writing Criteria

Every guide in this repo is written to the same
standard. If you're adding or editing a guide —
human or AI — hold it to these.

1. **Plain words, real ideas.** Simple language,
   never dumbed-down concepts. Explain jargon on
   first use; collect terms in the glossary.
2. **No hype.** No marketing adjectives, no
   "revolutionary," no promises about what
   something will do for the reader.
3. **Honest about limits.** Say what the thing
   *can't* do. State uncertainty instead of
   smoothing it over. Validation notes include
   what remains unverified.
4. **No invented numbers.** Benchmarks, defaults,
   and specs are either measured or qualified with
   "depends on your setup." If it wasn't tested,
   say so.
5. **Checkable.** Commands are meant to be run,
   with "Check:" blocks showing what success looks
   like. Sources are linked, not asserted.
6. **Respect the reader's setup.** Call out
   Windows/Mac/Linux differences. Don't assume a
   GPU, a budget, or prior experience.
7. **Safety isn't skipped.** Real risks get
   stated plainly, not buried.
8. **Scoped.** Each guide says what it covers and
   points elsewhere for the rest. No guide
   pretends to be the whole subject.
9. **Structure over prose.** Tables, before/after
   examples, symptom → fix. Scannable beats essay.

## Verification notes for this guide

Standard 3 and 4 applied to this guide,
specifically. The build machine ran Ubuntu
24.04 with **GNU grep 3.11**, **GNU sed
4.9**, and **mawk 1.3.4** as its awk
(there is no gawk on the build machine),
plus GNU coreutils sort and uniq 9.4.
Every quoted output in the chapters is a
capture from those tools, run against the
shipped specimen files. Working transcripts
are kept with the build
(`text-processing-lab/transcripts/`).

**The lab.** Five specimen files in
`examples/`: `app.log` (56 lines of
invented server log over three days),
`passwd.txt` (12 passwd-format accounts),
`orders.csv` (a header and 14 invented
orders), `app.conf` (10 lines, one setting
deliberately commented out and replaced),
and `messy.txt` (15 lines with mixed case,
leading and trailing spaces, duplicates,
and a blank line — the trailing spaces are
load-bearing). Three fixtures are made by
the reader, exactly as the chapters show:
`patterns.txt` and `fix.sed` (typed from
the printed contents) and the `site/` tree
(a copy of `app.log` plus a two-line
readme). Three more are made by commands
quoted in chapter 10: `bin.dat` (a NUL-
containing file from a `printf` shown in
the chapter), `crlf.txt` (likewise), and
the `scratch.txt`/`scratch.txt.bak` pair
in chapter 5, which is a copy of
`messy.txt` before and after a real
in-place edit.

**Verified by running, per chapter:**

- *Ch. 1:* all three `--version` lines as
  printed by the build machine (mawk's
  `--version` answering "mawk 1.3.4
  20240123"); `wc -l` over the five
  specimens; `head` of three of them;
  `grep -c ERROR` (10); the line-10 sed
  pair; the `NR==10` awk field print; the
  CSV item column's first four values.
- *Ch. 2:* the full ERROR listing (10
  lines); `-n` on "login failed" (11
  lines); `-c` counts for ERROR (10) and
  "login ok" (7); `-i` matching a
  differently-cased pattern; `-v nologin`
  (6 lines); `-A`, `-B`, `-C` around the
  account-locked line; `^2026-10-19`
  count (17); `status=ok$` (5 lines);
  `^root` and `bash$`; the `login` /
  `nologin` match (6 lines) and its `-w`
  counterpart (no output, exit 1); `-F`
  on the config address, and on a piped
  `echo` line whose `x`/`y`/`z` prove the
  un-`-F` dots match any character; `-l`
  across three files; per-file `-c`
  counts for `paul` (6/1/5).
- *Ch. 3:* the `[0-9]` lesson — a
  two-digit figure refusing a one-digit
  class (no output, exit 1), then
  matching with the class doubled; the
  escaped-dot count (14) and the
  unescaped count beside it (also 14 on
  this data, as the chapter says); `-E`
  `[0-9]+%` (3 lines) and the BRE `\+`
  spelling of the same; `-Ec 'ERROR|WARN'`
  (24); the `user=(paul|katherine)`
  alternation (10 lines); `-oP` with
  `\K` and with `\d{4}` — both work in
  this build's grep; `-e` with two
  patterns (3 lines); `-f patterns.txt`
  (4 lines); recursive search over the
  made `site/` tree, including `-rl` and
  the `--include='*.txt'` narrowing;
  exit codes 0, 1, and 2 captured via
  `echo $?`, including grep's own
  missing-file message; `-q` both ways
  (silent, exits 0 and 1).
- *Ch. 4:* first-match versus `g` on the
  root line's colons; substitution across
  all 7 "login ok" lines; pass-through of
  unmatched lines (head of a whole-file
  substitution); the `i` flag over the
  messy list's three spellings, with
  `grep -c Apples` afterwards proving
  the file unchanged (1); `-n` with line
  range `5,7p`; pattern address
  `/ERROR/p`; address-plus-substitute
  `/login failed/s/failed/FAILED/p`
  (11 lines); the mixed range `1,/ERROR/p`
  (shown by its last three lines);
  alternate delimiter `s|…|…|` beside
  its backslash-escaped slash form
  (identical outputs); `d` by pattern
  (`/^#/d` on the config), by pattern
  with a count (`/DEBUG/d` leaves 54),
  and by line range (`2,3d`).
- *Ch. 5:* two `-e` edits in one pass;
  `fix.sed` created by the quoted
  `printf`, `cat`ed, and run with `-f`
  (two lines shown by address, 18 LOGIN
  lines counted); BRE capture groups
  `\([a-z]*\)` with `\1` bracketing
  usernames on all 7 login-ok lines;
  the two-group date/time swap (first
  three lines); `&` as whole-match
  (`[ERROR]` on all 10 error lines);
  `-E` groups rewriting `ip=` to
  `address=` on the 3 heartbeat lines;
  in-place editing for real: `cp` to a
  scratch file, `sed -i.bak 's/ *$//'`,
  `ls` showing both files, `diff`
  reporting exactly the 4 lines that
  carried trailing spaces (and exiting
  1 — differences found, as the chapter
  explains), `cat -A` confirming the
  spaces are gone.
- *Ch. 6:* `$1, $3` printing; `-F:` and
  the `BEGIN {FS}` form (same table);
  `$NF` over the log's first six lines;
  `$1` over the whole messy list,
  blank line included; `{print NR, $0}`
  over the config's first four lines;
  pattern-only awk counted through
  `wc -l` (18 "failed" lines, 10 with
  `$3 == "ERROR"`); the uid range test
  **including its trap** — `nobody`
  (65534) genuinely printed by
  `$3 >= 1000`, then excluded by the
  `$3 < 60000` refinement, whose 5-line
  output is also captured; the full
  `NR>1` item column (14 values).
- *Ch. 7:* column sum (59); shipped
  revenue accumulated with a field
  condition, printed raw (248.66) and
  with `printf "%.2f"` (248.66);
  WARN count (14); average (4.21);
  `OFS=" -> "` output; the `printf`
  formatted order table; `BEGIN`/`END`
  framing lines; per-level counts from
  an associative array (INFO 30,
  ERROR 10, WARN 14, DEBUG 2 — quoted in
  the order mawk's `for` loop produced
  them, as the chapter warns); per-item
  quantity sums (sprocket 32 of 59);
  `NR` versus `FNR` over two files
  (68, 12); per-file counts indexed by
  `FILENAME`.
- *Ch. 8:* `uniq -c` on unsorted levels
  (run-length output, first 8 runs
  shown); `sort | uniq -c` (four true
  totals); the ranked form with
  `sort -rn`; UID sort with `-t: -k3 -n`
  (nobody last at 65534); quantity sort
  `-t, -k5 -nr`, top 3; shell frequency
  via awk field 7 through the counting
  pair (nologin 6); `sort -u` on the
  messy list removing nothing (15 lines,
  blank included); the full top-talkers
  pipeline (198.51.100.23 first at 11);
  per-day failed logins (4, 5, 2).
- *Ch. 9:* every recipe run as written,
  including the config trap — `grep
  db_host` returning both the live and
  the commented line — and both fixes;
  `cat -A` on the messy list; the
  cleanup pipeline's 8-line result,
  written to `messy-clean.txt` by
  redirection and `cat`ed back; the
  login-count pipeline (katherine 3,
  paul 4); the shipped-orders sentence
  (9 orders, revenue 248.66, average
  27.63).
- *Ch. 10:* every symptom reproduced:
  `grep CRITICAL` silence with exit 1;
  the missing-file message with exit 2;
  the binary refusal in **this build's
  wording** ("grep: bin.dat: binary
  file matches", exit 0), `-c` on the
  same file (1), and `-a` output shown
  through `cat -v` (NULs rendered
  `^@`); the CRLF file made by the
  quoted `printf`, exposed by
  `cat -A` (`^M$` on both lines),
  `grep 'two$'` failing (exit 1),
  unanchored `grep two` matching, and
  the `sed 's/\r$//'` fix matching;
  awk's empty `$2` on the CSV with the
  default separator (three blank
  lines, real output) and the same
  program fixed by `-F,`; the
  double-quoted awk program printing
  whole lines (run in a context with
  an empty `$1`, as an interactive
  shell presents it) beside the
  single-quoted one printing dates;
  the `&amp;` replacement producing
  `andamp;`, and `\&` producing the
  intended `&`.

**Capture hygiene (declared trims):**
long listings are sometimes shown with
`head`/`tail` in the command itself —
the trim is part of the quoted command,
never applied silently afterwards; two
chapter 2 and 3 listings state their full
counts in the text (11 login-failed
lines, 10 alternation lines) where the
page shows the first five. The capture
harness's own command echoes escaped
some characters for its bookkeeping
(doubled backslashes in its transcripts);
the chapters quote every command in the
form a reader types, and each was
verified against the capture's output.
Exit-code annotations in the transcripts
(the harness's `[exit N]` lines) are not
quoted except where the chapter ran
`echo $?` itself — which is most of the
places the guide discusses a code.

**Ordering, stated once:** `sort`
output and awk's `for (x in array)`
order are the build machine's (its
locale collation; mawk's hash order).
The chapters say both are
implementation weather, not promises:
counts are stable, order is not.

**Documented-only (not run), and why:**

- *BSD and macOS differences:* `sed -i ''`
  for in-place editing without a backup,
  the absence of `grep -P`, and the older
  phrasing "Binary file … matches" are
  stated from the tools' documentation
  and general knowledge, flagged in
  place. No BSD system was available to
  the build; the GNU forms they contrast
  with are all captured.
- *gawk-specific behaviour* is named
  (extensions such as `gensub()` and
  `BEGINFILE` exist) but never used —
  the build machine has no gawk, and
  everything taught is the awk common
  core, run under mawk.
- *BusyBox* versions of the trio (common
  in containers and embedded systems)
  accept most of this guide and a few
  spellings less; named here rather
  than tested, there being no BusyBox
  on the build machine.
- *The hold space* (sed's `h`/`H`/`g`/
  `G`/`x`) is named and scoped out in
  chapter 5, not demonstrated — a
  deliberate scope decision, recorded
  here so it isn't mistaken for an
  oversight.
- *Installation* on a reader's machine
  is described, not performed; the
  tools were already present.

Specimen discipline, for the record: the
only people in the guide are two specimen
accounts in an invented passwd file; the
only addresses are documentation ranges
and private ranges; the orders, prices,
and log events are invented, and their
totals in the text are the captures' own.
Nothing in the guide was run against a
real system file — the account file is a
specimen, and the chapters say to copy it,
not to aim the commands at `/etc/passwd`
or a live log until the reader means it.
