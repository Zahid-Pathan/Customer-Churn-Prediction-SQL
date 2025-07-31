# 📊 Customer Churn Prediction Pipeline (SQL + ML)

This project demonstrates a complete **machine-learning pipeline** for predicting customer churn, using **SQL** for feature engineering and **Python** for modelling and evaluation. All data are synthetic, generated to resemble realistic customer-subscription behaviour.

**Customer churn**, also known as customer attrition, refers to the rate at which customers stop doing business with a company over a specific period. It essentially measures the percentage of customers who discontinue their relationship with a business, whether through cancellation of a subscription, non-renewal of a contract, or simply ceasing to purchase products or services. 

---

## 📁 Project Structure

```text
├── 1_data_generation.ipynb        # Creates synthetic SQLite DB with raw data
├── 2_data_cleaning_etl.ipynb      # SQL-based feature engineering and ETL
├── 3_ml_pipeline.ipynb            # ML models, evaluation metrics, visualisations
├── cleaned_churn_data.csv         # Final ML‑ready dataset
├── customer_churn.db              # SQLite database with all tables
└── read.md                        # Project overview and documentation
```

---

## 🎯 Objective

Predict whether a customer will churn based on their

* Activity logs (login / stream frequency)
* Payment‑success behaviour
* Subscription plan type and country

---

## 🗃️ Synthetic Datasets

| Table               | Description                                                        |
|---------------------|--------------------------------------------------------------------|
| `customers`         | Customer profiles: plan, country, signup date                      |
| `activity_logs`     | Events such as `login`, `stream`, `cancel`, with timestamps        |
| `payments`          | Payment records: amount, success flag, date                        |
| `churned_customers` | Subset of customers labelled as churned (with churn date)          |

---

## 🧹 Step 1 – SQL Feature Engineering

### Core SQL Query (run in `2_data_cleaning_etl.ipynb`)

```sql
SELECT
    c.customer_id,
    c.plan_type,
    c.country,
    COUNT(DISTINCT a.event_date)                                             AS active_days,
    SUM(CASE WHEN p.success = 1 THEN 1 ELSE 0 END)                          AS successful_payments,
    MAX(p.payment_date)                                                     AS last_payment_date,
    CASE WHEN ch.customer_id IS NOT NULL THEN 1 ELSE 0 END                  AS churned
FROM customers            c
LEFT JOIN activity_logs   a  ON c.customer_id = a.customer_id
LEFT JOIN payments        p  ON c.customer_id = p.customer_id
LEFT JOIN churned_customers ch ON c.customer_id = ch.customer_id
GROUP BY c.customer_id, c.plan_type, c.country, ch.customer_id;
```

#### 🔍 Explanation

| Feature                | Rationale                                                                                                   |
|------------------------|--------------------------------------------------------------------------------------------------------------|
| `active_days`          | Measures engagement frequency (count of distinct activity days).                                            |
| `successful_payments`  | Captures payment reliability and revenue contribution (number of successful payments).                       |
| `last_payment_date`    | Enables calculation of payment recency downstream (most recent payment timestamp).                           |
| `churned`              | Binary target label built via `LEFT JOIN` to `churned_customers` (1 = churned, 0 = active).                  |

All joins are **LEFT** to keep every customer, even if they have no logs or payments.  
`GROUP BY` aggregates rows so each customer appears exactly once.

---

## 🤖 Step 2 – Machine Learning (in `3_ml_pipeline.ipynb`)

### Models Implemented

* **Logistic Regression** (`class_weight="balanced"`)
* **Random Forest** (`class_weight`, interpretable)
* **Gradient Boosting** (baseline boosting model)

### Evaluation Metrics

* Accuracy, Precision, Recall, F1‑Score
* ROC AUC
* Confusion Matrix
* Visualisations with **Matplotlib** + **Seaborn**

---


## 🚀 Future Improvements

* Add temporal features (recency, frequency, monetary – RFM).
* Apply **SMOTE** to oversample the minority (churn) class.
* Tune hyper‑parameters via `GridSearchCV`.
* Deploy an interactive **Streamlit** dashboard for live monitoring.

---

## ⚙️ Installation

```bash
pip install pandas numpy matplotlib seaborn scikit-learn faker imbalanced-learn
```

---
