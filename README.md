# Jonas, your Ona AI Security Engineer

Jonas is a 100% autonomous AI engineer who monitors for CVEs and other vulnerabilities and drives fixes.

## Data Flow

```mermaid
flowchart TD
    subgraph sources["Vulnerability Sources"]
        NVD["NVD API<br/>(CVE Feed)"]
        OSV_API["OSV API<br/>(Go/npm/Maven/PyPI)"]
        GH["GitHub Advisories"]
        CISA["CISA KEV<br/>(Actively Exploited)"]
        GOVULN["Go Vuln DB"]
        UBUNTU["Ubuntu USN"]
        DEBIAN["Debian Security"]
        AWS["AWS Bulletins"]
        GCP["GCP Bulletins"]
    end

    subgraph feed["[Jonas] Fetch Vulnerability Feeds"]
        FETCH["Filter: HIGH/CRITICAL<br/>Tech stack: Go, TS, JS, Kotlin, Java"]
    end

    subgraph linear_adv["Linear: Advisories"]
        ADV["[Advisory] ID: description<br/>Label: advisory<br/>(ID = CVE/GHSA/USN/DSA/GO-/vendor)"]
    end

    subgraph scan["[Jonas] Scan Repository"]
        TRIVY["Trivy Scanner"]
        OSV_SCAN["OSV Scanner"]
        ADV_CHECK["Advisory Matcher"]
        CONSOLIDATE["Consolidate Findings<br/>HIGH/CRITICAL only"]
    end

    subgraph linear_find["Linear: Findings"]
        FIND["[Finding] CVE-XXXX-XXXXX - repo<br/>Label: finding"]
    end

    subgraph fix["[Jonas] Fix Security Issue"]
        SELECT["Select Finding"]
        CHECK["Check if Fixed"]
        IMPLEMENT["Implement Fix<br/>• dependency_upgrade<br/>• code_migration<br/>• config_change<br/>• code_replacement"]
        PR["Create Pull Request"]
    end

    NVD --> FETCH
    OSV_API --> FETCH
    GH --> FETCH
    CISA --> FETCH
    GOVULN --> FETCH
    UBUNTU --> FETCH
    DEBIAN --> FETCH
    AWS --> FETCH
    GCP --> FETCH
    FETCH --> ADV
    ADV -->|after 7 days| CLOSE["Auto-close<br/>with comment"]

    TRIVY --> CONSOLIDATE
    OSV_SCAN --> CONSOLIDATE
    ADV --> ADV_CHECK
    ADV_CHECK --> CONSOLIDATE
    CONSOLIDATE --> FIND

    FIND --> SELECT
    SELECT --> CHECK
    CHECK -->|needs fix| IMPLEMENT
    IMPLEMENT --> PR
    CHECK -->|already fixed| COMMENT["Comment on Linear<br/>with existing PR"]
```

## Automations

| Automation | Trigger | Input | Output |
|------------|---------|-------|--------|
| `[Jonas] Fetch Vulnerability Feeds` | Manual/Scheduled | NVD, OSV, GitHub, CISA KEV, Go Vuln DB, Ubuntu USN, Debian, AWS, GCP | Advisory issues (label: `advisory`), auto-closes after 7 days |
| `[Jonas] Scan Repository` | Manual | Repository code + Advisories | Finding issues (label: `finding`) |
| `[Jonas] Fix Security Issue` | Manual | Finding issues | Pull requests |

## Linear Issue Types

**Advisories** (`[Advisory]` prefix, `advisory` label)
- General vulnerability awareness from public feeds
- Not repository-specific
- Contains detection guidance for engineers
- Includes "Why This Was Flagged" reason (CVSS score, CISA KEV, ecosystem match)
- Supports multiple ID types: CVE, GHSA, USN, DSA, GO-, or vendor-specific
- Auto-closed after 7 days with explanatory comment

**Findings** (`[Finding]` prefix, `finding` label)
- Confirmed vulnerability in a specific repository
- Created by scanner or advisory match
- Contains exact file locations and versions