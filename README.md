# Customer Account Data Quality: Salesforce Developer Edition

A hands-on project showing how I import, validate, cleanse, de-duplicate and monitor customer account data in Salesforce. It follows the same work a Data Governance Analyst does on customer master data: account setup, data cleansing, duplicate control and quality reporting.

> **Note:** Built in a free Salesforce Developer Edition org using **100% synthetic (fake) data**. No real customer or employer data is used.

---

## Objective

Load a deliberately messy set of customer accounts into Salesforce, find and fix the quality problems, put duplicate controls in place, and build reports that monitor data quality.

## Tools

- Salesforce Developer Edition (Data Import Wizard, Duplicate Management, Lightning Reports)
- CSV (source data and error files)

## Dataset

`data/messy_accounts.csv` contains 195 synthetic accounts with planted problems:

- Inconsistent Billing Country values (`UK`, `IN`, `india`, `Bharat`, `SG`)
- Exact and near-duplicate accounts
- Missing Phone, Industry, Website and City values
- One invalid phone value

---

## Process and Results

### 1. Initial import: 159 loaded, 36 failed
Imported 195 accounts. 36 records failed because Billing Country values were not valid picklist values. The error file showed exactly which values caused it.

![Import wizard](salesforce/01_import_wizard_195_records.png)
![Bulk job result](salesforce/02_bulk_job_159_loaded_36_failed.png)
![Error CSV, part 1](salesforce/03_error_csv_invalid_country1.png)
![Error CSV, part 2](salesforce/03_error_csv_invalid_country2.png)

### 2. Standardise and reload: 34 loaded, 2 failed, then 1 more
Standardised the country values to one format (for example `UK` becomes **United Kingdom**, `IN`, `india` and `Bharat` become **India**, `SG` becomes **Singapore**) and reloaded the failed records.

![Corrected CSV with valid countries](salesforce/03_corrected_csv_valid_country.png)

34 loaded and 2 failed: one was blocked as an exact duplicate by the duplicate rule, and one still had an invalid country value.

![Reload result, part 1](salesforce/04_reload_34_loaded_2_failed1.png)
![Reload result, part 2](salesforce/04_reload_34_loaded_2_failed2.png)

I corrected the last record with the invalid country value and reloaded it, and it loaded successfully.

![Corrected record](salesforce/05_corrected.png)
![Reload completed](salesforce/05_reloaded_1_completed.png)

Total accounts after all loads: **194** (159 + 34 + 1).

![All 194 records loaded](salesforce/06_all_194_completed.png)
![Accounts by Billing Country, 194 total](salesforce/11_report_billing_country_194_before1.png)
![Accounts by Billing Country, 194 total](salesforce/11_report_billing_country_194_before2.png)

### 3. Duplicate controls
Activated the **Standard Account Duplicate Rule** with its **Matching Rule**, set to **Allow with Alert and Report**.

- It blocked one exact duplicate during the reload.
- The Duplicate Record Sets list stayed empty, and **near-duplicates were not flagged**, so the standard rule is not enough on its own.

![Matching rule](salesforce/07_matching_rule_active.png)
![Duplicate rule](salesforce/07_duplicate_rule_active.png)

### 4. Manual duplicate review and cleanup: 194 to 179
Because the rule missed near-duplicates, I reviewed accounts by name, phone and website, and deleted **15 duplicates** (exact and near). Accounts went from **194 to 179**.

![Near-duplicate example 1](salesforce/12_near_duplicate_examples1.png)
![Near-duplicate example 2](salesforce/12_near_duplicate_examples2.png)
![Near-duplicate example 3](salesforce/12_near_duplicate_examples3.png)
![Accounts after deleting 15 duplicates: 179](salesforce/13_report_billing_country_179_after.png)

### 5. Data quality reports
Built and saved five Lightning reports to monitor completeness and consistency.

| Report | Records found |
|---|---|
| Missing Phone | 4 |
| Missing Industry | 4 |
| Missing Website | 6 |
| Missing City | 6 |
| Accounts by Billing Country (total) | 179 |

Accounts by Billing Country after cleanup:

| Billing Country | Accounts |
|---|---|
| India | *123* |
| Singapore | *27* |
| United Kingdom | *29* |
| **Total** | **179** |


![Accounts by Billing Country after cleanup: 179 total](salesforce/11_report_billing_country_179_after1.png)
![Accounts by Billing Country after cleanup: 179 total](salesforce/11_report_billing_country_179_after2.png)
![Missing Phone](salesforce/08_report_missing_phone.png)
![Missing Industry](salesforce/14_report_missing_industry.png)
![Missing Website](salesforce/09_report_missing_website.png)
![Missing City](salesforce/10_report_missing_city.png)

---

## Data Dictionary

| Field | Meaning | Rule applied |
|---|---|---|
| Account Name | Customer or company name | Required; checked for duplicates |
| Type | Account type, for example Partner | Descriptive field; not validated in this project |
| Industry | Customer industry | Completeness monitored by report |
| Phone | Main contact number | Must be a valid number (spaces allowed), no negative values; completeness monitored |
| Website | Company website | Completeness monitored; used to spot duplicates |
| Billing City | City of the billing address | Completeness monitored by report |
| Billing Country | Country of the billing address | Must be a standard value (India, Singapore, United Kingdom) |


## Data Quality Checks Used

- **Validity:** country values match the allowed list
- **Completeness:** missing phone, industry, website and city reports
- **Uniqueness:** matching and duplicate rules plus manual review
- **Consistency:** one standard format per field

## Key Learnings

- Import error files point directly to the field and value that caused a failure, which makes root-cause fixes quick.
- Inconsistent text values like `UK`, `IN` and `india` cause most load failures, so standardising them up front prevents rework.
- Standard duplicate rules catch exact matches but can miss near-duplicates, so manual review or stricter matching rules are still needed.
- Saved reports make quality problems visible without re-checking records one by one.

## Limitations and Next Steps

- Built in a Developer Edition org with a small synthetic dataset, not a production environment.
- Possible improvements: custom matching rules for near-duplicates, validation rules on Billing Country and Phone, and a Parent Account hierarchy example.

