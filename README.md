# HealthCare: Service Incentive & Efficiency Monitor
![Banner](images/ops_dashboard_banner.png)

## Project Overview
This project analyzes a **$1.67M annual bonus spend** for a global healthcare equipment provider. The goal was to identify why high incentive payouts were not correlating with improved **First Time Fix Rates** technicians hired by the company.

**Tools Used:** Power BI, DAX, Power Query.
**Key Achievement:** Identified a "Tenure Trap" inefficiency and proposed a model to save **$300k annually** while improving asset uptime.

---

## Business Context
**The Challenge:**
The Service Operations division faced stagnant reliability metrics (85% Fix Rate) despite increasing bonus budgets. Leadership suspected that the current "Time-Based" incentive model was rewarding seniority rather than technical performance.

---
**Key Business Questions:**
* Does paying more for Seniority yield better Technical Performance?
* At what stage in the machine lifecycle do failure rates spike?
* Which asset class (MRI, CT, X-Ray) drives the majority of downtime hours?

---

## Tech Stack & Methodology
* **Data Modeling:** Star Schema connecting `Fact_Tickets` (12,000 rows) to `Dim_Technicians`, `Dim_Machines`, and `Dim_Calendar`.

  ![Star Schema Data Model](images/data_model_ops.png)
  *(Snapshot of the Entity Relationship Diagram)*
  
* **Advanced DAX:** Used `SELECTEDVALUE` and disconnected tables to create a **"What-If" Scenario Slider**, allowing stakeholders to forecast budget impacts dynamically.
* **Visualization:** Custom "Dip & Recovery" charts to track asset lifecycle vulnerability.

---

## Key Insights & Recommendations

### 1. The "Tenure Trap"
**Finding:** 15-Year Master Techs maintained an **81% Fix Rate**, identical to 2-Year Junior Techs despite commanding significantly higher compensation packages.
* **Impact:** The organization was paying a premium for tenure without receiving efficiency gains from the senior employees.
* **Action:** Pivot to **Outcome-Based Pay**. Cap tenure multipliers at 1.2x and reallocate savings to training employees; specifically the junior employees that are showing great potential at being good assets to the organization. Training them on how to fix MRI which is the company's equipment that had the highest break down would improve efficiency and service delivery to our clients.

### 2. The Mid-Lifecycle Crisis
**Finding:** Age analysis exposed a specific vulnerability at **Year 6**, where Fix Rate drops from 87% to 83%.

  ![Dip and Recovery of First Time Fix](images/machine_age_chart.png)
  *(Snapshot of the Dip in year 6 based on the machine age)*
  
* **Root Cause:** Original factory parts wearing out simultaneously.
* **Action:** Instead of blaming the technicians. I recommend that the company institutes a mandatory **Year-5 Preventative Maintenance Overhaul** in view of preventing the great damage that leads to low first time fix in year 6.

### 3. Critical Asset Risk
**Finding:** **MRI Scanners** were identified as the primary driver of downtime amongst all the machines offered by the organization. Account for the highest volume of downtime hours.
* **Action:** Targeted technical training funded by the $300k bonus savings can help to produce better engineers who can adequately fix the MRI scanners and improve efficiency for customers, and ultimately lower the downtime.

---

## Dynamic Budget Forecasting
I implemented a parameter table to allow leadership to test new bonus structures in real-time. This would help to forecast what happens when we increases the total bonus in a specific year or when we increase it for specific reasons (For instance for targeted roles) where we need highly qualified technicians (engineers). This would help to evaluate the cost-benefit impact of the reduction or increment.
  
  ![Projected Bonus Cost](images/scenario_slider_demo.gif)
*(Snapshot: Real-time calculation of budget impact using the slider)*

**Parameter Table Generation:**
```dax
Bonus Adjustment = GENERATESERIES(-0.5, 0.5, 0.05)
```

**Forecast Measure:**
```dax
Projected Bonus Cost = 
VAR Adjustment = 'Bonus Adjustment'[Bonus Adjustment Value]
RETURN
[Total Bonus Paid] * (1 + Adjustment)
```
---

## Row-Level Security (RLS) Filter
I implemented a Row-Level Security (RLS) Filter on the Dim_Technicians table to restrict data access. A specific role was created for the Warsaw Regional Manager.

![Row Level Security](images/rls_demo.jpg)

![Row Level Security Output](images/rls_demo_output.jpg)
*(Snapshot: Row Level Security filter and output to the report)*

---
## 📁 Project Structure
```text
Healthcare-Service-Operations-Analytics/
│
├── dashboard/
│   └── Service_Incentive_Monitor.pbix
│
├── calculations/
│   └── key_dax_measures.md
│
├── docs/
│   └── Service_Incentive_Monitor.pdf
│
├── images/
│   ├── ops_dashboard_banner.png
│   ├── scenario_slider_demo.gif
│   ├── machine_age_chart.png
│   └── data_model_ops.png
│
├── data/
│   └── Service_Incentive_Monitor.xlsx
│
└── README.md
``` 

---
### Disclaimer
This project is a personal portfolio work designed for educational and demonstration purposes only. The dataset used is **synthetic and simulated**, created to mirror real-world operational challenges. It does not represent actual confidential data, financial records, or employee information from any healthcare company.

All logos and brand names are trademarks of their respective owners. This project is not affiliated with, endorsed by, or connected to any company in any way.