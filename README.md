# Wave Power Generation Short-Term Forecasting Service — Technical Manual & Service Specification v1.0

**Author:** INESC TEC
**Version:** 1.0
**Last updated:** 03 Feb 2026

---

## 1. Business Context & Definitions

The Wave Power Generation Short-Term Forecasting Service provides short-term forecasts (0–48 hours ahead) of electrical power generation for wave energy plants. The service is designed to support operational planning, grid balancing and market-facing decision making by energy stakeholders such as plant operators/asset owners, power traders, BRPs/BSPs, TSOs and DSOs.

Accurate and reliable wave power forecasts are critical to reduce imbalance exposure, improve operational scheduling and increase confidence in trading and dispatch decisions under highly variable marine conditions. The service is intended for continuous operational use, with defined availability, accuracy and auditability requirements.

### Key Terms

- **Plant / Site:** A wave energy plant identified by a unique plant identifier. A plant may include multiple units and sensors whose measurements are aggregated at plant level.
- **Timestamp:** Date/time at which measurements or forecasts apply, expressed in UTC and aligned to a fixed sampling interval (hourly).
- **Generated Power / Energy:** Measured electrical power (kW/MW) and/or energy (kWh/MWh) output from the wave plant, at unit and/or plant level.
- **Marine & Meteorological Inputs:** Environmental variables that influence generation, e.g. significant wave height, wave period/frequency, wave direction, wind speed/direction, sea state indicators and relevant weather forecasts.
- **Forecast horizon:** The time window ahead for which generation is predicted (0–48h).
- **Resolution / Granularity:** Forecast time step and spatial level (hourly; per plant / per unit where applicable).
- **Issue time:** The UTC time when the forecast is generated; only data available up to issue time may be used.
- **Backtesting:** Rolling evaluation over historical time ranges to emulate real-time usage.

### 1.1 Stakeholder Context

This service addresses the need of energy stakeholders to anticipate short-term wave generation under unpredictable wave conditions, enabling safer and more efficient plant operation, more robust grid balancing and improved trading and risk management. Forecast outputs should be traceable, time-aligned and suitable for operational workflows.

---

## 2. Problem Statement

The objective is to deliver a reliable, production-grade service that predicts wave power generation for a plant over the next 0–48 hours. The service must operate continuously and provide forecasts that are consistent, traceable and operationally usable across changing sea states and weather conditions.

The service shall expose an authenticated interface for requesting forecasts per plant (and optionally per unit) and time range, returning time-aligned outputs at an hourly resolution. Forecasts must include a clear issue time, sufficient metadata for auditability and reproducibility and remain within declared physical/operational limits when relevant.

The forecasting approach may combine historical plant measurements, real-time sensor telemetry and exogenous drivers such as marine/weather forecasts, provided inputs are available at the time the forecast is issued. The service will be assessed as an operational capability, with emphasis on forecast quality, stability over time and service reliability (availability and response performance).

---

## 3. Data Description

### 3.1 Data Dictionaries of Wave Power Service

#### 3.1.1 Data Dictionary – Weather Station

| Variable | Variable name | Type | Measurement unit | Description | Allowed values / Examples |
|---|---|---|---|---|---|
| Timestamp | `timestamp` | Timestamp | DD/MM/YYYY HH:MM | timestamp field | 21-03-2025 00:15 |
| Battery Voltage Meteohelix | `battery_voltage_meteohelix` | Numeric | V | Battery supply voltage measured by the MeteoHelix at the given timestamp | 3.6 |
| Battery Voltage Meteowind | `battery_voltage_meteowind` | Numeric | V | Battery supply voltage measured by the MeteoWind at the given timestamp | 4.0 |
| Wind Speed Average | `wind_speed_avg` | Numeric | m/s | Wind speed | 2.22 |
| Wind Direction Average | `wind_direction_avg` | Numeric | ° | Average wind direction (0-360) | 90.65 |
| Alarm | `alarm_flag` | Numeric | - | 1 if at least one alarm condition is active, else 0 | 1 |
| Humidity | `humidity` | Numeric | % | Relative humidity | 40 |
| Air Pressure | `air_pressure_avg` | Numeric | Pa | Average air pressure | 101413.64 |
| Irradiance | `irradiance` | Numeric | W/m² | Solar irradiance (global horizontal) | 1.22 |
| Temperature | `temperature` | Numeric | °C | Ambient air temperature | 18.98 |

#### 3.1.2 Data Dictionary – Instrumental Buoy

| Variable | Variable name | Type | Measurement unit | Description | Allowed values / Examples |
|---|---|---|---|---|---|
| Timestamp | `timestamp` | Timestamp | DD/MM/YYYY HH:MM | Measurement timestamp (UTC) | 21-03-2025 00:15 |
| Buoy ID | `buoy_id` | String | - | Unique buoy identifier | BUOY_01 |
| Current direction | `current_direction` | Numeric | ° | Current direction. Range: 0–360°. Resolution: 0.01°. Accuracy: ±5° to ±7.5° | 245.32 |
| Sensor tilt | `sensor_tilt` | Numeric | ° | Tilt of the current sensor. Range: 0–35°. Resolution: 0.01°. Accuracy: ±1.5° | 2.15 |
| Current speed | `current_speed` | Numeric | cm/s | Current speed. Range: 0–100 cm/s. Resolution: 0.1 mm/s (=0.01 cm/s). Accuracy: ±0.15 mm/s (=±0.015 cm/s) | 12.34 |
| Water temperature | `water_temperature` | Numeric | °C | Temperature. Range: -5–40 °C. Resolution: 0.01 °C. Accuracy: ±0.1 °C | 16.27 |
| SPL (Broadband) | `spl_broadband` | Numeric | dB re 1 µPa | Sound Pressure Level (broadband) | 85.3 |
| SPL (Deci-decade bands) | `spl_deci_decade_bands` | Numeric | dB re 1 µPa | SPL per deci-decade frequency bands | 70.1 |
| Spectrogram | `spectrogram` | Array / Matrix | dB re 1 µPa (or relative) | Spectrogram representation computed from acoustic signal | [T×F matrix] |
| Cetacean detections | `detection_cetaceans` | Boolean | - | Automatic detections of some cetacean species | True / False |
| Vessel detections | `detection_vessels` | Boolean | - | Automatic detections of vessels | True / False |
| Waves (TBD) | `ocean_waves_tbd` | Text | - | Wave-related parameters | TBD |
| Wind (TBD) | `ocean_wind_tbd` | Text | - | Wind-related parameters | TBD |
| Barometric pressure (TBD) | `ocean_barometric_pressure_tbd` | Text | - | Barometric pressure | TBD |

#### 3.1.3 Data Dictionary – Sentinel

| Variable | Variable name | Type | Measurement unit | Description | Allowed values / Examples |
|---|---|---|---|---|---|
| Geopotential height at 500 mb | `HGT500` | number | m (geopotential meters) | Geopotential height at 500 hPa. | ~5000–6000 m |
| Geopotential height at 850 mb | `HGT850` | number | m (geopotential meters) | Geopotential height at 850 hPa. | ~1300–1700 m |
| Geopotential height (model level 1) | `HGTlev1` | number | m | Geopotential height at model level 1. | model dependent |
| Geopotential height (model level 2) | `HGTlev2` | number | m | Geopotential height at model level 2. | model dependent |
| Geopotential height (model level 3) | `HGTlev3` | number | m | Geopotential height at model level 3. | model dependent |
| Temperature at 500 mb | `T500` | number | K | Air temperature at 500 hPa. | ~230–270 K |
| Temperature at 850 mb | `T850` | number | K | Air temperature at 850 hPa. | ~250–310 K |
| CAPE | `cape` | number | J/kg | Convective available potential energy. | 0–5000 J/kg (e.g. 1200) |
| CIN | `cin` | number | J/kg | Convective inhibition (usually negative). | ≤0 J/kg (e.g. -50) |
| Cloud cover (high) | `cfh` | number | fraction (0–1) | Cloud cover at high levels. | 0.0 clear … 1.0 overcast |
| Cloud cover (low+mid) | `cfl` | number | fraction (0–1) | Cloud cover at low and mid levels. | 0.0–1.0 |
| Cloud cover (mid) | `cft` | number | fraction (0–1) | Cloud cover at mid levels. | 0.0–1.0 |
| Accumulated precipitation | `prec` | number | mm | Total accumulated rainfall between model outputs. | 0–100+ mm per step |
| Convective precipitation | `conv_prec` | number | mm | Total accumulated convective rainfall between model outputs. | 0–100 mm per step |
| Accumulated snowfall (large-scale) | `snow_prec` | number | mm (liquid equivalent) | Total accumulated large-scale snowfall between model outputs. | 0–100 mm per step |
| Water equivalent of accumulated snow depth | `weasd` | number | mm (kg/m²) | Water equivalent of accumulated snow on the ground. | 0–500 mm |
| Snow level | `snowlevel` | number | m a.s.l. | Approximate altitude of the snow level. | 0–3000 m+ depending on situation |
| Downwelling shortwave flux (surface) | `swflx` | number | W/m² | Surface downwelling shortwave radiation. | 0–1000 W/m² (e.g. 742) |
| Downwelling longwave flux (surface) | `lwflx` | number | W/m² | Surface downwelling longwave radiation. | 100–450 W/m² |
| Latent heat flux (downward) | `lhflx` | number | W/m² | Surface downward latent heat flux. | ~-50–400 W/m² |
| Sensible heat flux (downward) | `shflx` | number | W/m² | Surface downward sensible heat flux. | ~-50–400 W/m² |
| Air temperature at 2 m | `temp` | number | K | Air temperature at 2 meters above ground. | ~250–320 K (e.g. 290.7) |
| Relative humidity at 2 m | `rh` | number | fraction (0–1) | Relative humidity at 2 meters. | 0.0–1.0 (e.g. 0.82) |
| Zonal wind at 10 m | `u` | number | m/s | East–west wind component at 10 m (positive eastward). | -50–50 m/s |
| Meridional wind at 10 m | `v` | number | m/s | North–south wind component at 10 m (positive northward). | -50–50 m/s |
| Wind speed at 10 m | `mod` | number | m/s | Wind speed magnitude at 10 m. | 0–50 m/s |
| Wind direction at 10 m | `dir` | number | degrees (0–360) | Meteorological wind direction at 10 m. | 0/360=N, 90=E, 180=S, 270=W |
| Wind gust | `wind_gust` | number | m/s | Modelled near-surface wind gust. | 0–50 m/s |
| Sea surface temperature | `sst` | number | K | Sea surface (skin) temperature. | 270–310 K |
| Visibility | `visibility` | number | m | Horizontal visibility at the surface. | 0–50,000 m |
| PBL height | `pbl_height` | number | m AGL | Planetary boundary layer height above ground. | 0–4000 m |
| Topography | `topo` | number | m a.s.l. | Terrain elevation (orography). | 0–4000+ m |
| Land/water mask | `lwm` | integer (categorical) | 0/1 | Indicates land (1) or water (0). | 0=water, 1=land |
| Land use / vegetation type | `land_use` | integer (categorical) | class id | Discrete land cover class used by the source model. | integer codes (e.g. 1,2,3…) |
| Zonal wind (model level 1) | `ulev1` | number | m/s | Zonal wind component at model level 1. | -100–100 m/s |
| Zonal wind (model level 2) | `ulev2` | number | m/s | Zonal wind component at model level 2. | -100–100 m/s |
| Zonal wind (model level 3) | `ulev3` | number | m/s | Zonal wind component at model level 3. | -100–100 m/s |
| Meridional wind (model level 1) | `vlev1` | number | m/s | Meridional wind component at model level 1. | -100–100 m/s |
| Meridional wind (model level 2) | `vlev2` | number | m/s | Meridional wind component at model level 2. | -100–100 m/s |
| Meridional wind (model level 3) | `vlev3` | number | m/s | Meridional wind component at model level 3. | -100–100 m/s |
| Mean sea level pressure | `mslp` | number | hPa | Sea-level reduced pressure. | 950–1050 hPa |

#### 3.1.4 Data Dictionary – SCADA

| Variable | Variable name | Type | Measurement unit | Description | Allowed values / Examples |
|---|---|---|---|---|---|
| Timestamp | `timestamp_utc` | Timestamp | YYYY-MM-DD HH:MM | Logging time of the measurement | 18/3/2025 14:00 |
| average active power delivered | `turbine_activepower_avg` | Numeric | kW | Average active power delivered to the grid in the logging | 0-6000 (ex: 2450) |
| Average rotor rotational speed | `rotor_rpm_avg` | Numeric | RPM | Average rotor rotational speed in the logging interval | 0-20 (ex: 14.9) |

> **Status:** SCADA data dictionary is under consolidation with the data provider. The table above defines the minimum expected fields required to support training, inference and evaluation.

#### 3.1.5 Data Dictionary – Wave Energy Converter

| Variable | Variable name | Type | Measurement unit | Description | Allowed values / Examples |
|---|---|---|---|---|---|
| Timestamp | `timestamp_utc` | Timestamp | YYYY-MM-DD HH:MM | Measurement timestamp (UTC) | 2026-02-01 12:00 |
| Plant ID | `plant_id` | Text | - | Unique plant identifier | WAVE_PLANT_01 |
| WEC ID | `wec_id` | Text | - | Device identifier | WEC_03 |
| Operational mode | `op_mode` | Text | - | Device operating mode | Normal/Derated/Safe |
| PTO status (optional) | `pto_status` | Text | - | Power take-off status | On/Off/Fault |
| Control setpoint (optional) | `control_setpoint` | Numeric | - | Control setpoint (if available) | TBD |

> **Status:** WEC device-level telemetry is expected to be available but the variable list is not final. The table above specifies the minimum expected fields to support operational regime identification and feature engineering.

---

## 4. Analytics, Scope & Update Frequency

- **Temporal scope:** Forecasts cover 0–48 hours ahead of the forecast issue time (UTC), with outputs aligned to a fixed hourly time grid.
- **Update frequency:** Forecasts are generated on demand via API requests.
- **Output format:** For each plant, the service returns a structured forecast package including:
  - **Issue time & validity window:** Issue timestamp (UTC) and start/end forecast horizon (0–48h).
  - **Time-indexed site forecast:** Plant-level (and optionally unit-level) time series of forecasted generation at hourly resolution.
  - **Traceability metadata:** Forecast/run ID, model version and configuration identifiers to support logging and reproducibility.

---

## 5. Evaluation Protocols & Metrics

The purpose of the evaluation is to verify that the service operates reliably and delivers consistent, accurate and operationally usable wave power forecasts. Evaluation focuses on compliance with agreed metrics, service availability and delivery of specified forecast outputs per request.

### 5.1 Data Usage & Forecasting Protocol

- The service shall generate forecasts on demand for horizons up to 0–48 hours ahead of the stated issue time (UTC), aligned to the agreed hourly step.
- Only information available at or before the issue time may be used (e.g. telemetry up to that time and marine/weather forecast inputs available at that time).
- Forecast outputs shall be time-aligned and include a unique identifier and minimal traceability fields (forecast/run ID, model version).
- The service shall behave consistently under repeated requests for the same issue time and configuration.

### 5.2 Data Gaps and Exceptions

- Time periods with missing, invalid or flagged measurements in actual generation shall be excluded from forecast-quality scoring.
- If a plant has insufficient recent data to produce a forecast, the service shall return a clear structured error (or documented fallback behaviour). Such events shall be counted under service reliability KPIs.
- Input data gaps (e.g. missing marine forecasts) shall be logged and reported; the service shall document how it degrades gracefully (e.g. baseline model / persistence / last-known-good inputs), if applicable.

### 5.3 Service Evaluation Metrics & KPIs

The service shall be evaluated using the following quantitative metrics:

- **MAE:** Mean Absolute Error of forecasted wave generation versus measured generation, aggregated across forecast steps (and across plants if multiple plants are evaluated).
- **RMSE:** Root Mean Squared Error aggregated across forecast steps.
- **Response time:** API response latency for forecast requests under nominal load.
- **Availability:** Percentage of forecast requests that return a valid response within the agreed service levels.

---

## 6. Deliverables & Submissions

The selected provider shall deliver three reports, aligned with the lifecycle of the service, together with the required technical specifications and deployment artefacts.

### 6.1 Deliverable Reports

1. **Pre-Service Deliverable – Service Design & Setup Report**
   Submitted prior to service start, this report shall describe the proposed analytical approach, forecasting methodology, baseline definition, data requirements, system architecture, security measures and integration plan. It shall also define the operational schedule and procedures for service execution.

2. **Intermediate Deliverable – Interim Performance & Operations Report**
   Submitted at an agreed midpoint of the service period, this report shall summarise service operation to date, data coverage, preliminary analytical results and performance against the defined metrics and KPIs. Any issues, adaptations or refinements to the methodology shall be documented.

3. **Final Deliverable – Final Evaluation & Recommendations Report**
   Submitted at the end of the service period, this report shall present final performance results and site-level insights. The report shall also include guidance for future service continuation or scaling.

### 6.2 Technical Specifications & Submissions

- **Service Interface Documentation:** Full documentation of APIs, data formats, authentication and access controls.
- **Deployment Artefacts:** The provider shall specify the deployment approach. Where containerisation is used, a Dockerfile or equivalent container specification shall be delivered to support reproducible deployment. Alternative deployment models shall be clearly documented.
- **Configuration, Versioning & Handover:** Documentation of configuration parameters, model and data versioning and operational handover procedures.
- **Security & Data Protection Documentation:** Description of data handling, access control and compliance with applicable data protection requirements.
