# Credit Risk Decision Engine

## From Default Prediction to Underwriting Policy

This project builds an end-to-end credit risk decisioning framework using LendingClub accepted and rejected loan data. The objective is to move beyond default prediction and translate credit risk estimates into practical lending decisions around approval, pricing, limit assignment, profitability, eligibility, and portfolio monitoring.

The core question addressed in this project is:

> How should a lender approve, decline, price, and limit new applicants based on expected credit risk and profitability?

Most credit risk projects stop at predicting whether a borrower will default. This project goes further by connecting probability of default modelling to expected loss, risk-adjusted profitability, pricing floors, limit caps, manual review rules, and a final underwriting policy engine.

---

## Project Overview

The project simulates how a lender could use application data and credit risk modelling to make new applicant decisions.

It covers:

- Probability of Default, or PD, modelling
- Logistic regression and tree-based challenger models
- Model calibration and out-of-time validation
- WOE/IV production-style scorecard development
- Scorecard points table and score band validation
- LGD and expected loss analysis
- Risk-adjusted profitability analysis
- Risk-based pricing floors
- Limit assignment strategy
- Hard eligibility and manual review rules
- Rejected applicant analysis
- Final approval, repricing, manual review, decline, and exception-only policy engine

---

## Business Problem

A lender receives many loan applications and must decide:

- Who should be approved?
- Who should be declined?
- Who should go to manual review?
- What interest rate should approved borrowers receive?
- What loan amount or credit limit should be offered?
- Which segments are profitable after expected losses and costs?
- Which applicants are underpriced or overexposed?
- How should rejected applicants be reviewed?

A single PD score or credit cut-off is not enough to answer these questions. A useful lending strategy needs to combine risk, pricing, affordability, profitability, exposure control, and eligibility rules.

This project therefore treats credit risk modelling as part of a broader credit policy and decisioning framework.

---

## Dataset

The project uses LendingClub accepted and rejected loan data.

### Accepted Loans

Accepted loans are used for supervised modelling because they contain observed repayment outcomes.

They are used for:

- Target definition
- PD model development
- Model validation
- Calibration analysis
- Scorecard development
- LGD and expected loss analysis
- Profitability analysis
- Pricing and limit policy
- Final policy engine design

### Rejected Loans

Rejected loans are used for applicant population analysis and reject-inference-style review.

They are used for:

- Accepted versus rejected applicant comparison
- Rejected applicant data quality checks
- Scorable versus unscorable rejected applicant analysis
- Directional reject-inference-style simulation

Rejected applicants do not have observed repayment outcomes, so they are not used directly in the main supervised PD model.

---

## Target Definition

The modelling target is a binary default indicator based on accepted loan outcomes.

| Loan Outcome | Target | Interpretation |
|---|---:|---|
| Fully Paid | 0 | Good loan |
| Charged Off | 1 | Bad loan |
| Default | 1 | Bad loan |
| Active or unresolved status | Excluded | Final outcome not observed |

The target answers the underwriting question:

> At the time of application, how likely is this borrower to become a bad loan?

---

## Methodology

The project follows a structured credit risk modelling and policy design process.

### 1. Data Cleaning and Target Definition

- Standardized column names
- Cleaned key fields such as loan term, interest rate, employment length, DTI, and revolving utilization
- Parsed date fields
- Defined resolved loan population
- Created binary default target

### 2. Rejected Applicant Cleaning and Overview

- Cleaned rejected applicant risk score and DTI fields
- Identified missing, invalid, and extreme values
- Compared accepted and rejected applicants using common fields
- Defined scorable, high-caution, and unscorable rejected applicant groups

### 3. Feature Engineering and Governance

- Created application-time affordability and credit features
- Applied caps and log transformations using train-only parameters
- Added missingness and no-event flags
- Excluded leakage variables, post-origination fields, target variables, and LendingClub decision outputs
- Created a feature governance table documenting feature inclusion and exclusion decisions

### 4. Out-of-Time Train, Validation, and Test Split

The model was evaluated using an out-of-time split to better simulate future originations.

This is important because borrower mix, underwriting standards, pricing, and default rates can change over time.

### 5. PD Model Development

Models tested:

- Baseline logistic regression
- Class-weighted logistic regression
- Tuned logistic regression
- Gradient boosting challenger model

Evaluation metrics:

- AUC
- Gini
- KS statistic
- Brier score
- Average precision
- Decile analysis
- Actual versus predicted default rates

The tuned logistic regression was selected as the primary PD model because it provided the best balance of discrimination, calibration, interpretability, and policy usability.

### 6. Calibration and Risk Banding

Several calibration methods were tested, including:

- Simple scaling
- Intercept calibration
- Logistic recalibration
- Isotonic calibration

The final PD model was then segmented into business-friendly risk bands:

| PD Band | Description |
|---|---|
| A | Very Low Risk |
| B | Low Risk |
| C | Medium Risk |
| D | High Risk |
| E | Very High Risk |

### 7. Approval Cut-off Analysis

The project explored statistical and business cut-offs, including Youden’s J statistic.

However, a single PD or credit score cut-off was not used as the final decision rule because lending decisions also depend on:

- Loan amount
- Pricing
- Loan term
- LGD
- Funding cost
- Operating cost
- Affordability
- Expected profitability
- Risk appetite

### 8. WOE/IV Production Scorecard

A production-style scorecard was developed using:

- Train-only binning
- Weight of Evidence transformation
- Information Value screening
- Monotonicity checks
- Logistic regression on WOE-transformed variables
- Scorecard points scaling
- Score band validation
- PSI stability checks

The scorecard provides an interpretable alternative to the primary PD model.

### 9. LGD and Expected Loss

Expected loss was calculated as:

```text
Expected Loss = PD × LGD × EAD
