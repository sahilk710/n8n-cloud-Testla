# Test Results

## Test Execution Log

**Date:** April 2, 2026
**Platform:** n8n Cloud
**Model:** OpenAI GPT-5-mini (n8n free credits)

---

### TC-001: Basic Stock Analysis (TSLA)

| Field | Result |
|-------|--------|
| **Status** | PASS |
| **Query** | "Analyze Tesla stock performance and give me insights" |
| **Response Time** | 8.148 seconds |
| **Tokens Used** | ~11,873 |
| **Agents Triggered** | Controller -> Data Collector -> Analyst |
| **Tools Fired** | HTTP Request, SerpAPI, OpenAI Chat Models |

**Output Summary:**
- Generated Stock Analysis Report for TSLA
- Current Price: $360.59 (closing price April 2, 2026)
- Identified 5.4% decline from previous day
- Volume analysis: 82.5M shares (increased)
- Linked decline to disappointing Q1 delivery numbers
- Risk factors identified: selling pressure, market sensitivity
- Disclaimer included

**Screenshot:** (attach screenshot from n8n)

---

### TC-002 through TC-010

*(Run each test case and fill in results here)*

| Test ID | Query | Status | Response Time | Key Observations |
|---------|-------|--------|---------------|------------------|
| TC-001 | Analyze Tesla stock | PASS | 8.1s | Full report generated |
| TC-002 | AAPL stock state | | | |
| TC-003 | MSFT risk analysis | | | |
| TC-004 | GOOGL health score | | | |
| TC-005 | Follow-up question | | | |
| TC-006 | Invalid ticker | | | |
| TC-007 | NVDA news impact | | | |
| TC-008 | AMZN metrics | | | |
| TC-009 | Vague query | | | |
| TC-010 | META then NFLX | | | |

---

## Performance Summary

| Metric | Result |
|--------|--------|
| Average Response Time | ~8s (TC-001) |
| Success Rate | 1/1 (more tests pending) |
| Avg Token Usage | ~11,873 per query |
| All Agents Triggered | Yes |
| Memory Working | Pending (TC-005) |
| Error Handling | Pending (TC-006) |
