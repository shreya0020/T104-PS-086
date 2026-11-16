# PS-086  AQL Sampling, Inspection Severity and Batch Release Engine

**Team:** T104
**Project:** AQL Sampling, Inspection Severity and Batch Release Engine
**Language:** Python 3.10+
**Repository:** https://github.com/Shrishti2602/T104-PS-086

## 1. Problem Overview

Manufacturing companies inspect incoming materials and production batches to ensure that products meet quality standards before release.

Manual inspection and decision-making can lead to inconsistent quality checks, delayed releases, errors in defect evaluation, and difficulty tracking supplier performance.

This project develops a Python-based quality inspection engine that automates AQL-based sampling, defect evaluation, supplier switching rules, batch lifecycle management, and report generation.

The application is a local, command-line and file-driven backend system.

## 2. Objectives

- Automate AQL-based acceptance sampling.
- Classify defects as critical, major, and minor.
- Accept or reject batches based on inspection results.
- Track supplier and production-line inspection states.
- Implement rework, hold, and recall procedures.
- Apply corrective actions to discontinued sources.
- Validate shelf-life requirements before release.
- Identify late inspection results.
- Generate structured CSV reports for evaluation.

## 3. Technologies Used

| Technology | Purpose |
|---|---|
| Python 3.10+ | Main programming language |
| Pytest | Automated testing |
| CSV | Input data and report generation |
| Git | Version control |
| GitHub | Source code repository |
| ReportLab | PDF generation |

## 4. Project Structure

```text
T104-PS-086/
-------- data/
---   --------- products.csv
---   --------- sources.csv
---   --------- batches.csv
---   --------- lot_size_bands.csv
---   ---------sampling_table.csv
---   --------- commands.txt
--------- src/
---   --------- quality_engine/
---       --------- __init__.py
---       --------- engine.py
--------- tests/
--------- docs/
---   --------- design_document.md
---   --------- design_document.pdf
---   --------- decision_log.md
--------- run.py
--------- make_pdf.py
--------- batch_decisions.csv
--------- switching_history.csv
--------- release_register.csv
--------- README.md
--------- .gitignore
```

## 5. Files Included in the Sample Data Pack

The supplied sample data pack contains six input files.

| File | Purpose | Data rows / commands |
|---|---|---:|
| `products.csv` | Product details, shelf life and defect-class AQL values | 8 |
| `sources.csv` | Six suppliers and two production lines, with initial switching states | 8 |
| `lot_size_bands.csv` | Inclusive lot-size ranges mapped to code letters | 11 |
| `sampling_table.csv` | Sample size, Ac, Re and reduced limits | 99 |
| `batches.csv` | Incoming and production batch information | 120 |
| `commands.txt` | Ordered command stream | 382 non-comment commands |

The data pack includes both supplier sources (`SUP-xx`) and production-line sources (`LINE-xx`).

## 6. Input File Column Reference

### 6.1 products.csv

Contains product information and the AQL values used for inspection.

| Column | Description |
|---|---|
| `product_id` | Unique product identifier, such as PRD-nn |
| `product_name` | Product name |
| `category` | Product category: PHARMA or FOOD |
| `shelf_life_days` | Total shelf life in days |
| `aql_critical` | Critical defect AQL, 0.065 |
| `aql_major` | Major defect AQL, 1.0 |
| `aql_minor` | Minor defect AQL, 1.0 or 2.5 |

### 6.2 sources.csv

Contains suppliers and production lines that own inspection histories.

| Column | Description |
|---|---|
| `source_id` | Unique source ID, such as SUP-nn or LINE-nn |
| `source_type` | SUPPLIER or LINE |
| `source_name` | Source name |
| `location` | Source location |
| `initial_state` | Initial state: NORMAL or REDUCED |
| `last_batch_date` | Previous batch date used for the 30-day gap rule |

### 6.3 lot_size_bands.csv

Maps batch lot sizes to sampling code letters.

| Column | Description |
|---|---|
| `lot_from` | Inclusive lower lot-size limit |
| `lot_to` | Inclusive upper lot-size limit |
| `code_letter` | Sampling code letter A through L, with I omitted |

### 6.4 sampling_table.csv

Contains the sampling plan for each code letter, AQL and inspection severity.

| Column | Description |
|---|---|
| `code_letter` | Lot-size sampling code |
| `aql` | Acceptance Quality Limit |
| `severity` | NORMAL, TIGHTENED or REDUCED |
| `sample_size` | Required sample size (n) |
| `ac` | Acceptance number |
| `re` | Rejection number |
| `reduced_limit` | Maximum total defects allowed for the NORMAL-to-REDUCED test; used from NORMAL rows |

If the required sample size is greater than or equal to the lot size, the entire lot is inspected. The Ac and Re values remain unchanged.

### 6.5 batches.csv

Contains batch and raw-material information.

| Column | Description |
|---|---|
| `batch_id` | Unique batch identifier, such as BT-nnnn |
| `product_id` | Reference to products.csv |
| `source_id` | Reference to sources.csv |
| `rm_lot` | Raw-material lot identifier used for hold and recall relationships |
| `lot_size` | Total units in the batch |
| `mfg_date` | Manufacturing date |
| `expiry_date` | Expiry date |

### 6.6 Data Format Conventions

- Dates use `YYYY-MM-DD`.
- Timestamps use `YYYY-MM-DD HH:MM:SS` in 24-hour IST format, without a timezone suffix.
- IDs use uppercase letters and fixed prefixes.
- Boolean values use `true` or `false`.
- Empty cells indicate not applicable where permitted.
- Row references in the sample pack are 1-based data-row numbers; the header is row 0.
- Command-file line references count physical lines, including comments.

## 7. Command File Format

The `commands.txt` file contains one command per line.

General format:

```text
COMMAND_NAME|arg1|arg2|...
```

Commands must be processed strictly in file order.

- Lines starting with `#` are comments.
- Blank lines are ignored.
- An empty argument, represented by two consecutive pipes, means that the argument was not supplied.

### Supported Commands

| Command | Arguments in order | Purpose |
|---|---|---|
| `RECEIVE_BATCH` | `batch_id|timestamp` | Receives a batch into QUARANTINE |
| `SAMPLE` | `batch_id|timestamp` | Selects the sample using the current inspection state |
| `RECORD_RESULT` | `batch_id|timestamp|critical|major|minor` | Records defect counts and determines the decision |
| `REWORK` | `batch_id|timestamp` | Requests one permitted rework attempt |
| `HOLD_REQUEST` | `batch_id|timestamp|cause_batch_id` | Requests a hold on a related released batch |
| `COMPLAINT` | `batch_id|timestamp|complaint_ref` | Records a customer complaint and triggers recall processing |
| `CORRECTIVE_ACTION` | `source_id|timestamp|car_ref` | Restarts a discontinued source at TIGHTENED |
| `PRINT_DECISIONS` | No arguments | Requests inspection decision output |
| `PRINT_SWITCHING` | `source_id` | Requests a source switching timeline |
| `PRINT_RELEASE_REGISTER` | No arguments | Requests lifecycle event output |

### Example Commands

```text
RECEIVE_BATCH|BT-0001|2026-05-01 09:00:00
SAMPLE|BT-0001|2026-05-01 10:00:00
RECORD_RESULT|BT-0001|2026-05-02 10:00:00|0|1|2
REWORK|BT-0001|2026-05-02 11:00:00
PRINT_DECISIONS
PRINT_SWITCHING|SUP-01
PRINT_RELEASE_REGISTER
```

The actual sample command file contains the supplied command sequence. The example above illustrates the syntax only.

## 8. Planted Scenarios and Edge Cases

The sample data pack deliberately includes scenarios to test important business rules.

| Scenario | What it tests | Sample reference |
|---|---|---|
| EC-01 | Lot size 280 versus 281 maps to different code letters | BT-0019 and BT-0029 |
| EC-02 | Defects exactly equal to Ac are accepted; exactly equal to Re are rejected | BT-0018 and BT-0044 |
| EC-03 | A result between Ac and Re under REDUCED is accepted but switches the source to NORMAL | BT-0004 |
| EC-04 | Two rejections within five consecutive batches trigger TIGHTENED | SUP-02, BT-0022 and BT-0054 |
| EC-05 | Rework is allowed when major defects equal exactly twice Re | BT-0030; comparison with BT-0070 |
| EC-06 | Hold requests on day 30 and day 31 after release are treated differently | BT-0073 and BT-0072 |
| EC-07 | Recall cascades to eligible related batches but excludes already expired batches | BT-0073 and related RM lot RM-S05-2608 |
| EC-08 | A result recorded exactly 48 hours after sampling is on time | BT-0047; comparison with BT-0031 |

### Additional Planted Traps

- **BT-0086:** A late acceptance is skipped in the 10-counted-acceptance REDUCED test.
- **SUP-03 / BT-0082:** The fifth rejection while TIGHTENED leads to DISCONTINUED. Sampling is refused until corrective action.
- **BT-0090:** An otherwise accepted batch is rejected as SHORT_DATED because remaining shelf life is below 75%.
- **BT-0001:** A production gap greater than 30 days causes a REDUCED source to return to NORMAL before sampling.
- **Complaint commands:** Additional complaint scenarios test recall cascades for BT-0061 and BT-0074.

These cases help validate boundary conditions, switching transitions, lifecycle rules, and timing requirements.

## 9. What Is Not Included in the Sample Pack

The supplied sample pack does not provide expected output CSV files for the sample input set.

The developer must validate the implementation using automated tests and the business rules in the project brief.

A separate hidden input set with the same schema is used during final evaluation. The evaluator compares generated output fields against expected results.

The sample pack is an input and evaluation-clarification resource; it does not replace the original assignment brief or mentor clarifications.

The project is a local backend application. A graphical user interface, web application, database server, authentication system, and external integrations are outside the stated scope.

## 10. Output Format Expected at Evaluation

The engine must generate three CSV files with header rows and the exact column order specified below.

### 10.1 batch_decisions.csv

Contains one row per `RECORD_RESULT` command. Re-inspection results are recorded as separate rows.

Rows must be in command order.

**Columns in exact order:**

```text
batch_id,inspection_no,source_state,code_letter,n_critical,n_major,n_minor,full_inspection,critical,major,minor,ac_major,re_major,ac_minor,re_minor,decision,batch_status,late,rule_ids
```

- `decision`: ACCEPT or REJECT.
- `batch_status`: RELEASED, REJECTED, ON_HOLD or RECALLED.
- `late`: identifies results recorded more than 48 hours after sampling.
- `rule_ids`: rule-reference field.

### 10.2 switching_history.csv

Contains one row for every source inspection-state transition.

Rows must be in time order.

**Columns in exact order:**

```text
source_id,timestamp,from_state,to_state,trigger_batch_id,reason
```

### 10.3 release_register.csv

Contains one row per lifecycle event.

Events include RELEASED, REJECTED, REWORK, ON_HOLD, HOLD_REFUSED, RECALLED, NOT_RECALLED_EXPIRED and SAMPLE_REFUSED.

**Columns in exact order:**

```text
timestamp,batch_id,event,from_status,to_status,reason,cause_ref
```

### Evaluation Binding Rules

The following clarifications apply where the original brief leaves a format detail open:

- Each defect class uses its own sample size, Ac and Re from its AQL row.
- The inspector draws the largest class-specific sample size.
- Each defect class is judged using its own Ac and Re.
- Switching uses first-inspection decisions only. Rework and hold re-inspections do not count.
- A SHORT_DATED rejection counts as an acceptance for switching.
- Within five consecutive batches means the current batch and the four before it in the current NORMAL spell.
- A late accepted batch is skipped in the 10-in-a-row REDUCED test: it neither counts nor breaks the run. A rejection breaks the run.
- The reduced limit comes from the NORMAL row of the tenth counted lot's code letter, separately for each defect class.
- The 30-day production gap is measured between consecutive RECEIVE_BATCH dates for a source and is applied before sampling.
- The hold window is calculated as the date of HOLD_REQUEST minus the release date, and must be 30 days or less.
- Recall cascades to RELEASED or ON_HOLD batches sharing the same raw-material lot where expiry date is on or after the recall date.
- A batch is SHORT_DATED when remaining shelf life at release is less than 75% of its total shelf life.
- Decision values are ACCEPT or REJECT.
- Batch status values are RELEASED, REJECTED, ON_HOLD or RECALLED.
- Example reason values include CRITICAL, MAJOR_GE_RE, MINOR_GE_RE, SHORT_DATED and BETWEEN_AC_RE.

## 11. Developer Setup and Usage

### Clone the repository

```bash
git clone https://github.com/Shrishti2602/T104-PS-086.git
cd T104-PS-086
```

### Create a virtual environment

```bash
python -m venv .venv
```

### Activate the environment (Windows PowerShell)

```powershell
.venv\Scripts\Activate.ps1
```

### Install the project and required tools

```bash
python -m pip install -e .
python -m pip install pytest reportlab markdown
```

### Run the project

```bash
python run.py
```

### Run automated tests

```bash
python -m pytest -q
```

### Generate the design document PDF

```bash
python make_pdf.py
```

## 12. Week 3 Compliance Summary

### Implemented Features

- AQL-based sampling and lot-size code-letter selection.
- Critical, major, and minor defect classification.
- Batch acceptance and rejection decisions.
- Supplier and production-line switching between NORMAL, TIGHTENED, REDUCED and DISCONTINUED.
- Rework and reinspection.
- Batch hold and hold reinspection.
- Recall and related-batch recall handling.
- Corrective action for discontinued sources.
- Shelf-life validation before release.
- Late inspection result handling.
- CSV report generation.

### Testing Evidence

The supplied sample data was executed successfully.

| Item | Result |
|---|---:|
| Products loaded | 8 |
| Sources loaded | 8 |
| Batches loaded | 120 |
| Sampling rules loaded | 99 |
| Commands executed | 382 |
| Inspection decisions | 122 |
| Switching transitions | 9 |
| Lifecycle events | 134 |
| Automated tests passed | 68 |

Three CSV reports are generated:

- `batch_decisions.csv`
- `switching_history.csv`
- `release_register.csv`

The local automated test suite completed with 68 passing tests. This is local test evidence and does not represent a claim that the separate hidden evaluation set has been passed.

### Remaining Tasks

- Record and add the five-minute project demonstration video link.
- Complete the final repository and submission review.

## 13. Conclusion

The AQL Sampling, Inspection Severity and Batch Release Engine automates important manufacturing quality-control activities. It provides consistent inspection decisions, supplier monitoring, batch lifecycle tracking, and structured reports.

The project is designed to process command-file inputs and produce evaluation-ready CSV outputs while applying the specified sampling, switching, rework, hold, recall, and shelf-life rules.
