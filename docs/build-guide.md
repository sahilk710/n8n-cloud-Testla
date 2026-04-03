# Step-by-Step Build Guide

This guide walks through building the Financial Data Analysis Multi-Agent System from scratch in n8n Cloud.

## Prerequisites

- n8n Cloud account (free trial at app.n8n.cloud)
- OpenAI API key (or use n8n's free credits)
- Alpha Vantage API key (free at alphavantage.co)
- SerpAPI key (free at serpapi.com)

## Phase 1: Create Workflow & Chat Trigger

1. Log into n8n Cloud
2. Click "Add workflow" and name it: `Financial Data Analysis - Multi-Agent System`
3. Add a **Chat Trigger** node (search "Chat" in the node panel)
4. This is the user interface entry point

## Phase 2: Build the Controller Agent

1. Click "+" on the Chat Trigger node
2. Add an **AI Agent** node — rename it to `Controller Agent`
3. Add **OpenAI Chat Model** under Chat Model (use gpt-5-mini or gpt-4o-mini)
4. Add **Simple Memory** under Memory
5. Add the system prompt via "+ Add Option" > "System Message" (see technical docs for full prompt)

## Phase 3: Build Data Collector Sub-Agent

1. Click "+" under Tool on the Controller Agent
2. Add **AI Agent Tool** — this becomes a sub-agent
3. Set Name to `data_collector`
4. Add description, system message, and prompt (`{{ $json.chatInput }}`)
5. Add **OpenAI Chat Model** under its Chat Model
6. Add **HTTP Request Tool** under its Tool slot:
   - Method: GET
   - URL: `https://www.alphavantage.co/query`
   - Query params: function=TIME_SERIES_DAILY, symbol={ticker}, outputsize=compact, apikey=YOUR_KEY
7. Add **SerpAPI** under its Tool slot:
   - Configure with your SerpAPI credential

## Phase 4: Build Analyst Sub-Agent

1. Click "+" under Tool on the Controller Agent (adds alongside data_collector)
2. Add another **AI Agent Tool**
3. Set description and system message for analysis
4. Add **OpenAI Chat Model** under its Chat Model
5. Add **Code Tool** (Metric Calculator) — JavaScript code for moving averages, volatility, etc.
6. Add **Code Tool** (Financial Health Scorer) — Custom scoring algorithm

## Phase 5: Test

1. Click "Open Chat" at the bottom
2. Type: "Analyze Tesla stock performance and give me insights"
3. Check the Logs panel to verify all agents and tools are executing
4. Verify the output contains all expected sections

## Phase 6: Deploy

1. Click "Publish" in the top right to activate the workflow
2. Share the chat URL for public access (if Chat Trigger is set to public)
