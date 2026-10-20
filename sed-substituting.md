# sed: Substituting

**sed** is a *stream editor*: it reads lines
in, applies editing commands, and writes the
results out — the way a copy editor works
through a manuscript with a pencil, except
at a million lines a second and with no
judgement calls. This chapter teaches its
one indispensable command, substitution,
plus the addressing that aims it. A fact to
hold onto from the first line: **by default
sed changes nothing on disk.** It prints the
edited stream; your file is edited only
when you explicitly ask, in chapter 5.

## The substitute command

The shape is `s/old/new/` — *s* for
substitute, then a pattern, then its
replacement, separated by slashes. sed
applies it to every line and prints every
line, changed or not:

```
$ grep 'login ok' app.log | sed 's/login/sign-in/'
2026-10-17 07:15:22 INFO auth sign-in ok user=paul ip=192.168.1.10
2026-10-17 10:22:08 INFO auth sign-in ok user=katherine ip=192.168.1.25
```

(and five more, each with `login` replaced
by `sign-in`). Two properties surprise every
beginner once. First, lines that don't
match print **unchanged** — sed is a filter,
not a highlighter:

```
$ sed 's/ERROR/problem/' app.log | head -3
2026-10-17 06:00:01 INFO cron job nightly-backup started
2026-10-17 06:04:37 INFO cron job nightly-backup finished status=ok
2026-10-17 07:15:22 INFO auth login ok user=paul ip=192.168.1.10
```

Second, without help the substitution
replaces only the **first** match on each
line. The root account line, colons swapped
for a spaced bar:

```
$ grep '^root' passwd.txt | sed 's/:/ | /'
root | x:0:0:root:/root:/bin/bash
```

Only the first colon moved. The flag `g` —
*global*, written after the final slash —
means every match on the line:

```
$ grep '^root' passwd.txt | sed 's/:/ | /g'
root | x | 0 | 0 | root | /root | /bin/bash
```

Flags live after the closing delimiter, and
they compose. `i` makes the *pattern*
case-insensitive, which the messy list —
one item in three spellings — puts to work:

```
$ sed 's/apples/Apples/i' messy.txt | head -3
Apples
Apples 
 Apples
```

`apples`, `Apples `, and ` APPLES` all
matched and were rewritten with the
replacement's spelling. And here is the
"changes nothing" promise, checked: the
file itself still contains just one line
starting `Apples`:

```
$ grep -c Apples messy.txt
1
```

The edited text existed only in the pipe.
Saving edits is a deliberate act — chapter
5's `-i`, with its seatbelts on.

## Addresses: aiming at some lines

A command with no address applies to every
line. Prefix one, and it applies only where
the address matches. An address can be a
line number, a range of line numbers, or a
pattern between slashes. Read lines 5–7 and
nothing else:

```
$ sed -n '5,7p' app.log
2026-10-17 08:41:03 WARN auth login failed user=admin ip=198.51.100.23
2026-10-17 08:41:09 WARN auth login failed user=admin ip=198.51.100.23
2026-10-17 08:41:31 WARN auth login failed user=root ip=198.51.100.23
```

That command introduces the other half of
sed's everyday grammar. `p` is the *print*
command, and `-n` tells sed **not** to
print lines automatically. Think of sed as
carrying two printing switches: by default
every line prints (that's why substitutions
show the whole file, changed and unchanged
alike); `-n` turns the automatic printing
off, and then a line appears only when a
command with `p` says so. `-n` plus `p` is
sed's equivalent of grep:

```
$ sed -n '/ERROR/p' app.log | head -3
2026-10-17 09:58:47 ERROR app job import-orders failed reason=timeout
2026-10-17 15:44:10 ERROR http POST /orders 500 ip=203.0.113.7
2026-10-17 20:47:33 ERROR app job export-report failed reason=disk-full
```

Addresses and commands combine — apply a
substitution *only* on lines the address
selects, and print only those:

```
$ sed -n '/login failed/s/failed/FAILED/p' app.log
2026-10-17 08:41:03 WARN auth login FAILED user=admin ip=198.51.100.23
2026-10-17 08:41:09 WARN auth login FAILED user=admin ip=198.51.100.23
```

(and nine more — every failed login, with
only those lines edited and shown). This is
the shape precise sed work takes: *on lines
like this, change that*.

A range can mix forms — a number and a
pattern. "From the top of the file to the
first error" is `1,/ERROR/`; shown here by
its last three lines:

```
$ sed -n '1,/ERROR/p' app.log | tail -3
2026-10-17 09:02:55 INFO http GET /guide/curl.html 200 ip=203.0.113.7
2026-10-17 09:03:12 INFO http GET /images/logo.png 200 ip=203.0.113.7
2026-10-17 09:58:47 ERROR app job import-orders failed reason=timeout
```

## Other delimiters

The slashes in `s/old/new/` are a habit,
not a law: **the character right after `s`
is the delimiter**, whatever it is. Path
work makes the habit painful — a pattern
full of slashes forces a thicket of
backslashes:

```
$ grep 'bash$' passwd.txt | sed 's/\/bin\/bash/\/usr\/bin\/bash/'
root:x:0:0:root:/root:/usr/bin/bash
paul:x:1000:1000:Paul Sullivan:/home/paul:/usr/bin/bash
```

Every `\/` is a slash that means "part of
the text, not a separator" — correct and
nearly unreadable. Pick a delimiter that
doesn't appear in the text — `|` is the
usual choice — and the same edit reads
itself:

```
$ grep 'bash$' passwd.txt | sed 's|/bin/bash|/usr/bin/bash|'
root:x:0:0:root:/root:/usr/bin/bash
paul:x:1000:1000:Paul Sullivan:/home/paul:/usr/bin/bash
```

(The `s/:/ | /` examples earlier were
quietly doing the same thing in reverse —
there the *replacement* contained the bar.)

## Deleting lines

The `d` command deletes lines the address
selects; nothing prints for them. "The
config file without its comments" is the
everyday case:

```
$ sed '/^#/d' app.conf

db_host = db.example.test
db_port = 5432
db_name = appdb
log_level = INFO
cache_enabled = true
max_connections = 50
```

The blank line is still there — it isn't a
comment, and sed does what it's told, no
more. Addresses work for `d` exactly as for
the other commands:

```
$ sed '/DEBUG/d' app.log | wc -l
54
$ sed '2,3d' app.conf
# appserver configuration - specimen file
db_host = db.example.test
db_port = 5432
# db_host = old.example.test
db_name = appdb
log_level = INFO
cache_enabled = true
max_connections = 50
```

54 is 56 minus the two DEBUG lines; `2,3d`
removed the second and third lines of the
config (its blank line and the first
comment). And again — printed streams only.
`app.conf` on disk still has all ten lines.

Next: several commands at once, remembered
patterns, and the deliberate act of editing
the file itself.
