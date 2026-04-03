# Technical Documentation - Financial Data Analysis Multi-Agent System

## 1. System Architecture

### 1.1 Overview

This system implements a multi-agent architecture on n8n Cloud where a Controller Agent orchestrates two specialized sub-agents to perform end-to-end financial data analysis. The architecture follows a hub-and-spoke pattern where the Controller acts as the central hub, delegating tasks to specialized agents.

### 1.2 Architecture Diagram

```
+------------------+     +----------------------+
|   Chat Trigger   |---->|  Controller Agent    |
| (User Interface) |     |  (Orchestrator)      |
+------------------+     |                      |
                          |  - OpenAI GPT Model  |
                          |  - Simple Memory     |
                          |  - System Prompt     |
                          +----------+-----------+
                                     |
                          +----------+-----------+
                          |                      |
                   +------v------+    +----------v---------+
                   | Data        |    | Analyst            |
                   | Collector   |    | Agent              |
                   | Agent       |    |                    |
                   |             |    |  - OpenAI GPT      |
                   | - OpenAI    |    |  - Metric          |
                   |   GPT       |    |    Calculator      |
                   | - HTTP Req  |    |  - Financial       |
                   |   (Alpha    |    |    Health Scorer   |
                   |   Vantage)  |    |    (CUSTOM)        |
                   | - SerpAPI   |    |                    |
                   +-------------+    +--------------------+
```

### 1.3 Data Flow

1. User sends a natural language query via Chat Trigger
2. Controller Agent parses the query and identifies the stock ticker and analysis type
3. Controller calls Data Collector Agent with the ticker and data requirements
4. Data Collector fetches price data (HTTP Request) and news (SerpAPI)
5. Data Collector returns structured data to Controller
6. Controller passes raw data to Analyst Agent
7. Analyst runs Metric Calculator and Financial Health Scorer
8. Analyst returns analysis to Controller
9. Controller formats the final report and sends it to the user

## 2. Agent Specifications

### 2.1 Controller Agent (Orchestrator)

| Property | Value |
|----------|-------|
| Node Type | AI Agent |
| Model | OpenAI GPT-5-mini |
| Temperature | 0.3 (default) |
| Memory | Simple Memory (Window Buffer) |
| Role | Parse queries, delegate to sub-agents, compile final report |

**System Prompt Summary:**
- Identifies stock tickers, time periods, and analysis types from user queries
- Delegates data fetching to Data Collector and analysis to Analyst
- Implements feedback loop: re-queries Data Collector if data is insufficient
- Formats output as a structured report with Summary, Key Metrics, Health Score, Insights, Risk Factors, and Disclaimer

### 2.2 Data Collector Agent (Sub-Agent #1)

| Property | Value |
|----------|-------|
| Node Type | AI Agent Tool |
| Model | OpenAI GPT-5-mini |
| Temperature | 0.2 |
| Tools | HTTP Request, SerpAPI |
| Role | Fetch stock prices and financial news |

**Capabilities:**
- Fetches daily stock prices (OHLCV) from Alpha Vantage API
- Searches for recent financial news and market sentiment via SerpAPI
- Returns structured JSON with price data, news, and API status
- Handles API failures gracefully with partial data returns

### 2.3 Analyst Agent (Sub-Agent #2)

| Property | Value |
|----------|-------|
| Node Type | AI Agent Tool |
| Model | OpenAI GPT-5-mini |
| Temperature | 0.2 |
| Tools | Metric Calculator (Code), Financial Health Scorer (Code - Custom) |
| Role | Process data, calculate metrics, generate insights |

**Capabilities:**
- Computes financial metrics (moving averages, % changes, volatility)
- Generates composite health scores using custom algorithm
- Synthesizes data into actionable insights
- Identifies risk factors based on data patterns

## 3. Tool Specifications

### 3.1 HTTP Request Tool (Alpha Vantage API)

| Property | Value |
|----------|-------|
| Method | GET |
| Base URL | https://www.alphavantage.co/query |
| Function | TIME_SERIES_DAILY |
| Output Size | compact (last 100 data points) |

**Query Parameters:**
- `function`: TIME_SERIES_DAILY
- `symbol`: Stock ticker (e.g., TSLA)
- `outputsize`: compact
- `apikey`: Alpha Vantage API key

**Returns:** JSON with daily OHLCV (Open, High, Low, Close, Volume) data for the last ~100 trading days.

### 3.2 SerpAPI (Web Search Tool)

| Property | Value |
|----------|-------|
| Type | SerpAPI Tool |
| Purpose | Financial news and sentiment search |
| Rate Limit | 100 searches/month (free tier) |

**Usage:** The Data Collector Agent formulates search queries like "TSLA Tesla stock news analysis" to find recent articles, analyst opinions, and market sentiment.

### 3.3 Metric Calculator (Code Tool)

| Property | Value |
|----------|-------|
| Language | JavaScript |
| Input | JSON with `prices` array of {date, close, volume} objects |
| Output | JSON with computed metrics |

**Computed Metrics:**
- **7-Day Moving Average** - Short-term trend indicator
- **30-Day Moving Average** - Medium-term trend indicator
- **7-Day % Change** - Short-term momentum
- **30-Day % Change** - Medium-term momentum
- **Volatility** - Standard deviation of daily returns (as %)
- **Volatility Level** - Categorical: Low (<1.5%), Medium (1.5-3%), High (>3%)
- **Volume Trend** - Compares recent 7-day avg volume to prior 7-day avg

### 3.4 Financial Health Scorer (Custom Tool)

| Property | Value |
|----------|-------|
| Language | JavaScript |
| Type | Custom-built composite scoring algorithm |
| Input | JSON with pct_change_30d, volatility_pct, volume_trend, sentiment |
| Output | Composite score (0-100), breakdown, rating, interpretation |

**Scoring Algorithm:**

The Financial Health Scorer is the custom tool built for this project. It calculates a weighted composite score from four factors:

#### Factor 1: Price Momentum (Weight: 30%)
```
Input: 30-day percentage change
Mapping: -20% or worse = 0, +20% or better = 100
Formula: ((pctChange30d + 20) / 40) * 100
```

#### Factor 2: Volatility (Weight: 25%)
```
Input: Daily return volatility (%)
Mapping: 0% = 100 (very stable), 5%+ = 0 (very volatile)
Formula: ((5 - volatilityPct) / 5) * 100
Rationale: Lower volatility = higher score (stability rewarded)
```

#### Factor 3: Volume Trend (Weight: 20%)
```
Input: volume_trend (increasing/stable/decreasing) + price direction
Logic:
  - Increasing volume + positive price = 85 (strong buying)
  - Increasing volume + negative price = 40 (selling pressure)
  - Stable volume = 60 (neutral)
  - Decreasing volume + positive price = 50 (weak rally)
  - Decreasing volume + negative price = 30 (bearish)
```

#### Factor 4: News Sentiment (Weight: 25%)
```
Input: sentiment (positive/neutral/mixed/negative)
Mapping:
  - Positive = 85
  - Neutral = 55
  - Mixed = 45
  - Negative = 15
```

#### Composite Score
```
Score = (Momentum * 0.30) + (Volatility * 0.25) + (Volume * 0.20) + (Sentiment * 0.25)
```

#### Ratings
| Score Range | Rating | Interpretation |
|-------------|--------|----------------|
| 70-100 | Strong | Positive momentum and favorable conditions |
| 45-69 | Moderate | Mixed signals, monitor closely |
| 0-44 | Weak | Weakness across multiple factors, exercise caution |

#### Input Validation
- Checks that `pct_change_30d` and `volatility_pct` are valid numbers
- Validates `volume_trend` against allowed values: increasing, decreasing, stable, unknown, insufficient_data
- Validates `sentiment` against allowed values: positive, negative, neutral, mixed
- Returns descriptive error messages for invalid inputs

#### Error Handling
- Entire function wrapped in try/catch
- Returns structured error object on failure: `{ error: "message", score: null }`

## 4. Memory System

| Component | Type | Scope |
|-----------|------|-------|
| Simple Memory | Window Buffer Memory | Controller Agent |

The Controller Agent uses n8n's Simple Memory (Window Buffer) to maintain conversation context across turns. This allows:
- Follow-up questions about previously analyzed stocks
- Comparative analysis referencing earlier results
- Contextual responses without re-fetching data

Sub-agents do not have their own memory — they receive full context from the Controller via the prompt passthrough.

## 5. Feedback Loop

The feedback loop is implemented in the Controller Agent's system prompt:

1. Controller sends data request to Data Collector
2. Data Collector returns data (may be partial if APIs fail)
3. Controller evaluates completeness of the data
4. If insufficient data, Controller re-queries Data Collector with adjusted parameters
5. Controller sends complete data to Analyst
6. Analyst returns analysis
7. Controller compiles and formats the final report

## 6. Error Handling Strategy

| Layer | Handling |
|-------|----------|
| API Failure (Alpha Vantage) | Data Collector notes failure in api_status, attempts to get data from news search |
| API Failure (SerpAPI) | Data Collector returns partial data without news |
| Invalid Ticker | Data Collector returns error message, Controller informs user |
| Insufficient Data | Metric Calculator returns null for metrics that can't be computed |
| Code Errors | try/catch in both Code Tools, returns structured error JSON |
| Model Fallback | Sub-agents can be configured with fallback models |

## 7. Challenges and Limitations

### Challenges Faced
1. **API Rate Limits** - Alpha Vantage free tier limits to 25 requests/day, requiring efficient query planning
2. **Data Freshness** - Stock data may be delayed; real-time data requires premium APIs
3. **Sub-Agent Communication** - Ensuring structured data passes correctly between agents required careful prompt engineering
4. **Token Usage** - Multi-agent calls consume more tokens; optimized with gpt-5-mini model

### Limitations
1. Limited to daily price data (no intraday)
2. Health Score relies on available data — may be incomplete if APIs fail
3. Sentiment analysis is based on news search results, not dedicated NLP
4. Free tier API limits restrict the number of analyses per day

## 8. Future Improvements

- Add more data sources (Yahoo Finance, Finnhub)
- Implement dedicated sentiment analysis using NLP models
- Add technical indicators (RSI, MACD, Bollinger Bands)
- Create scheduled analysis reports via n8n cron triggers
- Add portfolio tracking across multiple stocks
