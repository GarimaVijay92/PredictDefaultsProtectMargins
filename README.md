# Predict Defaults, Protect Margins
This project focuses on building a predictive model to estimate the Probability of Default (PD) for small business loans using historical data from the U.S. Small Business Administration (SBA). The initiative supports regulatory compliance under the Basel Framework &amp; aids banks in optimizing loan approvals for profitability &amp; financial stability. This project was done as a part of California State University wide competition in 10 days.
# General Info
Small business loans inherently carry higher risk, so the aim of the project was to make accurate PD predictions essential for:

1. Bank Decision-Making: Reducing financial losses and determining suitable loan terms.
2. Regulatory Compliance: Meeting Basel III Endgame's capital adequacy standards.
3. Profit Maximization: Optimizing returns while minimizing defaults. Using SBA data spanning 1987–2014 (899,164 records, 27 features), we addressed challenges such as class imbalance, missing values, and data inconsistencies. A stratified sampling approach and robust data preprocessing pipelines ensured effective analysis and modeling. The CatBoost model emerged as the best performer, delivering an average profit of $7,588.25 with a custom-tuned threshold (0.41). It demonstrated balanced performance across sensitivity, specificity, and profitability, providing an effective tool for loan decision-making.

# Tools
Jupyter Notebook 

# Data Preparation
Data cleaning:

1. City and State Mapping: Missing values in the City and State fields were filled using the first three digits of the ZIP code and external mapping data from the ‘L002 3-Digit ZIP Code Prefix Matrix.pdf’ to accurately understand the business geography.

2. ZIP Code Standardization: ZIP codes with a length of 4 were prefixed with a 0 to make them 5 digits, adhering to the standard ZIP code format.

3. Approval Fiscal Year (ApprovalFY) Alphabet Removal: Alphabetic characters in ApprovalFY were replaced with empty strings to clean the data. Type Conversion: The column was converted to integers for consistency.

4. Standardized Numeric Columns: Columns such as BalanceGross, DisbursementGross, ChgOffPrinGr, GrAppv, and SBA_Appv were converted to numeric by removing dollar signs and commas.

5. LowDoc Cleanup: Errors in the LowDoc column (expected to contain 'Y' or 'N') were corrected. Loans with approval amounts less than "150,000 dollars" were flagged as "Low Docs".

6. Term Adjustment: Terms less than 6 months were deemed erroneous so we updated to 6 months.

7. MIS_Status Conversion: Converted MIS_Status to binary (1 for defaulted, 0 for paid in full) for easier modeling and analysis.

8. Records with missing values in key fields like BankState, DisbursementDate, State, and City were dropped.

9. Missing values in the NewExists field were filled with 0(means undefined)


New Fields

10. SBA Portion Calculation: A new field (SBA_portion) was created to represent the percentage of the approved gross amount covered by SBA.

11. Recession Indicator: A new field (recession) was created to flag loans with a maturity date between 2007-12-01 and 2009-06-30 as occurring during the recession.

12. Real Estate Indicator: Loans with a term greater than 240 months were flagged with a new RealEstate column, indicating they were backed by real estate.

13. Franchise Flag: A new field (If_Franchise) was created to indicate whether a loan was associated with a franchise (FranchiseCode not equal to 0 or 1).

14. Revolving Line of Credit: A new column (correct_RevLineCr) was created to flag loans as part of a revolving line of credit based on predefined codes.

15. Industry Consolidation: Industries based on the NAICS column were grouped into 7 major categories, consolidating the less frequent ones into an "Other" bucket to optimize category sizing and reduce complexity.

16. DefaultRateBucket: This column categorizes each state into buckets based on default rate levels.

17. CompanySize: Categorized businesses into size groups based on the number of employees.

18. Logic: 0: No employees (smallest businesses). 1: Micro-businesses (1–5 employees). 2: Small businesses (6–50 employees). 3: Medium-sized businesses (51–250 employees). 4: Large businesses (more than 250 employees).

19. Job Created - Group businesses by the number of jobs created
Logic: 0: No or minimal job creation (0–1 jobs). 1: Small job creation (2–10 jobs). 2: Moderate job creation (11–20 jobs). 3: Significant job creation (more than 20 jobs).

# Data Sampling
Stratified sampling was employed to reduce the dataset size while preserving the distribution of loan outcomes and critical features that were highly imbalanced. This approach ensures that the sample maintains the same proportions of key variables as in the original dataset, which is particularly important when dealing with imbalanced data, like loan defaults (MIS_Status) and other influential categorical variables.

# Defined Business Metric
We developed a custom business metric to evaluate financial impact:

Correct Approvals: 5% profit of loan amounts.
Incorrect Defaults: Losses at 5x loan amounts.
Denied Loans: 0

Equation:
denied_correctly = 0 # No profit or loss for denying defaults
denied_incorrectly = - 5 * 0.05 * disbursement_gross[false_negative].sum() # Loss for incorrectly granting defaults
approved_incorectly = 0 # No profit or loss for denying Paid in Full
approved_correctly = (0.05 * disbursement_gross[true_negative]).sum() # Profit for correctly granting Paid in Full
total_profit = denied_correctly + denied_incorrectly + approved_incorectly + approved_correctly

The make_scorer function in scikit-learn is used to create a custom scoring function that allowed us to define a performance metric tailored to the specific needs. business_scorer = make_scorer(business_metric).set_score_request(disbursement_gross=True)

To further optimize profitability, TunedThresholdClassifierCV from Scikit Learn is used to fine-tune the decision threshold based on a business-specific metric. This approach helped identify the optimal threshold, balancing sensitivity and profitability.

# Output and Explanation
We chose CatBoost model as the final model which effectively predicts default probabilities for small business loans, balancing accuracy and profitability.

The tuned threshold of 0.41 delivers higher sensitivity (93.22%), improving default identification and reducing losses.

Profitability increased to $7,588.25 on average, demonstrating its practical utility for decision-making.

Trade-Off Analysis The selection of a tuned threshold (0.41) was guided by the primary goal of maximizing profitability while maintaining strong predictive performance. This adjustment in the decision threshold demonstrates a deliberate trade-off between key metrics such as sensitivity, precision, and specificity to align the model's output with business objectives.

Prioritizing Sensitivity
Sensitivity, also known as the true positive rate, measures the model’s ability to correctly identify loan defaults.

A higher sensitivity at the tuned threshold (93.22% vs. 91.51% at 0.5) indicates that the model is better at flagging loans that are likely to default.

This improvement is crucial because missed defaults (false negatives) have the largest financial impact, representing loans that are incorrectly approved and result in substantial losses (5x the loan amount in this scenario).

By increasing sensitivity, the model effectively reduces the occurrence of false negatives, minimizing these high-cost mistakes. This prioritization directly translates into greater financial gains, as evidenced by the increase in the business metric (profit improved by $33.27 on average).

Accepting Reductions in Precision and Specificity

Precision measures how many predicted defaults were actual defaults. At the tuned threshold, precision dropped slightly from 70.29% to 66.09%, reflecting a higher number of false positives (loans flagged as defaults but not actually defaulting).

Specificity, which measures the ability to correctly identify non-defaults (loans likely to be fully paid), also saw a minor decrease from 91.91% to 89.99%.

These reductions indicate that the model occasionally misclassifies some loans as defaults that might have been successfully repaid. However, the financial impact of such false positives is negligible compared to the cost of missed defaults. Denying these loans has no associated profit or loss in the business metric, making this trade-off acceptable.

CatBoost clearly stands out with an exceptional ROC-AUC of 0.9713, indicating outstanding model performance.
It outperforms other models not just in ROC-AUC, but also delivers the highest profitability of 7,588.25 dollars, making it the best for business purposes.
It balances high accuracy, good sensitivity, and high specificity, which means it is both effective and reliable for predicting defaults without many false positives.
