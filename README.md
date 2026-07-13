# 🚨 DisasterSense — Multi-Hazard AI Risk Alert System

An n8n workflow that monitors earthquakes, severe weather, and wildfires across multiple locations in real time, calculates a weighted risk score, and sends alerts via Telegram + Gmail when risk crosses a threshold — with full history logged to Google Sheets.

## Features

- **Hourly automated monitoring** via Schedule Trigger
- **Multi-location support** — reads target cities/coordinates from a Google Sheet config
- **Multi-hazard data fusion**:
  - 🌍 Earthquakes — [USGS Earthquake API](https://earthquake.usgs.gov/fdsnws/event/1/)
  - 🌦️ Weather — [OpenWeatherMap API](https://openweathermap.org/api)
  - 🔥 Wildfires — [NASA FIRMS API](https://firms.modaps.eosdis.nasa.gov/api/)
- **Weighted risk scoring** (0–10 scale): `0.4×Earthquake + 0.35×Weather + 0.25×Fire`
- **Instant alerts** via Telegram and Gmail when risk score ≥ threshold
- **Full audit trail** — every check (alert sent or not) logged to a Google Sheets "History" tab

## How It Works

1. **Schedule Trigger** fires every hour
2. **Get row(s) in sheet** pulls monitored locations (lat/long/radius) from the `Locations` tab
3. **Loop Over Items** processes each location one at a time
4. In parallel, fetches:
   - Recent earthquakes near the location (USGS)
   - Current weather conditions (OpenWeatherMap)
   - Active fire detections in a bounding box around the location (NASA FIRMS)
5. **Calculate Risk Score** (Code node) scores each hazard 0–10 and computes a weighted composite score
6. **Check Risk Threshold** routes to alerts if score ≥ 1.5 (tune this to your needs — see Configuration)
7. High-risk events trigger:
   - A formatted Telegram alert
   - An HTML email via Gmail
   - A logged row in the Google Sheets `History` tab (`alert_sent: yes`)
8. Low-risk checks are still logged (`alert_sent: no`) for full traceability

## Setup

### 1. Google Sheet
Create a Google Sheet with two tabs:

**`Locations`**
| city | lat | long | radius_km |
|------|-----|------|-----------|
| Delhi | 28.6139 | 77.2090 | 100 |

**`History`**
| timestamps | city | hazard_type | severity | risk_score | alert_sent |
|---|---|---|---|---|---|

### 2. API Keys
Get free API keys from:
- [OpenWeatherMap](https://openweathermap.org/api) — free tier
- [NASA FIRMS](https://firms.modaps.eosdis.nasa.gov/api/) — free, instant signup

### 3. Import Workflow
1. Import `disastersense_workflow.json` into n8n
2. Replace these placeholders with your own values:
   - `YOUR_GOOGLE_SHEET_ID` → your Google Sheet ID (in both Google Sheets nodes)
   - `YOUR_NASA_FIRMS_API_KEY` → your NASA FIRMS key (in "Fetch Fire Data" node URL)
   - `YOUR_OPENWEATHERMAP_API_KEY` → your OpenWeatherMap key (in "Fetch Weather Alerts" node URL)
   - `YOUR_TELEGRAM_CHAT_ID` → your Telegram chat ID (in "Send a text message" node)
   - `YOUR_EMAIL@example.com` → recipient email(s), comma-separated (in "Send a message" node)
3. Set up credentials in n8n for:
   - Google Sheets OAuth2
   - Telegram Bot API
   - Gmail OAuth2

### 4. Tune the risk threshold
The `Check Risk Threshold` node currently fires alerts at `risk_score >= 1.5`. Adjust this in the IF node condition based on how sensitive you want alerting to be.

## Tech Stack

n8n · JavaScript (Code nodes) · Google Sheets API · Telegram Bot API · Gmail API · USGS Earthquake API · OpenWeatherMap API · NASA FIRMS API

## ⚠️ Security Note

This repo ships with all credentials and API keys stripped out and replaced with placeholders. **Never commit real API keys, sheet IDs, chat IDs, or credential IDs to a public repository.** If you're forking this, generate your own free API keys rather than reusing anyone else's.

---
*DisasterSense • AI-Powered Multi-Hazard Early Warning System*
