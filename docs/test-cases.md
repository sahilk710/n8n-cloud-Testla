# Test Cases - Financial Data Analysis Multi-Agent System

## Test Environment
- **Platform:** n8n Cloud
- **Model:** OpenAI GPT-5-mini (n8n free credits)
- **Date:** April 2, 2026

---

## Test Case 1: Basic Stock Analysis

| Field | Value |
|-------|-------|
| **ID** | TC-001 |
| **Query** | "Analyze Tesla stock performance and give me insights" |
| **Expected Behavior** | Controller delegates to Data Collector (fetches TSLA data), then Analyst (computes metrics). Returns formatted report. |
| **Expected Output** | Stock Analysis Report for TSLA with Summary, Key Metrics, Insights, Risk Factors, Disclaimer |
| **Agents Triggered** | Controller -> Data Collector -> Analyst |
| **Tools Used** | HTTP Request (Alpha Vantage), SerpAPI, Metric Calculator, Health Scorer |
| **Pass Criteria** | Report contains current price, at least 3 insights, disclaimer present |

---

## Test Case 2: Different Stock Ticker

| Field | Value |
|-------|-------|
| **ID** | TC-002 |
| **Query** | "What's the current state of AAPL stock?" |
| **Expected Behavior** | System correctly identifies AAPL ticker and fetches Apple stock data |
| **Expected Output** | Report for AAPL with current price and analysis |
| **Agents Triggered** | Controller -> Data Collector -> Analyst |
| **Tools Used** | HTTP Request, SerpAPI, Metric Calculator |
| **Pass Criteria** | Report is about Apple (AAPL), not another stock |

---

## Test Case 3: Risk-Focused Query

| Field | Value |
|-------|-------|
| **ID** | TC-003 |
| **Query** | "Give me a risk analysis for MSFT" |
| **Expected Behavior** | System emphasizes risk factors in the analysis |
| **Expected Output** | Report with prominent Risk Factors section, volatility assessment |
| **Agents Triggered** | Controller -> Data Collector -> Analyst |
| **Tools Used** | HTTP Request, SerpAPI, Metric Calculator, Health Scorer |
| **Pass Criteria** | Risk Factors section contains at least 2 specific risks |

---

## Test Case 4: Health Score Request

| Field | Value |
|-------|-------|
| **ID** | TC-004 |
| **Query** | "Calculate the financial health score for GOOGL" |
| **Expected Behavior** | System computes and returns the composite health score with breakdown |
| **Expected Output** | Health Score (0-100) with momentum, volatility, volume, sentiment breakdown |
| **Agents Triggered** | Controller -> Data Collector -> Analyst |
| **Tools Used** | HTTP Request, SerpAPI, Metric Calculator, Health Scorer |
| **Pass Criteria** | Health score is present with a rating (Strong/Moderate/Weak) |

---

## Test Case 5: Follow-Up Question (Memory Test)

| Field | Value |
|-------|-------|
| **ID** | TC-005 |
| **Query** | First: "Analyze TSLA stock" → Then: "How does that compare to last week?" |
| **Expected Behavior** | Memory retains context from first query; second response references TSLA without re-asking |
| **Expected Output** | Follow-up response references TSLA and prior analysis |
| **Agents Triggered** | Controller (uses memory) -> may re-query Data Collector |
| **Tools Used** | Simple Memory |
| **Pass Criteria** | System remembers TSLA context and provides relevant follow-up |

---

## Test Case 6: Invalid Ticker

| Field | Value |
|-------|-------|
| **ID** | TC-006 |
| **Query** | "Analyze XYZABC123 stock" |
| **Expected Behavior** | System handles invalid ticker gracefully |
| **Expected Output** | Error message telling user the ticker was not found or data unavailable |
| **Agents Triggered** | Controller -> Data Collector (fails gracefully) |
| **Tools Used** | HTTP Request (returns error) |
| **Pass Criteria** | No crash; user receives a helpful error message |

---

## Test Case 7: News-Heavy Query

| Field | Value |
|-------|-------|
| **ID** | TC-007 |
| **Query** | "What's the latest news affecting NVDA stock and how might it impact the price?" |
| **Expected Behavior** | Data Collector prioritizes news search; Analyst interprets news impact |
| **Expected Output** | Report with recent news summaries and their potential price impact |
| **Agents Triggered** | Controller -> Data Collector -> Analyst |
| **Tools Used** | SerpAPI (primary), HTTP Request, Metric Calculator |
| **Pass Criteria** | Report includes at least 2 recent news items with analysis |

---

## Test Case 8: Multiple Metrics Request

| Field | Value |
|-------|-------|
| **ID** | TC-008 |
| **Query** | "Show me the moving averages, volatility, and volume trend for AMZN" |
| **Expected Behavior** | Analyst focuses on specific requested metrics |
| **Expected Output** | Report with 7-day MA, 30-day MA, volatility %, volume trend |
| **Agents Triggered** | Controller -> Data Collector -> Analyst |
| **Tools Used** | HTTP Request, Metric Calculator |
| **Pass Criteria** | All three requested metrics (MA, volatility, volume) are present |

---

## Test Case 9: Vague Query Handling

| Field | Value |
|-------|-------|
| **ID** | TC-009 |
| **Query** | "Is it a good time to buy stocks?" |
| **Expected Behavior** | Controller asks for clarification or provides general guidance with disclaimer |
| **Expected Output** | Response asking for a specific ticker, or general market overview |
| **Agents Triggered** | Controller (may not delegate to sub-agents) |
| **Tools Used** | None or SerpAPI for general market news |
| **Pass Criteria** | System doesn't crash; provides helpful response |

---

## Test Case 10: Rapid Successive Queries (Performance Test)

| Field | Value |
|-------|-------|
| **ID** | TC-010 |
| **Query** | "Analyze META stock" immediately followed by "Now do NFLX" |
| **Expected Behavior** | System handles both queries; memory maintains context |
| **Expected Output** | Two separate reports, one for META and one for NFLX |
| **Agents Triggered** | Full pipeline for each query |
| **Tools Used** | All tools |
| **Pass Criteria** | Both reports are accurate and complete; no data mixing between stocks |

---

## Evaluation Metrics

| Metric | How to Measure |
|--------|---------------|
| **Accuracy** | Compare reported prices with actual market data |
| **Response Time** | Measured in n8n execution logs (target: < 30 seconds) |
| **Completeness** | Does the report contain all expected sections? |
| **Error Handling** | Does the system handle failures gracefully? |
| **Memory** | Does follow-up context work correctly? |
| **Tool Usage** | Are all tools being called as expected? (Check logs) |

## Results Template

| Test ID | Status | Response Time | Notes |
|---------|--------|---------------|-------|
| TC-001 | PASS | 8.1s | Full report generated for TSLA |
| TC-002 | | | |
| TC-003 | | | |
| TC-004 | | | |
| TC-005 | | | |
| TC-006 | | | |
| TC-007 | | | |
| TC-008 | | | |
| TC-009 | | | |
| TC-010 | | | |
