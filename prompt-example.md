Act as a security investigator. Using the Dynatrace MCP tools, investigate the host "idt013933.cc.cec.eu.int" (HOST-4C39E8153120687C) in the digit-dev environment.

Gather:
1. All open vulnerabilities affecting this host or its processes/services
2. The severity and risk score of each vulnerability
3. Affected software packages and their versions
4. Any related problems or events in the last 30 days

Then produce a 1-page security report for the service owner with the following structure:

Write the final report to a Markdown file named `security-assessment-report.md`.
Use Markdown headings, a table for vulnerabilities, and bullet or numbered lists where appropriate.
Keep the file concise, well-formatted, and ready to share with the service owner.

## Security Assessment Report — idt013933.cc.cec.eu.int
**Date:** [today's date]
**Environment:** digit-dev
**Prepared by:** Security Operations

### Executive Summary
2-3 sentences summarizing the overall risk posture.

### Vulnerabilities Found
A table with columns: Priority (P1/P2/P3), CVE ID, Title, Package, Risk Level, Recommendation

### Prioritisation Criteria
- P1 (Critical): RCE, authentication bypass, SSRF — patch within 7 days
- P2 (High): Injection, XSS, TLS bypass — patch within 30 days  
- P3 (Medium): DoS, information disclosure — patch within 90 days

### Recommended Actions
Numbered list of specific remediation steps, ordered by priority.

### Next Steps
What the service owner should do immediately.

Keep it concise, factual, and actionable. No fluff.
