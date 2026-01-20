# Security Fix Pipeline Demo

Automated vulnerability detection and remediation: fetch threat intel, scan, fix, and ship.

```mermaid
flowchart LR
    A[Fetch CISA KEV] --> B[Trivy Scan]
    B --> C[Select Top Finding]
    C --> D[Implement Fix]
    D --> E[Test]
    E --> F[Create PR]
```

**Steps:**
1. **Fetch** - CISA Known Exploited Vulnerabilities (not in scanners)
2. **Scan** - Trivy finds HIGH/CRITICAL issues
3. **Select** - Prioritize: KEV > CRITICAL > HIGH > oldest
4. **Fix** - Update dependency, adapt code if needed
5. **Test** - Build, run tests, verify CVE resolved
6. **PR** - Submit fix for review
