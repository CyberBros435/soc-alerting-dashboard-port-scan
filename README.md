# 🛡️ Port Scan Alerting & Dashboard — Splunk Automation Layer

![Splunk](https://img.shields.io/badge/SIEM-Splunk-black?logo=splunk)
![Status](https://img.shields.io/badge/status-complete-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)

Converted port scan detection logic into two operational SOC
artifacts: a scheduled alert flagging high-risk exposed ports, and a
dashboard panel visualizing risk distribution — closing the gap
between "we can detect this" and "this is actively monitored."

**Full report:** [report/report.md](report/report.md)

---

## 📌 Table of Contents
- [Overview](#overview)
- [Alert](#alert)
- [Dashboard Panel](#dashboard-panel)
- [Data Loss Note](#data-loss-note)
- [What I Learned](#what-i-learned)
- [Related Projects](#related-projects)

---

## Overview

| | |
|---|---|
| **Goal** | Operationalize port scan detection via alert + dashboard |
| **SIEM** | Splunk Enterprise (local lab) |
| **Builds on** | [Port Scan Detection](https://github.com/CyberBros435/soc-port-scan-detection-splunk), [Lookup Enrichment](https://github.com/CyberBros435/soc-lookup-enrichment-splunk) |
| **Deliverables** | 1 scheduled alert, 1 new dashboard panel (3rd panel on existing SOC Detection Dashboard) |

## Alert

**Query**:
```spl
index=main sourcetype=<sourcetype> event_type=port_probe
| lookup port_risk_lookup dst_port OUTPUT risk_label
| search risk_label="*high risk*"
```

| Setting | Value |
|---|---|
| Title | High Risk Port Exposure |
| Trigger | Number of Results > 0 |
| Schedule | Hourly, 15 min past the hour |
| Action | Add to Triggered Alerts |
| Severity | High |

*(Full config screenshots in [report/report.md](report/report.md#part-1--alert))*

## Dashboard Panel

**Query**:
```spl
index=main sourcetype=<sourcetype> event_type=port_probe
| lookup port_risk_lookup dst_port OUTPUT risk_label
| stats count by risk_label
```

Added as a third panel — **Port Scan Risk Assessment** (column chart)
— on the existing `SOC Detection Dashboard`, alongside Ransomware
Event Breakdown and Failed Logon Sources.

*(Full panel build screenshots in [report/report.md](report/report.md#part-2--dashboard-panel))*

## Data Loss Note
The original port scan dataset had disappeared from the local Splunk
index by this session — recovered by regenerating and re-ingesting the
log. Confirms local/free-tier Splunk lab data isn't guaranteed
persistent; treated as regenerable, not permanent. Full diagnosis in
[report/report.md](report/report.md#note-on-data-loss--recovery).

## What I Learned
- Alerts and dashboards are the "last mile" of detection engineering — a working query isn't operational until scheduled or visualized for ongoing use.
- Always verify data exists (`stats count by sourcetype`) before debugging a query that returns nothing — could be a query bug or missing data entirely.
- Extend existing dashboards with complementary panels rather than building disconnected one-off views.

## Next Steps
- [ ] Reduce alert schedule interval on a production-tier Splunk instance
- [ ] Add a 4th dashboard panel combining all three detection domains into one overall risk view

## Related Projects
- 🔗 [Port Scan Detection](https://github.com/CyberBros435/soc-port-scan-detection-splunk)
- 🔗 [Lookup Enrichment](https://github.com/CyberBros435/soc-lookup-enrichment-splunk)
- 🔗 [Unified Monitoring Dashboard](https://github.com/CyberBros435/soc-unified-monitoring-dashboard)
- 🔗 [Ransomware Detection & Live Alert](https://github.com/CyberBros435/ransomware-siem-analysis)

---

**Author**: Mudasir Zia | [GitHub](https://github.com/CyberBros435) | [LinkedIn](https://linkedin.com/in/mudasir-zia-a535243b5)
