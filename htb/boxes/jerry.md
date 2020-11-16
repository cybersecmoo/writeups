# HackTheBox Jerry Report

## Executive Summary

The target of this penetration test was found to have two critical vulnerabilities, which would allow an attacker complete access to the system, including any confidential or customer information stored therein.

While the vulnerabilities themselves are very serious, they are also simple to mitigate. By taking the actions suggested in the Vulnerabilities and Suggested Mitigations section, the security of the server and its services can be greatly improved.

## Vulnerabilities and Suggested Mitigations

### Scope

The test was restricted to a single target server, at the internal IPv4 address 10.10.10.95. The scope included attacking any services on the machine, but not carrying out exploits against the availability of the machine or any services therein.

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

#### Default Credentials Used for Tomcat

**Description:** The Tomcat server running on the target machine used a default set of administrator credentials, which are simple to crack (even without knowing them) and are widely-known regardless.

**Mitigation:** Before deploying any service (whether internal or external) always change any default credentials. At the very least, change the passwords, even if the usernames remain default.

**Vulnerability Score:**

| Metric | Score | Comment |
| ------ | ----- | ------- |
| Impact | 9.1   | This vulnerability allows an attacker to deploy arbitrary code to the server, as well as stop/start any services managed by Tomcat, and drop users' sessions |
| Stealth | 5    | A brute-force attack would be detected by standard monitoring methods, unless the attacker was lucky and chose the correct set of default credentials on their first attempt. |
| Ease-of-Exploitation | 10 | The attack only requires entry of credentials into a login prompt. | 
| Overall | 8.0 | This is a critical vulnerability which must be mitigated as soon as possible. |

**Ease of Mitigation:** Easy. Requires some simple configuration changes to ensure the default credentials are changed.

#### Tomcat Running as Administrator

**Description:** The Tomcat service is running under an administrative user account. This means that an attacker with access to run commands via Tomcat (e.g. through exploitation of the above vulnerability) can access all data on the server.

**Mitigation:** The service must be set to run as a non-privileged, dedicated user.

**Vulnerability Score:**

| Metric | Score | Comment |
| ------ | ----- | ------- |
| Impact | 9.1   | The attacker will have complete control of the machine, and can compromise any confidential data held therein, or take the machine offline if they wish. |
| Stealth | 8    | An actor with the permissions required to conduct this attack will need to deploy code to the server, which might be detected by integrity monitoring. However, once on the server itself, as an administrator the attacker could cover their tracks effectively. |
| Ease-of-Exploitation | 8 | Exploiting this vulnerability requires the attacker to write or use a tool to provide them real-time access to execute commands as the administrative user. These are not difficult to obtain or write. | 
| Overall | 8.4 | This is a critical vulnerability which must be mitigated as soon as possible. |

**Ease of Mitigation**: Easy. This requires a creation of a simple new user, and requires Tomcat to be reconfigured to run as that user.

## Narrative

The test followed this general flow:

1. External reconnaisance
2. Exploitation for foothold
3. Internal reconnaisance
4. Privilege escalation
5. Data extraction

This section will provide a narrative, at a technical level, describing how the test progressed, and the thought processes involved. This should be of use to development and security operations centre (SOC) members who wish to know more about the attacking mindset, and about the details of the attack.

### External Reconnaisance

I first ran an NMap scan to find which ports were open on the target and which services were running on those ports:

`sudo nmap -sS -sC -oN nmap-scripted.nmap 10.10.10.95`

One service was found to be listening: Tomcat (version 7.0.88) on port 8080. A second run of NMap, targeting all ports but not running scripts, revealed nothing new.

The next step was to investigate the Tomcat server itself. I went to look at the UI in my web browser and was greeted by a default Tomcat welcome screen, with administrative functionalities locked behind authentication prompts.

### Exploitation for Foothold

A search for known vulnerabilities in this version of Tomcat revealed [CVE-2019-0232](https://www.cvedetails.com/cve/CVE-2019-0232/), which has an associated Metasploit module. Running the module through Metasploit:

```
use exploit/windows/http/tomcat_cgi_cmdlineargs
set rhosts 10.10.10.95
run
```

showed that the server was not vulnerable to this attack.

However, the fact that the server was left with the default Tomcat welcome screen indicated that it may still be using default credentials. Using a publically-available list of default Tomcat passwords with a basic bruteforcing script allowed me to successfully gain access.

After gaining access, I found that I had permission to use the Web Application Manager interface. This allows the uploading of application code to be run on the Tomcat server. It was then a simple matter to build a WAR containing code for a web shell, and deploy that to the target. This gave me access to the target machine, as the Tomcat user.

### Internal Reconnaisance

Once I had access to the target via the web shell, I proceeded to explore the filesystem to see which areas I had access to. To my surprise, I had access to areas which should be restricted to administrative users, which indicated that the Tomcat server was running with Administrator permissions.

### Privilege Escalation

Since the Tomcat user already had administrator rights, privilege escalation was unnecessary in this case.

### Data Extraction

Data within both standard-user and administrative-user directories was extracted via the web shell. In a real attack, this level of access would have permitted stealing of confidential customer information (if stored on this particular server), as well as intellectual property such as source code.