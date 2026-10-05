
# Linux Authentication Log Investigation

## 1. Project Overview

This project involved examining authentication-related and system session logs on an Ubuntu Linux virtual machine. The objective was to understand how Linux records login sessions, privilege escalation through `sudo`, scheduled tasks, and other authentication-related activity.

The investigation represents a beginner-level Security Operations Center (SOC) exercise focused on log analysis, event interpretation, and distinguishing routine system activity from events requiring further investigation.

## 2. Objectives

- Explore Linux authentication logs.
- Identify user and system session activity.
- Understand how `sudo` commands are recorded.
- Filter log entries using `grep`.
- Review login history using `last`, `lastlog`, and `who`.
- Interpret log events and avoid false positives.
- Develop a basic incident investigation workflow.

## 3. Lab Environment

| Component | Details |
|---|---|
| Operating System | Ubuntu Linux |
| Virtualization | Oracle VirtualBox |
| Log Source | `/var/log/auth.log` |
| Tools | Linux Terminal, `grep`, `tail`, `last`, `lastlog`, `who` |
| Investigation Type | Local authentication and session log analysis |

## 4. Investigation Methodology

### Step 1: Examine Recent Authentication Logs

Command:

```bash
sudo tail -n 100 /var/log/auth.log
```

Purpose:

Displayed the most recent 100 lines of the authentication log to identify recorded login sessions, privilege escalation events, scheduled tasks, and other authentication-related activity.

### Step 2: Search for Failed or Suspicious Authentication-Related Messages

Command:

```bash
sudo grep -iE "failed|failure|invalid|authentication failure" /var/log/auth.log | tail -n 30
```

Purpose:

Searched the log for entries containing keywords associated with failures or authentication problems.

Observation:

The results included a D-Bus service timeout involving `org.bluez`, as well as a record of the investigation command itself.

The D-Bus message was:

```text
Failed to activate service 'org.bluez':
timed out (service_start_timeout=25000ms)
```

Interpretation:

This message indicates that a system service activation timed out. It is not, by itself, evidence of a failed login or malicious activity.

The search also demonstrated an important limitation: searching for the word `failed` can return unrelated system errors. Log entries must be interpreted in context.

### Step 3: Search for Successful Authentication Events

Command:

```bash
sudo grep -i "accepted" /var/log/auth.log | tail -n 20
```

Purpose:

Searched for entries containing the word `accepted`, commonly associated with successful SSH authentication.

Observation:

The visible results did not establish a successful SSH authentication event. The output included a record of the search command itself.

Interpretation:

No successful SSH authentication event was confirmed from this search. This does not prove that no successful login occurred, because desktop logins and other authentication mechanisms may use different log messages.

### Step 4: Investigate Session Activity

Command:

```bash
sudo grep -i "session opened" /var/log/auth.log | tail -n 30
```

Purpose:

Reviewed session-opening events to identify which users or services opened sessions.

Observed activity included:

- Desktop login activity involving `gdm-password`.
- Sessions opened for the user account.
- Privileged sessions opened through `sudo`.
- Scheduled task sessions associated with `CRON`.
- System sessions involving services such as `systemd` and `polkit`.

Interpretation:

These entries are consistent with routine operating system activity. The presence of a root session is not automatically suspicious; its initiating process, timestamp, user, and associated command must also be considered.

### Step 5: Review Commands Executed Through Sudo

Example log entry:

```text
sudo: fagbemi-elisha-kehinde :
TTY=pts/0 ;
PWD=/home/fagbemi-elisha-kehinde ;
USER=root ;
COMMAND=/usr/bin/whoami
```

Interpretation:

- `sudo`: The command was executed through the privilege-escalation mechanism.
- `TTY=pts/0`: Identifies the terminal session.
- `PWD`: The working directory when the command was executed.
- `USER=root`: The command was executed with root privileges.
- `COMMAND=/usr/bin/whoami`: Identifies the command executed.

The logs also recorded investigation commands such as `tail` and `grep`. This demonstrated that the actions performed during an investigation can themselves generate audit records.

### Step 6: Examine Login History

Command:

```bash
last
```

Purpose:

Reviewed recorded login sessions, system boots, session endings, and historical entries marked as crashes.

Observation:

The output showed current terminal and desktop-related sessions, together with historical entries containing `crash`, `down`, and reboot information.

Interpretation:

These entries provide useful session-history context. A session marked `crash` does not, on its own, establish malicious activity; additional system logs would be needed to determine why a session ended unexpectedly.

### Step 7: Review Last Login Records

Command:

```bash
lastlog
```

Purpose:

Reviewed the last-login records available for local accounts.

Observation:

The output showed `Never logged in` for several accounts, including the root account and the named user account.

Interpretation:

This output should be treated cautiously because `lastlog` relies on its own login-recording mechanism. Its results did not fully align with the active sessions shown by `who` and the historical entries shown by `last`.

Further validation would be required before drawing a conclusion about the discrepancy.

### Step 8: Identify Currently Logged-In Sessions

Command:

```bash
who
```

Purpose:

Identified currently recorded login sessions.

Observation:

The output showed the user account associated with a desktop login-screen session (`seat0`) and a terminal session (`tty2`).

Interpretation:

These entries provide a snapshot of the sessions recorded at the time of the investigation. They are useful when comparing current activity against authentication logs and historical session records.

## 5. Key Findings

1. **Privilege escalation:** The authentication logs recorded commands executed through `sudo`, including the investigative commands used in this project.

2. **Routine system activity:** The logs contained scheduled `CRON` sessions, desktop login activity, system sessions, and authentication-related service activity.

3. **Service timeout:** A D-Bus message reported a timeout while activating `org.bluez`. The available evidence does not establish that this event was security-related.

4. **No confirmed failed login:** The keyword search did not establish a failed authentication attempt. The matching results included an unrelated service timeout and a record of the search command itself.

5. **No confirmed successful SSH login:** The search did not establish a successful SSH authentication event in the visible results.

6. **Session-history discrepancy:** The `last` and `who` outputs showed sessions, while `lastlog` reported that the named user had never logged in. This discrepancy would require further validation.

7. **Historical session endings:** Some older entries were marked as crashes. These entries alone do not establish a security incident.

## 6. Security Assessment

**Verdict: No confirmed security incident identified from the evidence reviewed.**

The investigation identified routine system activity and one service activation timeout. The available results did not provide sufficient evidence to classify any event as malicious.

This conclusion is limited to the logs and commands examined. It does not establish that the entire system is free from compromise.

## 7. Skills Practised

- Linux command-line navigation and investigation.
- Authentication log inspection.
- Text searching and filtering with `grep`.
- Output management using `tail`.
- Reviewing user sessions and login history.
- Understanding `sudo` and root privileges.
- Basic security event triage.
- Distinguishing system errors from authentication failures.
- Evidence-based reporting and avoiding false positives.

## 8. Evidence

Screenshots documenting the investigation are stored in the `screenshots/` directory.

Suggested evidence:

- Authentication log inspection using `tail`.
- Searching authentication logs with `grep`.
- Reviewing session history with `last`.
- Checking login records with `lastlog`.
- Identifying current sessions with `who`.

## 9. Limitations

- The investigation was conducted on a single Ubuntu virtual machine.
- The analysis was limited to locally available log records.
- No confirmed SSH authentication events were identified in the displayed search results.
- No centralized monitoring or correlation with network events was performed.
- The session-history discrepancy was not fully investigated.
- The available evidence was insufficient to establish whether any security incident occurred outside the reviewed records.

## 10. Lessons Learned

This project demonstrated that Linux records a variety of user, system, and privilege-escalation events that can be examined during security investigations.

It also reinforced the importance of contextual analysis. A keyword match is only a starting point; an analyst must examine the timestamp, process, account, command, and surrounding events before classifying an alert.

## 11. Next Steps

The next stage of the portfolio will focus on centralized security monitoring with Wazuh, including collecting endpoint logs, exploring security alerts, and investigating events through a security monitoring dashboard.
