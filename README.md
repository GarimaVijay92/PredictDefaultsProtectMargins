# Predict Defaults, Protect Margins
This project focuses on building a predictive model to estimate the Probability of Default (PD) for small business loans using historical data from the U.S. Small Business Administration (SBA). The initiative supports regulatory compliance under the Basel Framework &amp; aids banks in optimizing loan approvals for profitability &amp; financial stability. This project was done as a part of California State University wide competition in 10 days.
# General Info
Small business loans inherently carry higher risk, so the aim of the project was to make accurate PD predictions essential for:

Bank Decision-Making: Reducing financial losses and determining suitable loan terms.
Regulatory Compliance: Meeting Basel III Endgame's capital adequacy standards.
Profit Maximization: Optimizing returns while minimizing defaults. Using SBA data spanning 1987–2014 (899,164 records, 27 features), we addressed challenges such as class imbalance, missing values, and data inconsistencies. A stratified sampling approach and robust data preprocessing pipelines ensured effective analysis and modeling. The CatBoost model emerged as the best performer, delivering an average profit of $7,588.25 with a custom-tuned threshold (0.41). It demonstrated balanced performance across sensitivity, specificity, and profitability, providing an effective tool for loan decision-making.
