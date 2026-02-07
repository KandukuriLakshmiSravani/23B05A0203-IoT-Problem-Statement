# 23B05A0203-IoT-Problem-Statement
IoT- based Monitoring Solution for Agriculture
                           Problem Statement: Smart Monitoring System
1.	System Architecture:
<img width="915" height="410" alt="image" src="https://github.com/user-attachments/assets/c30ed6d5-fb1d-49c7-80f8-f7f41929d1ce" />

 
2. Sensor selection
1)	Soil-moisture sensor options
1. Capacitive soil-moisture sensors (recommended choice for most projects)
•	Pros: not easily corroded, reasonably linear output, low cost (except VH400 which is mid-range), easy to read with ADC on ESP32/Arduino, good long-term stability.
•	Cons: still needs calibration per soil type; lower absolute accuracy than lab instruments.
2. Resistive / cheap probe sensors
•	Pros: very cheap, simple.
•	Cons: electrodes corrode quickly, unreliable over time, noisy readings — not recommended for long-term installations.
3. Dielectric / Frequency-domain sensors (EC-5, ECH2O style)
•	Pros: better accuracy and stability than basic capacitive/resistive probes, good for research/longer deployments.
•	Cons: more expensive and sometimes require dedicated interface electronics.
4. TDR (Time-Domain Reflectometry) probes
•	Pros: highest accuracy and least affected by salinity; industry/research standard.
•	Cons: very expensive and bulky — overkill for most student projects.
2)	Temperature sensor options (for soil/ambient)
1. DS18B20 (digital, waterproof probe) — my top pick
•	Pros: digital 1-wire interface (easy to chain many sensors), waterproof stainless-steel probes available for soil use, good accuracy (~±0.5°C typical), easy to interface with ESP32/Arduino.
•	Cons: slightly larger physical probe than tiny thermistors, needs pull-up resistor on bus.
2. Thermistor (e.g., 10k NTC)
•	Pros: cheap and fast.
•	Cons: requires analog input and calibration/lookup table; less plug-and-play.
3. TMP36 / LM35 (analog temperature ICs)
•	Pros: simple linear voltage output, easy to read with ADC.
•	Cons: need ADC calibration; not waterproof by default (need casing/probe).
4. DHT22 / AM2302 (air temp + humidity)
•	Pros: gives humidity and air temp in one sensor.
•	Cons: not suitable for measuring soil temperature (probe is not waterproof), slower, less accurate than DS18B20 for soil.


<img width="755" height="637" alt="image" src="https://github.com/user-attachments/assets/b4554fe9-046a-40ef-9c96-257f907c6cc1" />


Why this combo? 
•	Reliability: capacitive probes resist corrosion — that means the sensor won’t fail after a few weeks in wet soil like cheap resistive probes do.
•	Cost vs performance: capacitive probes give a good balance inexpensive enough for multiple points in a field but good enough accuracy for irrigation decision logic. VH400 is pricier but gives more consistent, linear readings if you want higher quality.
•	Ease of use: both capacitive sensors and DS18B20 work well with ESP32/Arduino. DS18B20’s digital output avoids analog noise and allows several temperature probes on one wire, which is perfect for monitoring multiple locations without many ADC channels.
•	Field-friendliness: waterproof DS18B20 probes are simple to bury, and capacitive probes are designed for soil insertion. Combined they let your edge unit (ESP32) make reliable threshold decisions locally (e.g., “soil moisture < 30% → turn pump ON”).

3. Data flow explanation
Overall Data Flow
Soil Moisture Sensor + Temperature Sensor → ESP32 (Edge) → Internet (WiFi/GSM) → Cloud → Dashboard & Alerts → Pump Control
The soil moisture and temperature sensors collect field data and send it to the ESP32. The ESP32 processes the data and checks predefined threshold values. If soil moisture falls below the set limit, especially when temperature is high, the system activates the irrigation pump and sends an alert to the cloud. The cloud stores the data, displays it on a dashboard, and notifies the farmer through SMS or app notifications.

1. The soil moisture sensor and temperature sensor continuously collect field data.
Example reading:
•	Dry soil → Low value (e.g., 25%)
•	Wet soil → High value (e.g., 60%)
•	 Example reading: 32°C
Why both sensors?
•	High temperature increases evaporation.
•	Low moisture + high temperature = urgent irrigation need.

2. The data is sent to the ESP32 (edge device), where it is processed and converted into meaningful values (moisture % and temperature °C).
3. The ESP32 checks the threshold condition:
•	If soil moisture is below the set limit (e.g., 30%) and temperature is high,
→ It turns ON the irrigation pump and generates an alert.
•	For example:
IF soil_moisture < 30%
   AND temperature > 25°C
THEN
   Turn ON irrigation pump
   Generate alert
ELSE
   Keep pump OFF
4. The processed data is sent via WiFi/GSM to the cloud server.
5. The cloud stores the data, updates the dashboard, and sends notifications (SMS/App alert) to the farmer.

4. Alert logic:

IF soil_moisture < 30% AND temperature > 25°C
    Turn ON pump
    Send Alert: "Dry Soil & High Temperature - Irrigation Activated"

ELSE IF soil_moisture < 30% AND temperature ≤ 25°C
    Send Warning Alert: "Low Moisture - Irrigation May Be Required"

ELSE IF soil_moisture > 45%
    Turn OFF pump



5. Sample Dashboard:
------------------------------------------------
  SMART IRRIGATION DASHBOARD
-------------------------------------------------
Soil Moisture: 28%        [ LOW ]
Temperature: 33°C         [ HIGH ]
Pump Status: ON           [ ACTIVE ]
-------------------------------------------------
Moisture Trend Graph (24h)
Temperature Trend Graph
-------------------------------------------------
Alerts:
• Low Soil Moisture - Irrigation Started
-------------------------------------------------

Sample Cloud JSON Data (Dashboard Input)
{
  "device_id": "FIELD_01",
  "soil_moisture_percent": 28,
  "temperature_celsius": 33,
  "pump_status": "ON",
  "alert": "Low Moisture - Irrigation Activated",
  "timestamp": "2026-02-05T14:30:00Z"
}
