# Fraud Strategy Lab — Databricks

Hands-on learning project focused on **fraud strategy for card transactions**, using **Databricks, PySpark and Delta Lake**.

The goal is to reproduce the analytical workflow behind an antifraud strategy: prepare transactional data, investigate suspicious behavior, build rule candidates, measure performance, tune thresholds and validate the strategy over time.

> **Project status:** in progress. The data foundation is complete and the project is currently moving into temporal validation and fraud investigation.

---

## Objective

This project is designed to answer practical antifraud questions such as:

- How should transactional data be prepared for fraud analysis?
- How can suspicious behavioral patterns be identified?
- How do we distinguish transaction duplication from repeated customer/card behavior?
- How should development, validation and test periods be separated?
- How can fraud rules be evaluated beyond raw fraud capture?
- How do false positives, conversion impact and fraud loss interact?
- How should a fraud strategy be monitored after deployment?

The focus is primarily on **fraud strategy and decision rules**, not on building a machine-learning model.

---

## Tech stack

- **Databricks**
- **PySpark**
- **Spark SQL**
- **Delta Lake**
- **Unity Catalog**
- Python
- Git / GitHub

---

## Dataset

The project uses the synthetic **Credit Card Transactions Fraud Detection Dataset** available on Kaggle.

Main files:

- `fraudTrain.csv`
- `fraudTest.csv`

The dataset contains transaction-level information such as transaction ID, card ID, timestamp, merchant, category, amount, customer/location attributes and a fraud label.

The raw CSV files are **not stored in this repository**.

---

## Data architecture

The project follows a simplified medallion-style architecture inside the `fraud_lab` catalog.

```text
fraud_lab
│
├── bronze
│   ├── Volume: raw_files
│   │   ├── fraudTrain.csv
│   │   └── fraudTest.csv
│   ├── Delta: fraud_train
│   └── Delta: fraud_test
│
├── silver
│   ├── Delta: fraud_train
│   ├── Delta: fraud_test
│   └── Delta: transacoes_base
│
├── gold
│   ├── Delta: transacoes_features
│   └── Delta: decisoes_simuladas
│
└── monitoring
    ├── Volume: artifacts
    ├── Delta: casos
    ├── Delta: monitor_diario
    └── Delta: monitor_retrospectivo
```

### Layer responsibilities

**Bronze**  
Preserves data close to the original source, with minimal transformation.

**Silver**  
Standardizes names and types and applies data-quality validation.

**Gold**  
Will contain behavioral features, rule outputs and analytical datasets used for fraud-strategy decisions.

**Monitoring**  
Will store operational and retrospective monitoring outputs.

---

## Repository structure

```text
00_config.ipynb
01_setup-databricks.ipynb
02_ingestao_bronze.ipynb
03_qualidade_silver.ipynb
04_separacao_temporal.ipynb
```

### `00_config`
Centralizes catalog, schema, volume and table names used across notebooks.

### `01_setup-databricks`
Creates the Databricks project structure: catalog, schemas and volumes.

### `02_ingestao_bronze`
Loads the raw CSV files and validates the transaction-level granularity before persisting the Bronze tables.

Main checks include:

- required columns
- row and column counts
- uniqueness of transaction IDs
- repeated card behavior
- Bronze persistence in Delta format

### `03_qualidade_silver`
Creates the first curated transactional layer.

Main transformations and validations include:

- standardized field names
- timestamp conversion
- amount conversion
- fraud-label conversion
- missing or empty transaction IDs
- duplicated transaction IDs
- invalid timestamps
- invalid or negative amounts
- fraud labels outside the expected `0/1` domain
- Silver persistence in Delta format

### `04_separacao_temporal`
Validates the chronological relationship between development and test data and prepares the project for time-based strategy validation.

This is important because fraud rules should be developed using past information and evaluated on future transactions.

---

## Transaction granularity

A key validation in the project is distinguishing **transaction uniqueness** from **repeated card activity**.

```text
1 row = 1 transaction

transaction ID
    └── expected to be unique

card ID
    └── expected to repeat across multiple transactions
```

A repeated card is therefore not automatically a duplicated transaction. Its transaction history is precisely what will later allow behavioral analysis.

---

## Temporal validation

The supplied datasets follow a chronological sequence:

```text
past --------------------------------------------------------> future

fraudTrain.csv                         fraudTest.csv
┌───────────────────────────────┐      ┌──────────────────────┐
│ strategy development period   │      │ final test period    │
└───────────────────────────────┘      └──────────────────────┘
```

Within the development period, the project will further separate data into **development** and **validation** windows.

```text
DEV
investigate + create rules
        ↓
VAL
compare + tune candidates
        ↓
TEST
final evaluation on a future period
```

This avoids repeatedly tuning a rule on the same data used to judge its final performance.

---

## Roadmap

- [x] Databricks environment setup
- [x] Bronze ingestion
- [x] Transaction-granularity validation
- [x] Silver data-quality layer
- [x] Train/test chronological validation
- [ ] Development / validation temporal split
- [ ] Exploratory fraud investigation with SQL
- [ ] Behavioral and velocity features
- [ ] Fraud-rule candidates
- [ ] Backtesting
- [ ] Precision, recall and false-positive analysis
- [ ] Rule tuning
- [ ] Final out-of-time test
- [ ] Alert prioritization
- [ ] Strategy monitoring
- [ ] Fraud modality studies: Card Testing, ATO and Friendly Fraud
- [ ] Final fraud-strategy case

---

## Analytical principles

1. **Do not use future information to make a past decision.**
2. **Fraud detection is not only about maximizing fraud capture.**
3. **False positives and conversion impact must be measured.**
4. **A signal is not the same thing as proof of fraud.**
5. **Rules should be tested outside the period used to create them.**
6. **Operational impact matters alongside statistical performance.**

---

## Why this project

The project is a practical study of the lifecycle of an antifraud strategy:

```text
transactional data
        ↓
investigation
        ↓
fraud hypothesis
        ↓
rule candidate
        ↓
backtest
        ↓
performance / false positives
        ↓
threshold tuning
        ↓
future-period validation
        ↓
monitoring
```

The objective is to connect technical analysis in Databricks with the business reasoning required to make fraud-prevention decisions.

---

## Notes

This is a **learning and portfolio project** built with synthetic/public data.

It is not intended to reproduce the fraud strategy, rules, thresholds, data or internal systems of any specific company.
