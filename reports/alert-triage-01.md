# Alert triage 01: repeated failed sudo attempts (lab detection exercise)

**Analyst:** Anggasukmana Adisastra
**Environment:** my own Kali Linux lab VM, running OSSEC HIDS
**Verdict:** True positive for the detection; benign cause (a test I ran on purpose)
**Detection gap found and fixed:** OSSEC was not reading the login log. See "What went wrong first"

## Summary

I triggered three failed `sudo` attempts on purpose, then checked whether my monitoring noticed. At first it did not: OSSEC raised nothing, because `/var/log/auth.log` was missing from its list of watched files. After I added that file and restarted OSSEC, a repeat of the same test produced 12 alerts, which I then triaged as I would a real one.

## What went wrong first

| | |
|---|---|
| **Test** | Three wrong `sudo` passwords at 16:48:20 to 16:48:26 |
| **System log** | `/var/log/auth.log` recorded all three failures, with the user, terminal and attempted command |
| **OSSEC alerts** | None. The alert file had no new entries for that time |
| **Root cause** | OSSEC's config watched package, web-server and monitoring logs, but not `auth.log` |
| **Fix** | Added a `localfile` entry for `/var/log/auth.log` (syslog format), tested the config, restarted OSSEC |
| **Retest** | Same test at 17:24:56 to 17:25:03. All three failures were alerted |

A monitoring tool that is running is not the same as a monitoring tool that is watching the right logs. I would not have known without testing it.

## Timeline of the retest (times from `auth.log`)

| Time | What the system log showed | OSSEC rules raised |
|---|---|---|
| 17:24:56 | First `sudo` authentication failure (user `er723`, terminal `pts/0`, local) | 5557 Password check failed; 5503 User login failed |
| 17:24:58 | "1 incorrect password attempt", command `/usr/bin/true` as root | 5401 Failed attempt to run sudo; 1002 Unknown problem somewhere in the system |
| 17:25:00 | Second failure, same pattern | Same four rules |
| 17:25:03 | Third failure, same pattern | Same four rules |

Each failed attempt produced four alerts, so 12 in total. The rules were low severity (level 2 to level 5). Alert times can run about two seconds behind the log line.

## Triage

**Who, what, where:** user `er723` on terminal `pts/0`, trying to run `/usr/bin/true` as root. The remote-host field is empty, so the attempts were local. The attempts came in a regular rhythm about two seconds apart, which looks scripted, not like a person mistyping.

**What a Tier 1 analyst would check for a real alert like this:**
1. Is this a known user, and did they expect to be using `sudo` at that time?
2. Is it local or remote? (Empty `rhost` and a local terminal: local.)
3. How many failures, and how fast? (Three in about seven seconds, then it stopped.)
4. Did a **successful** `sudo` or login follow the failures? That would suggest a guessed password.
5. Anything else from the same user or host around that time: new accounts, changed files, new listening ports.
6. Other users or hosts showing the same pattern.

**Result here:** all three failures came from me, from a script I ran deliberately. Nothing succeeded afterwards. **Closed as benign (authorised test).**

**When I would escalate to Tier 2:** failures from an unfamiliar user or remote address, a long run of failures (not three), a successful login right after failures, or other suspicious activity from the same account.

## Observations

- **Noise:** one failed attempt raised four alerts. A real queue would be crowded quickly. The catch-all rule 1002 added no useful information and could be tuned out.
- **No correlation:** none of the rules fired on the pattern "several failures in a row". Each alert stood alone, so an analyst has to join them up by hand. A custom rule that raises severity after repeated failures would help.
- **Extra check done:** while investigating, I confirmed nobody else was logged in, found no other failed attempts that day, and noticed SSH was listening on all addresses although I don't use it. I switched it off.
- **Two higher-severity alerts explained:** at 17:16 OSSEC raised two level 7 alerts (rule 553, "File deleted") for `/etc/systemd/system/multi-user.target.wants/ssh.service` and `/etc/systemd/system/sshd.service`. These are the service links removed when I disabled SSH, so they are expected and benign. A file-integrity alert always needs to be matched against what the administrator actually changed.

## What I did not do

- I did not run my own `soc-log-analyser` tool on this data. That would be a good follow-up.
- I did not write a custom OSSEC rule for repeated failures.
