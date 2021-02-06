# HackTheBox Netmon Report

## Executive Summary

The target of this penetration test was found to have two critical vulnerabilities, which would allow an attacker complete access to the system, including any confidential or customer information stored therein.

While the vulnerabilities themselves are very serious, they are also simple to mitigate. By taking the actions suggested in the Vulnerabilities and Suggested Mitigations section, the security of the server and its services can be greatly improved.

## Vulnerabilities and Suggested Mitigations

### Scope

The test was restricted to a single target server, at the internal IPv4 address 10.10.10.152. The scope included attacking any services on the machine, but not carrying out exploits against the availability of the machine or any services therein.

### Scoring Methodology

The method used for scoring vulnerabilities was as follows:

1. The impact of exploiting the vulnerability was assessed via the CVSSv3.1 system.
2. The stealth score was assessed according to the following metric:
    - 0: No stealth, trvially visible to any user.
    - 2: Little stealth, visible to basic monitoring.
    - 5: Moderate stealth, visible to standard monitoring and logging processes
    - 8: High stealth, only visible to specialised or tailored monitoring methods
    - 10: Very high stealth, blends in to normal user behaviour, or is only randomly/intermittently detectable.
3. The ease-of-exploitation score was taken by:
    - 0: Only theoretically possible; no known exploitation vector
    - 2: Requires extremely specialised skills, tools, or knowledge; e.g. those only available to state actors.
    - 5: Requires some specialised skills, due to exploits not being publically available and requiring skill to write, or a need to chain tools together.
    - 8: Requires little skill; exploits are simple to write, or require simple tools.
    - 10: Very little to no skill required; exploits available in common exploitation tools (such as Metasploit).
4. A simple mean is then taken of the three subscores.

#### Anonymous Login Enabled for FTP

**Description:** The FTP server running on the machine has anonymous login enabled; this means any person can connect and start browsing the filesystem without needing to provide valid credentials.

**Mitigation:** Ensure that anonymous login is disabled on the FTP service (and any other services, e.g. SMB).

**Vulnerability Score:**

| Metric | Score | Comment |
| ------ | ----- | ------- |
| Impact | 5.3   | This vulnerability permits a user an initial level of access to the filesystem of the target. |
| Stealth | 6    | It should be comparatively simple to arrange logging of anonymous logins, but that may not be something that is implemented in most cases. |
| Ease-of-Exploitation | 10 | The attack only requires a single command-line input. | 
| Overall | 7.1 | This is a severe vulnerability which should be mitigated as soon as practicable. |

**Ease of Mitigation:** Easy. Requires some simple configuration changes to ensure anonymous login is disabled.

#### Anonymous User has Access to Sensitive Files

**Description:** The `anonymous` user (the user under which anonymous FTP commands are run) has permission to access sensitive files; namely those for the PRTG Network Monitor (including configuration).

**Mitigation:** Disable anonymous login (per the above), and remove the `anonymous` user.

**Vulnerability Score:**

| Metric | Score | Comment |
| ------ | ----- | ------- |
| Impact |  5.3  |  |
| Stealth | 6 |  |
| Ease-of-Exploitation |  |  | 
| Overall |  |  |

**Ease of Mitigation:** 

#### Easy-to-Guess User Password Format

**Description:** 

**Mitigation:** 

**Vulnerability Score:**

| Metric | Score | Comment |
| ------ | ----- | ------- |
| Impact |  |  |
| Stealth |    |  |
| Ease-of-Exploitation |  |  | 
| Overall |  |  |

**Ease of Mitigation:** 

## Narrative

The test followed this general flow:

1. External reconnaisance
2. Exploitation for foothold
3. Internal reconnaisance
4. Privilege escalation
5. Data extraction

This section will provide a narrative, at a technical level, describing how the test progressed, and the thought processes involved. This should be of use to development and security operations centre (SOC) members who wish to know more about the attacking mindset, and about the details of the attack.

### External Reconnaisance

The test began with two NMap scans: one running with scripts but against only the top 1000 most common ports, the other running without scripts but against all 65535 ports.

`sudo nmap -sS -sC -oN nmap-scripted.nmap 10.10.10.152`
`sudo nmap -sS -p- -oN nmap-full.nmap 10.10.10.152`

These scans revealed an SMB service (with guest login enabled), a web server, and an FTP server (with anonymous login enabled).

### Exploitation for Foothold

Gaining a foothold was simple; anonymous FTP access, combined with unnecessarily broad access granted to the `anonymous` user account, allowed me access to many important files, including backups of the Netmon installation.

### Internal Reconnaisance

Once I was connected to the target via FTP, I explored the areas that I could access, which (as noted above) included the backups of the Netmon install. Included in the backup was a configuration file which revealed credentials for accessing the Netmon instance.

### Privilege Escalation

With Netmon credentials obtained, I could leverage an authenticated command-injection vulnerability that had turned up in earlier research into Netmon. Attempting to access the Netmon UI using the credentials saved in the backup was unsuccessful. However, the password in the backup was in the format `<word><year>`; so I tried incrementing the year, and was able to successfully gain access to the admin UI. 

Once in the admin interface, I could exploit the vulnerability (CVE-2018-9276) to add a Powershell notification which would, when fired, add an administrative account to the Windows server.

I then ran a "test" execution of the notification, which added the user; I could then log in to the SMB service.

### Data Extraction

With administrative access to the SMB share, I could steal whatever confidential information I wished.

