#  Healthcare Data Analysis: Billing Audit & Clinical Risk Assessment (SQL)

##  Project Overview
As a healthcare professional with 11 years of clinical experience and a final-year Software Engineering student, I conducted an exploratory data analysis and financial audit of a healthcare dataset (55,500 patient records) using **SQLite** and **DBeaver**.

The objective of this project is to validate data quality, uncover accounting anomalies, evaluate hospital bed-days (ALOS), and detect clinical discrepancies in medication prescriptions.

---

##  SQL Queries & Key Analytical Insights

### 1. Data Quality & Patient Distribution Check
```sql
-- Check total patient count per diagnosis to verify dataset balance
SELECT "Medical Condition", 
       COUNT(*) AS Total_Patients
FROM healthcare_dataset
GROUP BY "Medical Condition"
ORDER BY Total_Patients DESC;
```
* **Insight:** Patient distribution is nearly uniform across all diagnoses (ranging from **9,185** Asthma cases to **9,308** Arthritis cases). This confirms the synthetic nature of the dataset, shifting the analytical focus toward billing validation and clinical logic checks.

---

### 2. Financial Audit: Identifying Negative Billing Anomalies
```sql
-- Detect erroneous negative billing records and group losses by insurance provider
SELECT "Insurance Provider", 
       COUNT(*) AS Error_Records, 
       ROUND(SUM("Billing Amount"), 2) AS Total_Negative_Sum
FROM healthcare_dataset
WHERE "Billing Amount" < 0
GROUP BY "Insurance Provider"
ORDER BY Error_Records DESC;
```
* **Insight:** Identified **108 erroneous records** with negative billing amounts totaling **-$58,097**. **Aetna** (27 errors, -$18,000.85) and **Cigna** (24 errors, -$12,361.97) accounted for the highest volume of discrepancies.
* **Recommendation:** Enforce a database-level validation constraint (`CHECK ("Billing Amount" > 0)`) in the Electronic Health Record (EHR) system to block negative billing entries upon patient discharge.

---

### 3. Clinical Efficiency: Length of Stay (ALOS) by Admission Type
```sql
-- Calculate average hospital stay in bed-days and average cost by admission urgency
SELECT "Medical Condition",
       "Admission Type",
       COUNT(*) AS Total_Cases,
       ROUND(AVG(JULIANDAY("Discharge Date") - JULIANDAY("Date of Admission")), 1) AS Avg_Bed_Days,
       ROUND(AVG("Billing Amount"), 0) AS Avg_Bill_USD
FROM healthcare_dataset
GROUP BY "Medical Condition", "Admission Type"
ORDER BY "Medical Condition", Avg_Bed_Days DESC;
```
* **Insight:** Across all conditions and admission types, the average hospital stay remains standardized (**15.2 to 16.0 bed-days**) with an average bill of **~$25,000**. In a real clinical environment, acute emergency cases require vastly different resource allocation than elective admissions.

---

### 4. Clinical Risk Audit: High-Risk Discharges with Basic Medications
```sql
-- Flag severe cases discharged with abnormal lab results on basic pain relievers
SELECT "Medical Condition",
       "Medication",
       COUNT(*) AS Risky_Discharges,
       ROUND(AVG("Billing Amount"), 2) AS Avg_Bill_USD
FROM healthcare_dataset
WHERE "Test Results" = 'Abnormal'
  AND "Medication" IN ('Paracetamol', 'Ibuprofen', 'Aspirin')
GROUP BY "Medical Condition", "Medication"
ORDER BY Risky_Discharges DESC;
```
* **Insight:** The query flagged **644 Oncology (Cancer) patients** treated with Ibuprofen (average bill **$25,234**) and **632 Oncology patients** treated with Paracetamol (average bill **$24,552**) who were discharged with abnormal lab results. This highlights a major gap between high inpatient billing costs and documented clinical protocols.

---

##  Data Limitations & Proposed Database Architecture
To optimize hospital workflows by shifting **Elective (planned)** admissions to off-peak periods without overcrowding emergency departments, calculating the true **Bed Occupancy Rate** is essential.

* **Current Limitation:** The existing flat table lacks reference data on hospital department capacities (total available beds per specialized unit).
* **Architectural Solution:** Introduce a normalized relational table `Department_Capacity` (`Hospital_ID`, `Department_Name`, `Total_Beds`) linked via `JOIN` to monitor real-time bed utilization and prevent department bottlenecks.
