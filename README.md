# Survival Churn Prediction

This repository contains the solution for the Ryt Bank Data Scientist Technical Assessment. The challenge was to analyze 6 months of customer data from a digital bank's payment product and develop a model to identify at-risk customers and understand churn drivers.

This project uses a **Random Survival Forest** model to predict *when* a customer is likely to churn, rather than just *if*. This approach correctly handles censored data (i.e., active users) and provides a more actionable 30-day risk forecast.

The complete analysis, model, and recommendations are available in the [survival_analysis.ipynb](survival_analysis.ipynb) notebook.

---

## 🚀 How to Run

This project uses `Python 3.10+`.

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/fasa79/survival-churn-prediction.git
    cd survival-churn-prediction
    ```

2.  **Create a virtual environment (recommended):**
    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
    ```

3.  **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

4.  **Add Data:**
    Create a new directory named `data` in the root of the repository. Place the following four CSV files (as specified in the assessment) inside this `data/` directory:
    * `customers.csv`
    * `transactions.csv`
    * `app_usage.csv`
    * `customer_service.csv`

5.  **Run the notebook:**
    ```bash
    jupyter notebook survival_analysis.ipynb
    ```
    *(Note: The notebook has been updated to read files from the `data/` directory. You will need to replace the original "Load Data" cell with the corrected code provided by the assistant).*

---

## 📈 Analysis & Key Findings (EDA)

Exploratory Data Analysis (EDA) was performed on the raw data to understand customer behavior and its relationship with churn. The key insights were:

* **Infant Churn:** The `account_age_days` histogram clearly shows that churn is heavily concentrated in the first 0-250 days of a customer's lifecycle. We are failing to activate and retain new users.
* **Activation is Key:** `avg_logins_count` and `total_transactions_count` were identified as the most powerful predictors of churn. Users who fail to log in or transact early are almost certain to churn.
* **Friction Points:** Customers with an **'Incomplete' KYC status** have a churn rate of over 50%. This is a critical friction point in the user journey.
* **Customer Service:** There is a strong correlation between low satisfaction scores (1 or 2) on support tickets and a higher likelihood of churn.
* **Tier & Engagement:** 'Premium' tier customers are the most loyal (sub-10% churn), while 'Basic' tier users churn the most (over 25%).

---

## 🤖 Predictive Model: Random Survival Forest

Instead of a simple binary classification (if a user will churn), we built a **Survival Analysis** model to predict *when* a user is likely to churn.

* **Model:** We used a **Random Survival Forest (RSF)** from the `scikit-survival` library.
* **Why RSF?** This model correctly handles **"right-censored" data**—the active customers in our dataset who have *not yet* churned. This allows us to use their "survival time" as valuable information, rather than just labeling them as "not churned."
* **Feature Engineering:** We built a robust preprocessing pipeline that creates point-in-time-correct features (e.g., `avg_logins_count` is only calculated *before* a user churns). This was critical to **prevent data leakage**, as simple averages would have told our model the answer.
* **Evaluation:** The model performs exceptionally well with a **C-Index of 0.976**. This high score is driven by its ability to accurately identify the "infant churn" cohort.
* **Output:** The model produces a 30-day churn probability forecast for every active customer, which is a highly practical and actionable output for the business.

---

## 📋 Deliverables Checklist

This project fulfills all the requirements outlined in the assessment PDF:

* [x] **EDA and insights:** (Cells 3-7 in the notebook)
* [x] **Predictive model with evaluation metrics:** (Cells 11-15, C-Index: 0.976)
* [x] **Top 5 actionable recommendations for retention:** (Included in the notebook)
* [x] **Brief write-up on deployment considerations:** (Included in the notebook)

---

## 💡 Recommendations Summary (details explanation in notebook)

Our recommendations focus on solving the primary business problem identified by the model: **"Failure to Activate"** new users.

1.  **Launch a "First 7-Day" Onboarding Campaign** to drive initial login and habit-building.
2.  **Incentivize the First Transaction** with a small, immediate reward (e.g., RM1 cashback).
3.  **Create a "New User At-Risk" Dashboard** using our model's 30-day forecast to identify and re-engage high-risk users *before* they churn.
4.  **Introduce Tier-Based Onboarding** to A/B test different engagement strategies for 'Basic' vs. 'Premium' users.
5.  **Promote "Sticky" Features** (like "Scheduled Payments") *after* a user's first transaction to build deeper engagement.

---

## ⚙️ Deployment Plan (details explanation in notebook)

A detailed, technical deployment plan is included in the notebook. The high-level summary is to build an MLOps pipeline using:

* **MLflow:** For experiment tracking and model registry.
* **GitLab CI/CD:** To automate testing and model training on merge-to-main.
* **Airflow:** To orchestrate a weekly batch prediction job that populates a `churn_forecasts` table for business use.