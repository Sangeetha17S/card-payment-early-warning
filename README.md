# Early Warning for Card Payments

## 1. The problem in your own words

The goal of this project is to help a credit-card collections team identify customers who are more likely to miss their next month's payment, so the team can prioritize early outreach.

The solution provides:

- A ranked list of higher-risk customers.
- A simple and explainable payment-delay risk score.
- Short, data-grounded explanations for the 20 highest-risk customers.
- A human-review step before any collection decision is made.

The project uses the UCI Default of Credit Card Clients dataset, which contains 30,000 customers and payment history from April to September 2005.

---

## 2. Assumptions

- Each row represents one customer.
- `default_next_month` is the target indicating whether the customer defaulted in the following month.
- The six payment-status columns represent the recorded payment status for April through September.
- Payment-status values 1–8 are treated as documented payment-delay values based on the UCI documentation.
- Payment-status values -2 and 0 are retained because their meanings are not explicitly defined in the source documentation.
- Undocumented education and marriage codes are retained rather than assigning unsupported meanings.
- Negative bill amounts are retained because the source documentation does not provide a definitive correction rule.
- The points-based risk score is intended for prioritization, not as a guarantee that a customer will default.
- The LLM is used only to generate supporting explanations for already-selected high-risk customers. It does not determine the risk ranking.
- LLM-generated notes are checked against the original customer data before being used.

---

## 3. Data source and how to run

### Data source

The dataset used is the UCI Machine Learning Repository **Default of Credit Card Clients** dataset.

It contains 30,000 customer records with payment history from April to September 2005.

The original Excel dataset is kept outside the GitHub repository and is excluded through `.gitignore`.

### Tools and packages

The project was developed and executed in Google Colab using:

- Python
- Pandas
- Matplotlib
- Scikit-learn
- Groq
- `openai/gpt-oss-20b`

### How to run

1. Download the **Default of Credit Card Clients** dataset from the UCI Machine Learning Repository.
2. Open the notebook in Google Colab.
3. Create a folder named `Caspad Project` in Google Drive.
4. Upload `default of credit card clients.xls` into that folder.
5. Run the notebook cells from top to bottom.
6. The notebook performs:
   - Part A — Data Engineering
   - Part B — Analytics and Risk Ranking
   - Part C — LLM-generated explanations and factuality checking
   - Optional Analytics — Logistic Regression comparison
7. For Part C, add the Groq API key to Google Colab Secrets using the name `GROQ_API_KEY`.
8. The dataset and API keys are not stored in the GitHub repository.

---

## 4. Part A

### Data preparation

The original dataset columns were renamed to clearer names, including:

- `customer_id`
- `credit_limit`
- `education`
- `marriage`
- `age`
- `pay_status_sep`
- `pay_status_aug`
- `pay_status_jul`
- `pay_status_jun`
- `pay_status_may`
- `pay_status_apr`
- `bill_amt_sep`
- `pay_amt_sep`
- `default_next_month`

### Data quality checks

| Check | Finding | Decision |
|---|---:|---|
| Customer ID | 30,000 unique IDs among 30,000 rows | No change |
| Target | Only 0 and 1 | No change |
| Missing values | No missing values in the 25 columns | No treatment |
| Duplicate rows | 0 | No removal |
| Education | 345 records use undocumented codes 0, 5, or 6 | Retain and document |
| Marriage | 54 records use undocumented code 0 | Retain and document |
| Payment status | 25,939 customers have at least one -2 or 0 | Retain and document |
| Negative bill amounts | 1,930 customers have at least one negative bill amount | Retain and document |

### Undocumented codes

The dataset contains some codes whose meanings are not explicitly defined in the source documentation.

I retained these original values rather than converting them to missing values because an undocumented value does not necessarily mean that the value is missing.

No records were deleted or altered based only on these undocumented codes.

### Negative bill amounts

Negative bill amounts were found across the six monthly bill-amount fields.

The source documentation does not provide a definitive explanation or correction rule for these values, so they were retained as recorded rather than replaced or removed.

---

## 5. Part B

### Overall default rate

The overall default rate was **22.12%**.

This provides the baseline for comparing different customer groups.

### Default rate by credit-limit band

Five credit-limit bands were analyzed.

| Credit-limit band | Customers | Default rate |
|---|---:|---:|
| Band 1: NT$10,000–50,000 | 7,676 | 31.79% |
| Band 2: >NT$50,000–100,000 | 4,822 | 25.80% |
| Band 3: >NT$100,000–180,000 | 6,123 | 19.86% |
| Band 4: >NT$180,000–270,000 | 5,421 | 16.86% |
| Band 5: >NT$270,000–1,000,000 | 5,958 | 13.80% |

The observed default rate decreased from **31.79% in Band 1 to 13.80% in Band 5**.

This is an observed association in the dataset and is not interpreted as a causal relationship.

### Default rate by latest payment status

The latest payment status is September.

The strongest observed pattern was:

- Status 0: **12.81%**
- Status 1: **33.95%**
- Status 2: **69.14%**
- Status 3: **75.78%**

Statuses 4–8 also had high observed default rates, but these groups contained relatively few customers and were therefore interpreted cautiously.

### Points-based risk score

A simple six-month payment-delay risk score was created using the payment-status fields from April to September.

Only positive payment-status values from 1–8 contribute risk points.

Values below 0 and 0 contribute zero points.

The `default_next_month` target was not used when calculating the score or selecting the high-risk customers.

The score is the sum of the six monthly payment-status values after values below zero are treated as zero.

Higher scores indicate a greater amount of documented payment-delay history across the six months.

The 20 highest-risk customers were selected for Part C.

The highest risk score was **36**, followed by 19 customers tied at **33**.

### Top 20% risk capture

The dataset contains 30,000 customers, so the top 20% contains **6,000 customers**.

There were **6,636 total defaulters**, of which **3,099** were included in the top 20% risk group.

Therefore, the top 20% risk group captured **46.70% of all observed defaulters**.

This is an observed result from this dataset and should not be interpreted as a guarantee of future performance.

---

## 6. Part C

### LLM approach

The 20 customers with the highest payment-delay risk scores were selected.

The LLM received only relevant customer fields:

- Customer ID
- Credit limit
- Age
- Six monthly payment-status values
- Six monthly bill amounts
- Six monthly payment amounts

The `default_next_month` target was not provided to the LLM.

The model used was **`openai/gpt-oss-20b` through Groq**.

The LLM was asked to generate a concise two-line collections-support note using only facts explicitly present in the supplied customer data.

The prompt instructed the model to:

- Use only facts present in the supplied data.
- Focus on payment-status history, bill amounts, and payment amounts.
- Treat payment-status values 1–8 as documented payment delays.
- Avoid assigning unsupported meanings to -2 and 0.
- Use NT$ for monetary amounts.
- Avoid assumptions about income, employment, financial hardship, or personal circumstances.
- Avoid stating that a customer will definitely default.
- Produce exactly two lines.

### Factuality check

All 20 generated notes were checked against the original customer data supplied to the LLM.

Results:

- **20 notes reviewed**
- **17 notes without identified issues**
- **3 notes with identified issues**
- **85.0% without identified inconsistency**
- **15.0% issue rate**

Examples of identified issues:

- **Customer 11555:** The note mentioned a NT$15 April payment but later stated that payment amounts remained zero. The actual April payment was NT$15.
- **Customer 13713:** The note stated that there were no payments from April to August but also mentioned a NT$70 July payment. The actual July payment was NT$70.
- **Customer 750:** The note stated that there were zero payments in April but also mentioned a NT$3,000 April payment. The actual April payment was NT$3,000.

This shows that LLM-generated explanations can be useful, but they require factual verification against the underlying customer data.

---

## 7. Limitations and what you would do next

### Limitations

- The dataset contains historical information from April–September 2005, so the observed patterns may not represent current customer behavior.
- Some source codes are undocumented.
- Negative bill amounts are present and do not have a definitive explanation in the source documentation.
- The points-based score is intentionally simple and does not capture every factor associated with default.
- The top-20% capture result is based on this dataset and should not be treated as a guarantee of future performance.
- The LLM generated three notes containing factual inconsistencies.
- The analysis identifies observed associations and does not establish causation.
- The optional logistic regression result is based on one stratified train-test split and should be validated on additional data before production use.

### What I would do next

- Test the approach on more recent and representative customer data.
- Evaluate performance using a separate time-based validation period.
- Monitor the stability of the risk score over time.
- Compare additional interpretable models.
- Add automated factuality checks for LLM-generated notes.
- Keep human review in the collection decision process.

---

## 8. How you used AI tools

### Groq / LLM

Groq was used for Part C with the `openai/gpt-oss-20b` model.

The model generated concise two-line collections-support notes for the 20 highest-risk customers using only the selected customer fields.

The generated notes were checked against the original customer data, and all identified inconsistencies were reported.

### ChatGPT

ChatGPT was used during development as a supporting tool for understanding the project requirements, discussing analysis approaches, troubleshooting code, and improving explanations and documentation.

The final analysis was implemented and executed in the Google Colab notebook, and the results reported in this README come from the executed project notebook.

---

## 9. Optional extra

### Analytics — Logistic Regression vs Points Score

The optional Analytics track compares logistic regression with the simple points-based risk score using **top-20% capture**.

The model uses:

- 15 numerical features, including credit limit, age, bill amounts, and payment amounts.
- 6 payment-status features treated as categorical variables.
- StandardScaler for numerical features.
- OneHotEncoder for payment-status features.
- Logistic Regression.

An **80/20 stratified train-test split** was used with `random_state=42`.

This produced:

- Training set: **24,000 customers**
- Test set: **6,000 customers**
- Test-set defaulters: **1,327**
- Top 20% of test set: **1,200 customers**

The logistic regression model generated a predicted probability of default for each test customer. Customers were ranked by predicted probability, and the top 20% were evaluated.

### Results

| Method | Defaulters captured | Top-20% capture |
|---|---:|---:|
| Logistic Regression | 651 | **49.06%** |
| Points-based score | 595 | **44.84%** |

Logistic regression captured **4.22 percentage points more** test-set defaulters than the points-based score.

The comparison was performed on the **same 6,000-customer test set and the same top-20% group size**, making the comparison directly comparable.

Logistic regression performed better on this test set, while the points-based score remained simpler and easier to explain.
