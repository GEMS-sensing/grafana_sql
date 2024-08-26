# DRAFT README

## SQL Queries for Error Flagging

This document provides an overview of the SQL queries used for creating the error flagging dashboard. The queries are organized by parameter and are used to extract, flag, and transform the data for display.

### Sections

1. **Soil Temperature**
   - **Device**: Acclima TDR 315H
   - **Data**: Extracts soil temperature data, flags values out of range, and handles missing values.
     - **Error Handling and Processing**:
       - **Value Out of Range Flag**: Flags data points where the soil temperature is below `$TDR315H_ST_MIN` or above `$TDR315H_ST_MAX`.
       - **Missing Value Flag**: Flags timestamps where soil temperature data is missing for `Acclima Soil.Temperature.1.1`, `Acclima Soil.Temperature.1.2`, or `Acclima Soil.Temperature.1.3`.
       - **Standard Deviation Warning Flag**: Flags data points where the soil temperature deviates significantly from the mean, beyond `($stddev)` standard deviations.
   - **Daily Operations Flag**: Flags days with missing operations data by comparing recorded dates against the expected date range.

2. **Soil VWC (Volumetric Water Content)**
   - **Device**: Acclima TDR 315H
   - **Data**: Extracts soil VWC data, flags values out of range, and handles missing values.
     - **Error Handling and Processing**:
       - **Value Out of Range Flag**: Flags data points where the soil VWC is below `$TDR315H_VWC_MIN` or above `$TDR315H_VWC_MAX`.
       - **Missing Value Flag**: Flags timestamps where soil VWC data is missing for `Acclima Soil.VWC.1.1`, `Acclima Soil.VWC.1.2`, or `Acclima Soil.VWC.1.3`.
       - **Standard Deviation Warning Flag**: Flags data points where the soil VWC deviates significantly from the mean, beyond `($stddev)` standard deviations.
   - **Daily Operations Flag**: Flags days with missing operations data.

3. **Soil Bulk EC (Electrical Conductivity)**
   - **Device**: Acclima TDR 315H
   - **Data**: Extracts soil bulk EC data, flags values out of range, and handles missing values.
     - **Error Handling and Processing**:
       - **Value Out of Range Flag**: Flags data points where the soil bulk EC is below `$TDR_BEC_Min` or above `$TDR_BEC_Max`.
       - **Missing Value Flag**: Flags timestamps where soil bulk EC data is missing for `Acclima Soil.EC_BULK.1.1`, `Acclima Soil.EC_BULK.1.2`, or `Acclima Soil.EC_BULK.1.3`.
       - **Standard Deviation Warning Flag**: Flags data points where the soil bulk EC deviates significantly from the mean, beyond `($stddev)` standard deviations.

4. **Gaseous Oxygen**
   - **Device**: Apogee SO-421
   - **Data**: Extracts and flags gaseous oxygen data.
     - **Error Handling and Processing**:
       - **Value Out of Range Flag**: Flags data points where the oxygen value is below `$Apogee_SO421_MIN` or above `$Apogee_SO421_MAX`.
       - **Missing Value Flag**: Flags timestamps where gaseous oxygen data is missing.
       - **Standard Deviation Warning Flag**: Flags data points where the oxygen value deviates significantly from the mean, beyond `($stddev)` standard deviations.

5. **CO2**
   - **Device**: Hedorah CO2 Sensor
   - **Data**: Extracts and flags CO2 data.
     - **Error Handling and Processing**:
       - **Value Out of Range Flag**: Flags data points where the CO2 value is below `$hedorah_CO2_min` or above `$hedorah_CO2_max`.
       - **Missing Value Flag**: Flags timestamps where CO2 data is missing.
       - **Standard Deviation Warning Flag**: Flags data points where the CO2 value deviates significantly from the mean, beyond `($stddev)` standard deviations.

6. **Raw Data (within current time range)**
   - **Data**: Extracts raw sensor data for bulk download table.

### Database Parameters and Display Mapping

| Database Parameter                     | Displayed As                       | Definition                                     | Value Out of Range Flag Logic | Missing Value Flag Logic | Standard Deviation Warning Flag Logic |
|----------------------------------------|------------------------------------|------------------------------------------------|-------------------------------|--------------------------|---------------------------------------|
| Acclima Soil.Temperature.1.1           | Soil Temperature 1                 | Soil temperature sensor 1 measurement in °F    | `value < $TDR315H_ST_MIN OR value > $TDR315H_ST_MAX` | `temp1_observation IS NULL` | `abs(value - _avg) > ($stddev::numeric) * _stddev` |
| Acclima Soil.Temperature.1.2           | Soil Temperature 2                 | Soil temperature sensor 2 measurement in °F    | `value < $TDR315H_ST_MIN OR value > $TDR315H_ST_MAX` | `temp2_observation IS NULL` | `abs(value - _avg) > ($stddev::numeric) * _stddev` |
| Acclima Soil.VWC.1.1                   | Soil VWC 1                         | Volumetric water content sensor 1 measurement  | `value < $TDR315H_VWC_MIN OR value > $TDR315H_VWC_MAX` | `vwc1_observation IS NULL` | `abs(value - _avg) > ($stddev::numeric) * _stddev` |
| Acclima Soil.VWC.1.2                   | Soil VWC 2                         | Volumetric water content sensor 2 measurement  | `value < $TDR315H_VWC_MIN OR value > $TDR315H_VWC_MAX` | `vwc2_observation IS NULL` | `abs(value - _avg) > ($stddev::numeric) * _stddev` |
| Acclima Soil.EC_BULK.1.1               | Soil Bulk EC 1                     | Bulk electrical conductivity sensor 1 measurement | `value < $TDR_BEC_Min OR value > $TDR_BEC_Max` | `bec1_observation IS NULL` | `abs(value - _avg) > ($stddev::numeric) * _stddev` |
| Acclima Soil.EC_BULK.1.2               | Soil Bulk EC 2                     | Bulk electrical conductivity sensor 2 measurement | `value < $TDR_BEC_Min OR value > $TDR_BEC_Max` | `bec2_observation IS NULL` | `abs(value - _avg) > ($stddev::numeric) * _stddev` |
| Apogee O2.Oxygen_%.1.4                 | Gaseous Oxygen                     | Gaseous oxygen sensor measurement              | `value < $Apogee_SO421_MIN OR value > $Apogee_SO421_MAX` | `apo2_observation IS NULL` | `abs(value - _avg) > ($stddev::numeric) * _stddev` |
| Apogee O2.Temperature.1.4              | Gaseous Oxygen Temperature         | Temperature measurement from the oxygen sensor | `value < $Apogee_SO421temp_min OR value > $Apogee_SO421temp_max` | `apo2temp_observation IS NULL` | `abs(value - _avg) > ($stddev::numeric) * _stddev` |

### Explanation of Flag Calculations

- **Value Out of Range Flag**: Flags data points where the value is outside the acceptable range.
  - Example: `CASE WHEN (value < $TDR315H_ST_MIN OR value > $TDR315H_ST_MAX) THEN 1 ELSE 0 END AS value, 'Value Out of Range Flag' AS metric`
  
- **Missing Value Flag**: Flags timestamps where expected data is missing.
  - Example: `CASE WHEN (temp1_observation IS NULL OR temp2_observation IS NULL) THEN 1 ELSE 0 END AS value, 'Missing Value Flag' AS metric`

- **Standard Deviation Warning Flag**: Flags data points where the value deviates significantly from the mean, beyond a specified number of standard deviations.
  - Example: `CASE WHEN (abs(value - _avg) > ('$stddev'::numeric) * _stddev) THEN 1 ELSE 0 END AS value, 'Standard Deviation Warning Flag' AS metric`

### Additional Queries

- **Daily Operations Flag**: Flags days with missing operations data.
  - Example: `CASE WHEN (operating_day IS NULL) THEN 1 ELSE 0 END AS "Daily Operations Flag"`

- **Map - Transform Label to Field**: Extracts location data for visualization on a map.
  - Example: `SELECT time, node_id, display_name, metric, value FROM (SELECT ROW_NUMBER() OVER (PARTITION BY node_id, measure ORDER BY time DESC) r, time, value, measure as "metric", node_id, display_name FROM $project_key.data WHERE node_id = '$node' AND measure IN ('Longitude', 'Latitude')) x WHERE x.r <= 1 ORDER BY x.time ASC`

