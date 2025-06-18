### Comprehensive Insurance Portfolio Analysis Report: Profitability, Fraud, and Operational Efficiency  

---

#### **Table of Contents**  
1. [Project Name](#1-project-name)  
2. [Project Background](#2-project-background)  
3. [Project Goals](#3-project-goals)  
4. [Data Collection & Sources](#4-data-collection--sources)  
5. [Formal Data Governance](#5-formal-data-governance)  
6. [Regulatory Reporting](#6-regulatory-reporting)  
7. [Methodology](#7-methodology)  
8. [Data Structure & Initial Checks](#8-data-structure--initial-checks)  
9. [Documenting Issues](#9-documenting-issues)  
10. [Executive Summary](#10-executive-summary)  
11. [Insights Deep Dive](#11-insights-deep-dive)  
12. [Recommendations](#12-recommendations)  
13. [Future Work](#13-future-work)  
14. [Technical Details](#14-technical-details)  
15. [Assumptions & Caveats](#15-assumptions--caveats)  

---

### **1. Project Name**  
**Al-Rayan Insurance: Portfolio Profitability, Fraud Detection & Operational Efficiency Analysis**  
*Analysis of 50,000 policies (2023–2024) to diagnose financial sustainability, fraud patterns, and operational gaps.*  

---

### **2. Project Background**  
Al-Rayan Insurance (established 2010) operates in Saudi Arabia’s auto, health, and home insurance segments. Key metrics (2023–2024):  
- **Portfolio Size**: 50,000 active policies  
- **Annual Premium Revenue**: SAR 525M (Avg: SAR 10,498/policy)  
- **Claims Paid**: SAR 1.25B (Avg: SAR 25,070/claim)  
- **Complaints**: 10,001 (80.16/1,000 policies)  
Business model centers on broker-distributed policies with manual claims processing.  

---

### **3. Project Goals**  
**As a data analyst at Al-Rayan, this analysis aims to:**  
1. Diagnose root causes of **underwriting losses** (Combined Ratio: 288.68%)  
2. Quantify **fraud impact** (4.74% flagged policies) and optimize detection  
3. Identify **operational inefficiencies** (89.6% claims take >7 days to settle)  
4. Provide actionable strategies for **regulatory compliance** (25% policies below solvency threshold)  

**Key Insight Categories:**  
- **Category 1**: Profitability & Risk Ratios  
- **Category 2**: Fraud Detection & Financial Impact  
- **Category 3**: Policy-Type Performance (Home/Auto/Health)  
- **Category 4**: Operational Efficiency & Customer Experience  

---

### **4. Data Collection & Sources**  
- **Primary Source**: Internal policy database (SQL Server)  
- **Tables Used**:  
  - `policy_master` (Policy IDs, types, dates)  
  - `financials` (Premiums, claims, expenses, ratios)  
  - `customer_activity` (Claims, complaints, renewals)  
  - `fraud_flags` (Fraud labels, investigation notes)  
- **External Data**: KSA regulatory solvency benchmarks (SAMA)  
- **Time Frame**: Jan 2023–Mar 2024  

---

### **5. Formal Data Governance**  
**Existing Framework:**  
- **Standardization**: ISO 27001 for data encryption; policy IDs follow `[Type]-[Region]-[Seq]` syntax  
- **Quality Checks**: Daily null checks for `premium` and `claim_amount` fields  
**Improvements Needed:**  
- **Issue**: 100% null values in `persistency_rate` and `policies_renewed`  
- **Action**: Implement validation rules to auto-reject submissions missing renewal metrics  

---

### **6. Regulatory Reporting**  
- **Compliance Focus**: Solvency Ratio (KSA SAMA requires ≥0.85)  
- **Issue**: 25% of policies (12,500) fall below 0.85  
- **Resolution**: Scripts align solvency calculations with SAMA's formula:  

## 7. Methodology

### Machine Learning
- **Fraud detection** using Random Forest classifier  
- Dataset balanced with **SMOTE** (Synthetic Minority Oversampling Technique)

### Statistical Analysis
- Loss ratio volatility measured via **standard deviation** (σ = SAR 14,462)
- **Outlier detection** for claims where:  
  `loss_ratio > 40` (z-score > 3)

### Visual Analytics
- Scatter plots: **loss vs. expense ratios** (fraud vs. valid claims)
- **Time-to-settle histograms** for claims processing efficiency

---

## 8. Data Structure & Initial Checks
**Dataset:** Denormalized insurance portfolio  
**Records:** 50,000 (Jan 2023 - Mar 2024)  
**Columns:** 27  

### Table: `insurance_portfolio`
| Column                  | Description                          | Key Points & Usage Considerations                 |
|-------------------------|--------------------------------------|--------------------------------------------------|
| `Customer_ID`           | Unique policyholder identifier       | Primary key; no duplicates detected             |
| `Policy_Type`           | Insurance product category           | Home policies dominate (33.8% of portfolio)     |
| `Annual_Premium_SAR`    | Yearly premium amount                | Range: 1,000–19,999 SAR; σ = 5,485 SAR          |
| `Claims_This_Year`      | Number of claims filed               | 12.04% of customers file >2 claims              |
| `Total_Claim_Amount_SAR`| Total value of claims paid           | Avg 25,070 SAR (max 50,000 SAR)                 |
| `Operating_Expenses_SAR`| Administrative costs per policy      | Home policies costliest (avg 88.75M SAR)        |
| `Earned_Premiums_SAR`   | Recognized revenue                   | Exceeds written premiums (117.9%)               |
| `Policies_Renewed`      | Count of renewed policies            | **Critical issue:** 100% zero values           |
| `Solvency_Ratio`        | Net assets / Total liabilities       | 25% of values < 0.85                            |
| `Fraud_Flag`            | Binary fraud indicator               | 4.74% prevalence                                |
| `Loss_Ratio_new`        | Recalculated claims-to-premiums ratio| 7% values >40 capped at 10.0                   |
| `Persistency_Rate_new`  | Recalculated policy renewal rate     | **Data gap:** 29.7% nulls                      |

### Critical Data Quality Notes
1. **Zero-Renewal Anomaly**  
   - `Policies_Renewed` and `Persistency_Rate` contain ONLY zeros  
   - *Impact:* Invalidates retention/churn analysis  
   - *Resolution:* Validate ETL pipeline integration  

2. **Recalculated Ratio Discrepancies**  
   - `Loss_Ratio_diff` shows ≥15% variance in 8.2% of policies  
   - *Action:* Use `_new` columns for financial analyses  

3. **Solvency Data Integrity**  
   - `Solvency_Margin_SAR` and `Net_Assets_SAR` show logical consistency (r²=0.92)  
## 🔍 8. Initial Validation
- **Completeness**: Only `Persistency_Rate_new` has nulls (29.7%)  
- **Accuracy**: 0.1% extreme values excluded  
- **Consistency**: `Policy_Type` values standardized  

---

## 9. Documenting Issues
| Table               | Column             | Issue                      | Magnitude | Solvable? | Resolution                      |
|---------------------|--------------------|----------------------------|-----------|-----------|---------------------------------|
| `customer_activity` | `renewals`         | 100% null values           | High      | Yes       | Integrate renewal tracking API  |
| `customer_activity` | `persistency_rate` | All values = 0.0           | High      | Yes       | Map to policy lifecycle dates   |
| `financials`        | `loss_ratio`       | 7% records > 40            | Medium    | Yes       | Cap at logical max (10.0)       |
| `policy_master`     | `region`           | 15% records "UNK"          | Low       | No        | Exclude from geo-analysis       |

---

## 10. Executive Summary
**For Finance Director: Three Critical Insights**  
1. 🚨 **Profitability Crisis**: Valid policies lose SAR 2.75 for every SAR 1 earned  
2. 🔒 **Fraud Efficiency**: Detected fraud claims cost 84% less than valid claims  
3. ⚠️ **Operational Risk**: 12,500 policies breach solvency thresholds  

## 🔍 11. Insights Deep Dive

### 📊 Category 1: Profitability & Risk Ratios
- **50% of policies** exceed combined ratio > 1.0 (unprofitable)
- **Expense ratio spikes** to 9.69 for 5% of policies
- **25% of policies** have solvency ratios < 0.85
- **Loss ratio volatility** (σ = 14,462) signals pricing issues

### 🕵️ Category 2: Fraud Detection & Financial Impact
- Fraud claims cost **84% less** than valid claims
- Fraudulent home policies have **higher expense ratios**
- Auto fraud claims cost **SAR 19.78M** (highest absolute loss)
- Fraud complaints **6% lower** than average

### 🏠 Category 3: Policy-Type Performance
- Home insurance has highest **combined ratio (4.13)**
- **48.25% claim frequency** (1 claim/2 policies)
- Home policies generate **26% more complaints**
- Fraudulent home claims take **longest to settle (30.6 days)**

### ⚙️ Category 4: Operational Efficiency
- **89.6% claims** take >7 days to settle
- Day 26 settlements correlate with **SAR 13,706 premiums**
- **6,019 customers** file >2 claims/year
- Zero policy renewals indicate **data capture failure**

## 📋 12. Recommendations

| Team           | Action                                      |
|----------------|---------------------------------------------|
| **Underwriting** | Increase home premiums by 15% + add flood exclusions |
| **Fraud Team**   | Deploy Random Forest model to auto claims (97.85% accuracy) |
| **Operations**   | Fast-track claims < SAR 5,000 via AI triage |
| **Compliance**   | Freeze sales in low-solvency regions        |
| **CX Team**      | Chatbot rollout for status updates          |

## 🚀 13. Future Work
- Forecast long-tail claim liabilities using earned premium patterns
- Map fraud hotspots by region (Jeddah vs. Riyadh)
- Model Customer Lifetime Value with renewal data
- Implement telematics for auto policy pricing

## 🛠️ 14. Technical Details

**Tool Usage**
| Tool       | Usage                                  |
|------------|----------------------------------------|
| Python     | Pandas (EDA), Scikit-learn (Random Forest) |


**Explore:**
- Python

## ⚠️ 15. Assumptions & Caveats
- Policies with `loss_ratio > 40` excluded as data errors
- `"UNK"` region policies excluded from geo-analysis
- Fraud model trained only on flagged cases
- Zero renewal data treated as system error

---
