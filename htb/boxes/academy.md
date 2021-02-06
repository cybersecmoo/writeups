# HackTheBox Academy Report

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

**Description:** 

**Mitigation:** 

**Vulnerability Score:**

| Metric | Score | Comment |
| ------ | ----- | ------- |
| Impact |    |  |
| Stealth |     |  |
| Ease-of-Exploitation |  |  | 
| Overall |  |  |

**Ease of Mitigation:** 

The test followed this general flow:

1. External reconnaisance
2. Exploitation for foothold
3. Internal reconnaisance
4. Privilege escalation
5. Data extraction

This section will provide a narrative, at a technical level, describing how the test progressed, and the thought processes involved. This should be of use to development and security operations centre (SOC) members who wish to know more about the attacking mindset, and about the details of the attack.

### External Reconnaisance



### Exploitation for Foothold


### Internal Reconnaisance


### Privilege Escalation

### Data Extraction
