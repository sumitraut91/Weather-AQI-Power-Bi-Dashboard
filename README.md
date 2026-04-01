# ⛅ Weather Forecast & Air Quality Index Dashboard


<p align="center">
  <img src="https://raw.githubusercontent.com/sumitraut91/Weather-AQI-Power-Bi-Dashboard/main/Weather%20Forecast%20and%20AQI%20Dashboard.PNG" alt="Weather Forecast and AQI Dashboard" width="100%"/>
</p> 

<p align="center">
  <img src="https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black" />
  <img src="https://img.shields.io/badge/WeatherAPI.com-0078D4?style=for-the-badge&logo=icloud&logoColor=white" />
  <img src="https://img.shields.io/badge/DAX-blueviolet?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Power%20Query-green?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge" />
</p>

---

## 📌 About the Project

This is an interactive **Power BI dashboard** that displays **real-time weather conditions and a 7-day forecast** along with a detailed **Air Quality Index (AQI)** breakdown for **8 major Indian cities**. Live JSON data is fetched from the [WeatherAPI](https://www.weatherapi.com/) and transformed into an intuitive, health-aware visual experience.

The dashboard is designed for urban residents, event planners, and public health stakeholders who need weather and air quality data in one place — not scattered across multiple apps and sources.

---

## 🗂️ Table of Contents

- [Dashboard Overview](#-dashboard-overview)
- [Problem Statement](#-problem-statement)
- [Cities Covered](#-cities-covered)
- [Data Source](#-data-source)
- [ETL & Data Preprocessing](#-etl--data-preprocessing)
- [Data Model](#-data-model)
- [DAX Measures & Calculated Columns](#-dax-measures--calculated-columns)
- [Key Findings](#-key-findings)
- [Tools Used](#-tools-used)
- [Project Files](#-project-files)
- [How to Run Locally](#-how-to-run-locally)

---

## 📊 Dashboard Overview

The dashboard is built as a **single-page report** with the following sections:

| Section | What It Shows |
|---|---|
| 🌡️ **Current Weather Panel** | Live temperature, humidity, wind speed, visibility, pressure, UV index, precipitation |
| 📅 **7-Day Forecast Strip** | Day-wise temperature with condition icons across the top |
| 📈 **Forecast Line Chart** | Temperature trend visualized across all 7 days |
| 🌫️ **Air Quality Index** | Gauge + pollutant cards: SO2, PM10, PM2.5, CO, O3, NO2 — color-coded by severity |
| 🌅 **Sunrise & Sunset** | Formatted times with icons for the selected day |
| 🌧️ **Chance of Rain** | Horizontal bar chart showing daily rain probability across 7 days |
| 🎨 **Dynamic Background** | Background image changes automatically with the current weather condition |
| 🏙️ **City Switcher** | Toggle between 8 cities from the left navigation panel |

---

## ❓ Problem Statement

Urban residents and planners often rely on scattered sources for weather and air quality data. There's no single, visual, real-time interface that combines:
- Current conditions and multi-day forecasts
- Granular AQI with actionable health recommendations
- City-level comparison in one view

**This dashboard solves that** — by aggregating live API data, applying health-context DAX logic, and presenting everything in a single, auto-refreshable Power BI report.

---

## 🏙️ Cities Covered

| # | City | State |
|---|---|---|
| 1 | Bengaluru | Karnataka |
| 2 | Dehradun | Uttarakhand |
| 3 | Hyderabad | Telangana |
| 4 | Kolkata | West Bengal |
| 5 | Mumbai | Maharashtra |
| 6 | Nagpur | Maharashtra |
| 7 | Pune | Maharashtra |
| 8 | Noida | Uttar Pradesh |

---

## 📡 Data Source

| Detail | Info |
|---|---|
| **API** | [WeatherAPI.com](https://www.weatherapi.com/) |
| **Endpoint** | `https://api.weatherapi.com/v1/forecast.json` |
| **Parameters** | `city`, `days=7`, `aqi=yes` |
| **Format** | Nested JSON |


**Metrics collected per city:**

- *Weather:* Temperature (°C), Humidity (%), Wind Speed (kph), Pressure (mb), Visibility (km), UV Index, Precipitation (mm), Sunrise & Sunset
- *AQI Pollutants:* PM2.5, PM10, CO, SO2, O3, NO2

---

## 🔧 ETL & Data Preprocessing

All transformations were done inside **Power Query (M Language)**.

### Master Table
- Connected to the WeatherAPI JSON endpoint for each of the 8 cities
- Expanded all nested JSON objects: `location` → `current` → `forecast` → `day / astro / hour / condition / air_quality`
- Enforced correct data types across all columns
- Fixed broken image URLs by replacing `//` with `https://` in condition icon fields
- Combined all 8 city tables into one unified `Master` table using `Table.Combine`

### Daily Forecast Table
- Referenced `Master`
- Removed hourly columns; kept day-level data only
- Deduplicated on `{City, ForecastDate}` — one record per city per day

### Hourly Forecast Table
- Expanded `forecast.forecastday.hour` records
- Split timestamp into separate `Date` and `Hour` columns
- Dropped columns not relevant for hourly analysis

### Current Weather Table
- Retained only the live snapshot per city
- Removed forecast and hourly columns; cleaned nulls


---

## 🗃️ Data Model

The data model in Power BI was designed to connect fact tables (live, daily, hourly) with supporting dimension and helper tables, enabling flexible and dynamic weather analysis.

### Tables Used:

- Current (Fact Table) → Central table for live weather and air quality measures (temperature, humidity, pollutant levels) by city and timestamp.
- Forecast_Day (Fact Table) → Day-level forecasted weather per city, including temperatures, rain chances, and astro data (sunrise/sunset).
- Forecast_Hour (Fact Table) → Hour-level forecasted weather, enabling granular trend analysis and detailed visualizations.
- Master (Fact Table) → Integrates metadata for multi-source refresh and time-point validation.
- Measuress (Helper Table) → A disconnected table created specifically to group and organize all DAX measures.
- City-specific tables (Dimension Tables) → Extend easy multi-location tracking, promoting modularity and separation of city datasets.
  
### Time Intelligence:

Dedicated Date tables connected to timestamps across Current, Forecast_Day, Forecast_Hour, and Data_Refresh.
Enables analysis by day of week, month, custom periods, and precise filtering for historical vs. forecast trends.
Relationship Setup:

One-to-Many link Current with Forecast_Day and Forecast_Hour on city and date/time.
Bi-directional relationship between [Weather_Bg] and [Current] allows real-time condition-driven background changes.
City, day, and hour tables are synchronized to support drill-downs from overview → daily → hourly weather insights.

---

## 📐 DAX Measures & Calculated Columns

### Measures

**AQI Status**
```dax
AQI_Status =
VAR AQI = ROUND(SELECTEDVALUE('Current'[current.air_quality.pm10]), 0)
RETURN
SWITCH(
    TRUE(),
    AQI <= 50,  "Good",
    AQI <= 100, "Moderate",
    AQI <= 150, "Unhealthy For Sensitive",
    AQI <= 200, "Unhealthy",
    AQI <= 300, "Very Unhealthy",
    "Hazardous"
)
```

**AQI Health Suggestion**
```dax
AQI_Suggestion =
VAR AQI = SELECTEDVALUE('Current'[current.air_quality.pm10])
RETURN
SWITCH(
    TRUE(),
    AQI <= 50,  "Air is clean and healthy",
    AQI <= 100, "Acceptable air quality, stay active",
    AQI <= 150, "Sensitive groups should reduce outdoor time",
    AQI <= 200, "Limit prolonged outdoor exertion",
    AQI <= 300, "Avoid outdoor activity if possible",
    "Stay indoors, wear mask if outside"
)
```

**Current Temperature**
```dax
Curr_Temp_C = SUM('Current'[current.temp_c]) & " °C"
```

**Forecast Average Temperature**
```dax
Forecast_Temp_C = ROUND(AVERAGE(Forecast_day[forecast.forecastday.day.avgtemp_c]), 1) & " °C"
```

**Wind Speed**
```dax
Wind_Speed = SUM('Current'[current.wind_kph]) & " Kph"
```

**Last Updated Label**
```dax
Last_Updated_Date_Curr =
"Last Updated, " & FORMAT(FIRSTNONBLANK('Current'[current.last_updated], ""), "dd mmm")
```

**Rain Bar Remainder**
```dax
Left_value_Rain = 100 - SUM(Forecast_day[forecast.forecastday.day.daily_chance_of_rain])
```

---

### Calculated Columns

**Day Name**
```dax
Day_Name = FORMAT(Forecast_day[forecast.forecastday.date], "dd dddd")
```

**Daylight Duration**
```dax
DaylightDuration =
VAR Minutes = DATEDIFF(Forecast_day[Sunrise], Forecast_day[Sunset], MINUTE)
VAR Hours   = INT(Minutes / 60)
VAR Mins    = MOD(Minutes, 60)
RETURN Hours & " hrs " & Mins & " mins"
```

**Formatted Sunrise & Sunset**
```dax
Sunrise = FORMAT(Forecast_day[forecast.forecastday.astro.sunrise], "hh:mm AM/PM")
Sunset  = FORMAT(Forecast_day[forecast.forecastday.astro.sunset],  "hh:mm AM/PM")
```

---

## 🔍 Key Findings

- A **single-page interface** consolidates weather and AQI data that was previously spread across multiple apps and sources
- **Color-coded AQI pollutant cards** (green → yellow → red) allow instant health risk assessment without interpreting raw numbers
- The **7-day rain probability chart** proved particularly useful for outdoor planning — showing near-100% rain probability across multiple days at a glance
- **Daylight duration** — derived from sunrise to sunset — is a useful metric not commonly found in standard weather dashboards
- The **dynamic background** that shifts with live weather conditions (sunny, cloudy, rainy) significantly improves the visual experience and dashboard engagement
- The **modular star schema** design makes it straightforward to add new cities, additional pollutants, or extended forecast windows in the future

---

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| **Power BI Desktop** | Report building, data modelling, DAX, visuals |
| **Power Query (M)** | JSON ingestion, ETL, table transformations |
| **DAX** | KPI measures, dynamic labels, conditional formatting logic |
| **WeatherAPI.com** | Live weather & AQI data via REST API |


---

## 📁 Project Files

```
weather-aqi-dashboard/
│
├── Weather_Forecast_and_AQI_Dashboard.pbix   ← Power BI report file
├── Weather_Forecast_and_AQI_Dashboard.PNG    ← Dashboard screenshot
└── README.md                                 ← Project documentation
```

---


