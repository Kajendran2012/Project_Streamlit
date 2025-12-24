# 📊 Streamlit Excel Validation & Processing App

A **Streamlit-based internal tool** designed to validate, clean, and split Excel data based on complex business rules related to **Castle & DUNS decision logic**.  
This app ensures **data quality, rule compliance, and standardized outputs** for downstream processing.

---

## 🚀 Features Overview

✅ Secure Login (Session-Based)  
📤 Excel Upload & Sheet Validation  
🔍 Advanced Business Rule Validation  
🧹 Column Cleanup & Standardization  
📊 PH / FH Record Analysis  
⬇️ CSV Export (PH, FH, or Combined)

---

## 🧱 Tech Stack

- **Python**
- **Streamlit**
- **Pandas**
- **Excel (.xlsx) Processing**

---

## 🔐 Step 1: Login Page

- Uses **Streamlit session state** for authentication
- Prevents access unless logged in
- Styled with **custom CSS** for centered UI

> 🔒 *Authentication logic can be extended to integrate real user validation*

---

## 📤 Step 2: Upload Excel File

- Accepts only `.xlsx` files
- Blocks workflow until a valid file is uploaded

---

## 📄 Step 3: Sheet Selection & Preview

- Targets a **specific required sheet**:

## DS Feedback Template Castle 2.0
---

- Displays:
- Row & column count
- Data preview (first few rows)

❌ Stops execution if the sheet is missing

---

## 🧪 Step 4: Data Validation Engine (Core Logic)

### 🔹 Mandatory Columns Check
Ensures these columns exist and contain no missing values:
- `DS_CastleSiteDUNS_Decision`
- `DS_Research_Decision`
- `prtl_hrchy_flg`

---

### 🔹 Business Rule Validation

#### ✅ Accept_Castle_Site_DUNS
- Allowed `DS_Research_Decision`:
- `Accept Castle Match`
- `Proposed different Site DUNS`
- `prtl_hrchy_flg` must be **N**
- Additional mandatory DUNS fields if **Proposed different Site DUNS**

---

#### ❌ Reject_Both
- `prtl_hrchy_flg` must be **Y**
- Invalid if `DS_Research_Decision` is:
- `Accept Castle Match`
- `Proposed different Site DUNS`

Special handling for:
- No Site/Domestic/GU DUNS found
- Proposed GU / DU / Parent DUNS scenarios

---

### 🔢 DUNS Number Validation
All proposed DUNS fields must:
- Be **numeric**
- Be exactly **9 digits**

Validated using reusable helper logic.

---

### ⛔ Fail-Fast Design
- Any rule violation:
- Shows error message
- Displays affected rows
- Stops execution immediately

✔️ If all checks pass → success message displayed

---

## 🧹 Step 5: Column Cleanup

Removes unnecessary metadata columns such as:
- Match grades
- Confidence codes
- DNB hierarchy fields

✔️ Ensures final dataset contains **exactly 29 columns**

---

## 👀 Step 6: Cleaned Data Preview

Displays:
- Cleaned dataset preview
- Row & column counts
- Summary counts:
- **PH records (`prtl_hrchy_flg = Y`)**
- **FH records (`prtl_hrchy_flg = N`)**

---

## ⬇️ Step 7: Download Options

Users can download:
- 📁 **PH data only**
- 📁 **FH data only**
- 📁 **Combined PH + FH data**

✔️ Custom file naming  
✔️ UTF-8 encoded CSV output

---

## 🎯 Key Benefits

- Enforces **strict business rules**
- Prevents bad data from moving downstream
- Reduces manual validation effort
- Built for **operational efficiency & audit readiness**

---

## 📌 Ideal Use Cases

- Data Quality Validation
- Castle vs Research Decision Reconciliation
- DUNS Hierarchy Processing
- Internal Ops & Analytics Teams

---

## 🛠️ Future Enhancements

- Role-based authentication
- Logging & audit trail
- Config-driven rule management
- Deployment via Streamlit Cloud / Docker

---

## 👤 Author

**KJ**  
Business / Data Analyst  
Python • SQL • Power BI • Streamlit

---

⭐ *If this project helps you, consider giving it a star!*
