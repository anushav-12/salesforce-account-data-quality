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

![Import wizard](screenshots/01_import_wizard_195_records.png)
![Bulk job result](screenshots/02_bulk_job_159_loaded_36_failed.png)
![Error CSV](screenshots/03_error_csv_invalid_country.png)

### 2. Standardise and reload: 34 loaded, 2 failed
Standardised the country values to one format (for example `UK` and `united kingdom` become **United Kingdom**, `IN`, `india` and `Bharat` become **India**, `SG` becomes **Singapore**) and reloaded the failed records. 34 loaded and 2 failed: one was blocked as an exact duplicate by the duplicate rule, and one still had an invalid country value.

Total accounts after load: **194** *(TODO: add one line explaining how the last record was added, e.g. fixed and created manually)*.

![Reload result](screenshots/04_reload_34_loaded_2_failed.png)
![Accounts by Billing Country, 194 total](screenshots/10_report_billing_country_194_before.png)

### 3. Duplicate controls
Activated the **Standard Account Duplicate Rule** with its **Matching Rule**, set to **Allow with Alert and Report**.

- It blocked one exact duplicate during the reload.
- The Duplicate Record Sets list stayed empty, and **near-duplicates were not flagged**, so the standard rule is not enough on its own.

![Matching rule](screenshots/05_matching_rule_active.png)
![Duplicate rule](screenshots/06_duplicate_rule_active.png)
![Exact duplicate blocked](screenshots/07_exact_duplicate_blocked.png)
![Duplicate record sets empty](screenshots/08_duplicate_record_sets_empty.png)

### 4. Manual duplicate review and cleanup: 194 to 179
Because the rule missed near-duplicates, I reviewed accounts by name, phone and website, and deleted **15 duplicates** (exact and near). Accounts went from **194 to 179**.

![Near-duplicate examples](screenshots/09_near_duplicate_examples.png)
![Deleting duplicates](screenshots/11_delete_15_duplicates.png)

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
| India | *(TODO)* |
| Singapore | *(TODO)* |
| United Kingdom | *(TODO)* |
| **Total** | **179** |

![Billing country, 179 after cleanup](screenshots/12_report_billing_country_179_after.png)
![Missing Phone](screenshots/13_report_missing_phone.png)
![Missing Industry](screenshots/14_report_missing_industry.png)
![Missing Website](screenshots/15_report_missing_website.png)
![Missing City](screenshots/16_report_missing_city.png)

---

## Data Dictionary

| Field | Meaning | Rule applied |
|---|---|---|
| Account Name | Customer or company name | Required; checked for duplicates |
| Billing Country | Country of the billing address | Must be a valid standard value (India, Singapore, United Kingdom) |
| Billing City | City of the billing address | Completeness monitored by report |
| Phone | Main contact number | Digits only, no negative or invalid values; completeness monitored |
| Industry | Customer industry | Completeness monitored |
| Website | Company website | Completeness monitored; used to spot duplicates |

*(TODO: add any other columns from your CSV.)*

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

