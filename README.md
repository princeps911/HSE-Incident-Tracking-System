# HSE-Incident-Tracking-System
# 🛠️ Technical Documentation: HSE Metrics & DAX Logic

The core of this engine is built on **IOGP (International Association of Oil & Gas Producers)** standards. The primary challenge in HSE modeling is ensuring that "Total Hours Worked" (the denominator) does not double-count when slicing by incident types or root causes.

### 1. Data Architecture: The "Exposure" Solution
To maintain mathematical accuracy, I utilized a **Star Schema** with two distinct fact tables:
* **`Fact_Incidents`**: Granular level data (one row per incident).
* **`Fact_Exposure`**: Monthly aggregate data (total man-hours per field).

**Why this matters:** This prevents "Denominator Inflation." In a standard join, if a field has 10,000 work hours and 5 incidents, a simple sum would incorrectly show 50,000 hours. My model keeps these separate to ensure TRIR and LTIFR remain accurate.

---

### 2. Key DAX Measures

#### **Total Recordable Incident Rate (TRIR)**
Calculated based on the industry standard of 200,000 man-hours (representing 100 employees working 40 hours/week for 50 weeks).

```dax
TRIR = 
VAR TotalRecordables = 
    CALCULATE(
        COUNTROWS('Fact_Incidents'), 
        'Fact_Incidents'[Type] IN {"LTI", "Recordable", "Fatality", "Restricted"}
    )
VAR ExposureHours = SUM('Fact_Exposure'[Total_Hours_Worked])
RETURN
    DIVIDE(TotalRecordables * 200000, ExposureHours, 0)
```

#### **Lost Time Injury Frequency Rate (LTIFR)**
Calculated per 1,000,000 man-hours to align with international IOGP benchmarking.
```dax
LTIFR = 
VAR LTICount = 
    CALCULATE(
        COUNTROWS('Fact_Incidents'), 
        'Fact_Incidents'[Type] = "LTI"
    )
VAR ExposureHours = SUM('Fact_Exposure'[Total_Hours_Worked])
RETURN
    DIVIDE(LTICount * 1000000, ExposureHours, 0)
```
### 3. Leading vs. Lagging Indicators
While TRIR and LTIFR are Lagging Indicators (measuring past harm), I implemented the Near Miss Rate as a Leading Indicator.

Insight: In a proactive safety culture, a rising Near Miss Rate combined with a falling TRIR suggests improved reporting transparency and hazard identification before injuries occur.

### 4. Risk Analysis Logic
Pareto (80/20): Used RANKX to identify the vital few root causes driving the majority of incidents.

Key Influencers: Utilized Power BI’s ML engine to correlate employee experience years with incident severity, identifying a 3x higher risk profile for Short Service Employees (SSE).


