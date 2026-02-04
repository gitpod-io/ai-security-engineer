# Your Ona AI Security Engineer

A 100% autonomous AI engineer who monitors for CVEs and other vulnerabilities and drives fixes.

## Data Flow

```mermaid
flowchart TD
    subgraph sources["Vulnerability Sources"]
        subgraph scanner_sources["Scanner-covered (48h)"]
            NVD["NVD API"]
            OSV_API["OSV API"]
            GH["GitHub Advisories"]
        end
        subgraph non_scanner["Non-scanner (2 weeks)"]
            CISA["CISA KEV<br/>(Actively Exploited)"]
            AWS["AWS Bulletins"]
            GCP["GCP Bulletins"]
        end
    end

    subgraph feed["Fetch Vulnerability Feeds"]
        FILTER["Filter: HIGH/CRITICAL<br/>Stack: Go, TS, JS, Kotlin, Java, Python"]
        SCANNER_CHECK["Skip if already<br/>in Trivy/OSV"]
        FILTER --> SCANNER_CHECK
    end

    subgraph linear_adv["Linear: Advisories"]
        ADV["[Advisory] ID: description<br/>Label: advisory<br/>(early warning only)"]
    end

    subgraph scan["Scan Repository"]
        TRIVY["Trivy Scanner"]
        OSV_SCAN["OSV Scanner"]
        ADV_CHECK["Advisory Matcher"]
        CONSOLIDATE["Consolidate Findings<br/>HIGH/CRITICAL only"]
    end

    subgraph linear_find["Linear: Findings"]
        FIND["[Finding] ID - repo<br/>Label: finding"]
    end

    subgraph fix["Fix Security Issue"]
        SELECT["Select Finding"]
        CHECK["Check if Fixed"]
        IMPLEMENT["Implement Fix"]
        PR["Create Pull Request"]
    end

    subgraph drive["Drive PR until merge"]
        PR_CHECK["Find assigned PRs"]
        BUILD["Fix build failures"]
        FEEDBACK["Address review feedback"]
        MERGE["Enable auto-merge"]
    end

    NVD --> FILTER
    OSV_API --> FILTER
    GH --> FILTER
    CISA --> FILTER
    AWS --> FILTER
    GCP --> FILTER
    SCANNER_CHECK -->|not in scanners| ADV
    ADV -->|7 days old OR scanner detects| CLOSE["Auto-close"]

    TRIVY --> CONSOLIDATE
    OSV_SCAN --> CONSOLIDATE
    ADV --> ADV_CHECK
    ADV_CHECK --> CONSOLIDATE
    CONSOLIDATE --> FIND

    FIND --> SELECT
    SELECT --> CHECK
    CHECK -->|needs fix| IMPLEMENT
    IMPLEMENT --> PR
    CHECK -->|already fixed| COMMENT["Link existing PR"]

    PR --> PR_CHECK
    PR_CHECK --> BUILD
    PR_CHECK --> FEEDBACK
    PR_CHECK --> MERGE
```

## Automations

| Automation | Trigger | Input | Output |
|------------|---------|-------|--------|
| `Fetch Vulnerability Feeds` | Manual/Scheduled | NVD, OSV, GitHub (48h), CISA KEV, AWS, GCP (2 weeks) | Advisory issues for vulns not yet in scanners |
| `Scan Repository` | Manual | Repository code + Advisories | Finding issues (label: `finding`) |
| `Fix Security Issue` | Manual | Finding issues | Pull requests |
| `Drive PR until merge` | Manual | Open PRs assigned to user | Merged PRs |

## Vulnerability Sources

| Source | Scanner Coverage | Time Window | Purpose |
|--------|-----------------|-------------|---------|
| NVD API | Trivy, OSV | 48 hours | Early warning before scanner DB update |
| OSV API | OSV-Scanner | 48 hours | Early warning before scanner DB update |
| GitHub Advisories | Trivy, OSV | 48 hours | Early warning before scanner DB update |
| CISA KEV | None | 2 weeks | Actively exploited vulnerabilities |
| AWS Bulletins | None | 2 weeks | Cloud-specific issues |
| GCP Bulletins | None | 2 weeks | Cloud-specific issues |

## Linear Issue Types

**Advisories** (`[Advisory]` prefix, `advisory` label)
- Early warning for vulnerabilities not yet in scanner databases
- Only created if: affects our stack AND not in Trivy/OSV
- Includes "Why This Was Flagged" reason
- Supports multiple ID types: CVE, GHSA, or vendor-specific
- Auto-closed when scanners add coverage or after 7 days

**Findings** (`[Finding]` prefix, `finding` label)
- Confirmed vulnerability in a specific repository
- Created by Trivy/OSV scanner or advisory match
- Contains exact file locations and versions
- Supports advisory IDs: CVE, GHSA, or vendor-specific

## Tech Stack Coverage

- **Languages:** Go, TypeScript, JavaScript, Kotlin, Java, Python
- **Cloud:** AWS, GCP
- **Scanners:** Trivy (filesystem), OSV-Scanner (dependencies)
