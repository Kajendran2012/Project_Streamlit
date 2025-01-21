# Project_Streamlit

import streamlit as st
import pandas as pd

# Initialize session state for login
if "logged_in" not in st.session_state:
    st.session_state["logged_in"] = False

# Custom CSS for centering content
st.markdown(
    """
    <style>
    .centered-content {
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
        text-align: center;
    }
    h1 {
        text-align: center;
    }
    </style>
    """,
    unsafe_allow_html=True
)

# Step 1: Log In Page
if not st.session_state["logged_in"]:
    st.markdown('<div class="centered-content">', unsafe_allow_html=True)
    st.markdown("<h1>FT_Upload Process</h1>", unsafe_allow_html=True)

    # Mock authentication system
    username = st.text_input("Username")
    password = st.text_input("Password", type="password")
    login_button = st.button("Log In")

    if login_button or (username and password):
        if username and password:  # Add real authentication check here if needed
            st.session_state["logged_in"] = True
            st.success("Logged in successfully! Please proceed.")
        else:
            st.error("Invalid username or password. Please try again.")
    st.markdown('</div>', unsafe_allow_html=True)
    st.stop()

# Step 2: Upload Excel File
uploaded_file = st.file_uploader("Upload an Excel file (xlsx format):", type="xlsx")
if not uploaded_file:
    st.info("Please upload an Excel file to proceed.")
    st.stop()

# Step 3: Work on Specific Tab and Preview
sheet_name = "DS Feedback Template Castle 2.0"
try:
    excel_data = pd.ExcelFile(uploaded_file)
    if sheet_name not in excel_data.sheet_names:
        st.error(f"Sheet '{sheet_name}' not found in the uploaded file.")
        st.stop()

    df = pd.read_excel(uploaded_file, sheet_name=sheet_name, dtype=str)
    st.write("### Preview of Selected Sheet")
    st.write(f"Number of Rows: {df.shape[0]}, Columns: {df.shape[1]}")
    st.dataframe(df.head())
except Exception as e:
    st.error(f"An error occurred: {e}")
    st.stop()

# Step 4: Check Conditions
required_columns = [
    "DS_CastleSiteDUNS_Decision",
    "DS_Research_Decision",
    "prtl_hrchy_flg"
]

# Check for missing columns
missing_columns = [col for col in required_columns if col not in df.columns]
if missing_columns:
    st.error(f"The following required columns are missing: {', '.join(missing_columns)}")
    st.stop()

# Check for missing values in the required columns
missing_values = df[df[required_columns].isna().any(axis=1)]
if not missing_values.empty:
    st.error("Some rows are missing required values in one or more required columns. Highlighting the affected columns.")
    for col in required_columns:
        missing_rows = missing_values[missing_values[col].isna()]
        if not missing_rows.empty:
            st.write(f"### Missing Values in Column: {col}")
            st.dataframe(missing_rows)
    st.stop()

# Condition 1
condition_1 = df[df["DS_CastleSiteDUNS_Decision"] == "Accept_Castle_Site_DUNS"]
condition_1_missing = condition_1[(condition_1["DS_Research_Decision"].isna()) | (condition_1["prtl_hrchy_flg"].isna())]
if not condition_1_missing.empty:
    st.error("For rows where 'DS_CastleSiteDUNS_Decision' is 'Accept_Castle_Site_DUNS', the following columns have missing values:")
    if condition_1_missing["DS_Research_Decision"].isna().any():
        st.write("### Missing Values in Column: DS_Research_Decision")
        st.dataframe(condition_1_missing[condition_1_missing["DS_Research_Decision"].isna()])
    if condition_1_missing["prtl_hrchy_flg"].isna().any():
        st.write("### Missing Values in Column: prtl_hrchy_flg")
        st.dataframe(condition_1_missing[condition_1_missing["prtl_hrchy_flg"].isna()])
    st.stop()

# Sub-condition for Proposed different Site DUNS
condition_proposed_site_duns = condition_1[condition_1["DS_Research_Decision"] == "Proposed different Site DUNS"]
missing_proposed_site_duns = condition_proposed_site_duns[
    condition_proposed_site_duns[["Proposed_SiteDUNS", "Proposed_ParentDUNS", "Proposed_DomesticDUNS", "Proposed_GUDUNS"]].isna().any(axis=1)
]
if not missing_proposed_site_duns.empty:
    st.error("For rows where 'DS_CastleSiteDUNS_Decision' is 'Accept_Castle_Site_DUNS' and 'DS_Research_Decision' is 'Proposed different Site DUNS', the following columns must be filled:")
    st.write("- Proposed_SiteDUNS")
    st.write("- Proposed_ParentDUNS")
    st.write("- Proposed_DomesticDUNS")
    st.write("- Proposed_GUDUNS")
    st.dataframe(missing_proposed_site_duns)
    st.stop()

# Sub-condition for Reject_Both
condition_reject_both = df[df["DS_CastleSiteDUNS_Decision"] == "Reject_Both"]
invalid_reject_both = condition_reject_both[
    condition_reject_both["DS_Research_Decision"].isin(["Accept Castle Match", "Proposed different Site DUNS"])
]
if not invalid_reject_both.empty:
    st.error("For rows where 'DS_CastleSiteDUNS_Decision' is 'Reject_Both', 'DS_Research_Decision' cannot be 'Accept Castle Match' or 'Proposed different Site DUNS'.")
    st.dataframe(invalid_reject_both)
    st.stop()

# Condition 2
condition_2 = df[df["DS_CastleSiteDUNS_Decision"] == "Reject_Both"]
condition_2_missing = condition_2[condition_2["prtl_hrchy_flg"].isna()]
if not condition_2_missing.empty:
    st.error("For rows where 'DS_CastleSiteDUNS_Decision' is 'Reject_Both', the following column has missing values:")
    st.write("### Missing Values in Column: prtl_hrchy_flg")
    st.dataframe(condition_2_missing)
    st.stop()

condition_2_failed = condition_2[condition_2["prtl_hrchy_flg"] != "Y"]
if not condition_2_failed.empty:
    st.error("Some rows do not meet the required conditions for 'Reject_Both':")
    st.dataframe(condition_2_failed)
    st.stop()

# Sub-condition for Reject_Both with No Site/Domestic/GU DUNS found
condition_reject_both_no_duns = condition_reject_both[
    (condition_reject_both["DS_Research_Decision"] == "No Site/Domestic/GU DUNS found") &
    (condition_reject_both["prtl_hrchy_flg"].notna())
]
if not condition_reject_both_no_duns.empty:
    st.error("For rows where 'DS_CastleSiteDUNS_Decision' is 'Reject_Both' and 'DS_Research_Decision' is 'No Site/Domestic/GU DUNS found', 'prtl_hrchy_flg' must be blank.")
    st.dataframe(condition_reject_both_no_duns)
    st.stop()

# Validate Conditions for 'Accept_Castle_Site_DUNS'
condition_1_failed = condition_1[
    (~condition_1["DS_Research_Decision"].isin(["Accept Castle Match", "Proposed different Site DUNS"])) |
    (condition_1["prtl_hrchy_flg"] != "N")
]

if not condition_1_failed.empty:
    st.error("Some rows do not meet the required conditions for 'Accept_Castle_Site_DUNS':")
    st.dataframe(condition_1_failed)
    st.stop()

# New Condition: Validate 'Proposed Parent/DU/GU DUNS'
decision_parent_du_gu = df[df["DS_Research_Decision"] == "Proposed Parent/DU/GU DUNS"]
invalid_parent_du_gu = decision_parent_du_gu[
    (decision_parent_du_gu["Proposed_ParentDUNS"].str.len() != 9) |
    (decision_parent_du_gu["Proposed_DomesticDUNS"].str.len() != 9) |
    (decision_parent_du_gu["Proposed_GUDUNS"].str.len() != 9)
]

if not invalid_parent_du_gu.empty:
    st.error("For rows where 'DS_Research_Decision' is 'Proposed Parent/DU/GU DUNS', the following columns must be filled with 9-digit values:")
    st.write("- Proposed_ParentDUNS")
    st.write("- Proposed_DomesticDUNS")
    st.write("- Proposed_GUDUNS")
    st.dataframe(invalid_parent_du_gu)
    st.stop()

# New Condition: Validate 'Proposed DU/GU DUNS'
decision_du_gu = df[df["DS_Research_Decision"] == "Proposed DU/GU DUNS"]
invalid_du_gu = decision_du_gu[
    (decision_du_gu["Proposed_DomesticDUNS"].str.len() != 9) |
    (decision_du_gu["Proposed_GUDUNS"].str.len() != 9)
]

if not invalid_du_gu.empty:
    st.error("For rows where 'DS_Research_Decision' is 'Proposed DU/GU DUNS', the following columns must be filled with 9-digit values:")
    st.write("- Proposed_DomesticDUNS")
    st.write("- Proposed_GUDUNS")
    st.dataframe(invalid_du_gu)
    st.stop()

# New Condition: Validate 'Proposed GU DUNS'
decision_gu = df[df["DS_Research_Decision"] == "Proposed GU DUNS"]
invalid_gu = decision_gu[
    decision_gu["Proposed_GUDUNS"].str.len() != 9
]

if not invalid_gu.empty:
    st.error("For rows where 'DS_Research_Decision' is 'Proposed GU DUNS', the following column must be filled with a 9-digit value:")
    st.write("- Proposed_GUDUNS")
    st.dataframe(invalid_gu)
    st.stop()

## Additional Condition: Validate 'Proposed_ParentDUNS'
decision_parentduns_filled = df[
    (df["Proposed_ParentDUNS"].str.len() == 9) &
    (df["DS_Research_Decision"] != "Proposed Parent/DU/GU DUNS") &
    # Add exceptions here
    (~df["DS_Research_Decision"].isin(["Proposed different Site DUNS", "Other Valid Case"]))  # Add valid cases
]

if not decision_parentduns_filled.empty:
    st.error("For rows where 'Proposed_ParentDUNS' is filled with a 9-digit value, 'DS_Research_Decision' must be 'Proposed Parent/DU/GU DUNS', unless explicitly allowed.")
    st.dataframe(decision_parentduns_filled)
    st.stop()

# Condition 2: Validate 'Reject_Both' for 'prtl_hrchy_flg' = 'Y'
condition_2 = df[df["DS_CastleSiteDUNS_Decision"] == "Reject_Both"]

# Main Condition: Check if 'prtl_hrchy_flg' is not equal to 'Y'
condition_2_invalid = condition_2[condition_2["prtl_hrchy_flg"] != "Y"]
if not condition_2_invalid.empty:
    st.error("For rows where 'DS_CastleSiteDUNS_Decision' is 'Reject_Both', 'prtl_hrchy_flg' must be 'Y'. The following rows do not meet this condition:")
    st.dataframe(condition_2_invalid)
    st.stop()

# Helper function to validate 9-digit numeric values
def is_valid_numeric(series):
    """Ensure the series contains 9-digit numeric values."""
    series = series.fillna("").astype(str)  # Replace NaN with empty string and convert to string
    return series.str.isdigit() & (series.str.len() == 9)

# Subcondition 1: Validate 'Proposed GU DUNS'
subcondition_gu = condition_2[condition_2["DS_Research_Decision"] == "Proposed GU DUNS"]
invalid_gu_sub = subcondition_gu[
    (~is_valid_numeric(subcondition_gu["Proposed_GUDUNS"])) |  # 'Proposed_GUDUNS' must contain 9-digit numeric value
    (subcondition_gu["Proposed_DomesticDUNS"].notna() & subcondition_gu["Proposed_DomesticDUNS"].str.len() > 0) |  # 'Proposed_DomesticDUNS' must be empty
    (subcondition_gu["Proposed_ParentDUNS"].notna() & subcondition_gu["Proposed_ParentDUNS"].str.len() > 0)  # 'Proposed_ParentDUNS' must be empty
]

if not invalid_gu_sub.empty:
    st.error(
        "For rows where 'DS_CastleSiteDUNS_Decision' is 'Reject_Both' and 'DS_Research_Decision' is 'Proposed GU DUNS':\n"
        "- 'Proposed_GUDUNS' must contain a 9-digit numeric value.\n"
        "- 'Proposed_DomesticDUNS' and 'Proposed_ParentDUNS' must be empty.\n"
        "The following rows do not meet these conditions:"
    )
    st.dataframe(invalid_gu_sub)
    st.stop()

# Subcondition 2: Validate 'Proposed DU/GU DUNS'
subcondition_du_gu = condition_2[condition_2["DS_Research_Decision"] == "Proposed DU/GU DUNS"]
invalid_du_gu_sub = subcondition_du_gu[
    (~is_valid_numeric(subcondition_du_gu["Proposed_DomesticDUNS"])) |  # 'Proposed_DomesticDUNS' must contain 9-digit numeric value
    (~is_valid_numeric(subcondition_du_gu["Proposed_GUDUNS"])) |  # 'Proposed_GUDUNS' must contain 9-digit numeric value
    (subcondition_du_gu["Proposed_ParentDUNS"].notna() & subcondition_du_gu["Proposed_ParentDUNS"].str.len() > 0)  # 'Proposed_ParentDUNS' must be empty
]

if not invalid_du_gu_sub.empty:
    st.error(
        "For rows where 'DS_CastleSiteDUNS_Decision' is 'Reject_Both' and 'DS_Research_Decision' is 'Proposed DU/GU DUNS':\n"
        "- 'Proposed_DomesticDUNS' and 'Proposed_GUDUNS' must contain 9-digit numeric values.\n"
        "- 'Proposed_ParentDUNS' must be empty.\n"
        "The following rows do not meet these conditions:"
    )
    st.dataframe(invalid_du_gu_sub)
    st.stop()

# Subcondition 3: Validate 'Proposed Parent/DU/GU DUNS'
subcondition_parent_du_gu = condition_2[condition_2["DS_Research_Decision"] == "Proposed Parent/DU/GU DUNS"]
invalid_parent_du_gu_sub = subcondition_parent_du_gu[
    (~is_valid_numeric(subcondition_parent_du_gu["Proposed_ParentDUNS"])) |  # 'Proposed_ParentDUNS' must contain 9-digit numeric value
    (~is_valid_numeric(subcondition_parent_du_gu["Proposed_DomesticDUNS"])) |  # 'Proposed_DomesticDUNS' must contain 9-digit numeric value
    (~is_valid_numeric(subcondition_parent_du_gu["Proposed_GUDUNS"]))  # 'Proposed_GUDUNS' must contain 9-digit numeric value
]

if not invalid_parent_du_gu_sub.empty:
    st.error(
        "For rows where 'DS_CastleSiteDUNS_Decision' is 'Reject_Both' and 'DS_Research_Decision' is 'Proposed Parent/DU/GU DUNS':\n"
        "- 'Proposed_ParentDUNS', 'Proposed_DomesticDUNS', and 'Proposed_GUDUNS' must contain 9-digit numeric values.\n"
        "The following rows do not meet these conditions:"
    )
    st.dataframe(invalid_parent_du_gu_sub)
    st.stop()

st.success("All required fields are filled in, and all conditions are met.")

# Step 5: Delete Unwanted Columns
columns_to_delete = [
    "DCN Combo Key",
    "DnB_Company_name",
    "DNB_Address1",
    "Dell Hierarchy ID",
    "Dell_Hierarchy_Stamp",
    "Dnb_Hq_Duns_Number",
    "Dnb_Dom_Ult_Duns_Number",
    "Dnb_Glbl_Duns_Number",
    "Match_Grade",
    "Confidence_Code",
    "Match_Status"
]
df_cleaned = df.drop(columns=[col for col in columns_to_delete if col in df.columns], errors='ignore')

# Ensure cleaned data has exactly 29 columns
if len(df_cleaned.columns) > 29:
    df_cleaned = df_cleaned.iloc[:, :29]

# Step 6: Preview the Cleaned File
st.write("### Preview of Cleaned Data")
st.write(f"Number of Rows: {df_cleaned.shape[0]}, Columns: {df_cleaned.shape[1]}")
st.dataframe(df_cleaned.head())

# Additional Preview: PH and FH Counts
st.write("### PH and FH Counts")
ph_count = df_cleaned[df_cleaned["prtl_hrchy_flg"] == "Y"].shape[0]
fh_count = df_cleaned[df_cleaned["prtl_hrchy_flg"] == "N"].shape[0]
st.write(f"PH Count: {ph_count}")
st.write(f"FH Count: {fh_count}")

# Step 7: Download Options for PH and FH
ph_data = df_cleaned[df_cleaned["prtl_hrchy_flg"] == "Y"]
fh_data = df_cleaned[df_cleaned["prtl_hrchy_flg"] == "N"]

if not ph_data.empty:
    ph_file_name = st.text_input("Enter the file name for PH data (including .csv):", "ph_data.csv")
    if ph_file_name:
        ph_csv_output = ph_data.to_csv(index=False).encode('utf-8-sig')
        st.download_button(
            label="Download PH Data as CSV",
            data=ph_csv_output,
            file_name=ph_file_name,
            mime="text/csv"
        )

if not fh_data.empty:
    fh_file_name = st.text_input("Enter the file name for FH data (including .csv):", "fh_data.csv")
    if fh_file_name:
        fh_csv_output = fh_data.to_csv(index=False).encode('utf-8-sig')
        st.download_button(
            label="Download FH Data as CSV",
            data=fh_csv_output,
            file_name=fh_file_name,
            mime="text/csv"
        )

if not ph_data.empty and not fh_data.empty:
    both_file_name = st.text_input("Enter the file name for both PH and FH data (including .csv):", "both_data.csv")
    if both_file_name:
        both_data = pd.concat([ph_data, fh_data])
        both_csv_output = both_data.to_csv(index=False).encode('utf-8-sig')
        st.download_button(
            label="Download Both PH and FH Data as CSV",
            data=both_csv_output,
            file_name=both_file_name,
            mime="text/csv"
        )
