# Black Friday purchases

This project asks which groups have higher observed spend in a public retail file, and whether person attributes can predict an individual amount.

It can describe group differences. It cannot price a future customer, and it does not estimate the effect of marketing.

## Question and scope

1. On one customer-product record, what does `Purchase` look like?
2. Across customers, which groups have a higher total of observed `Purchase`?
3. If the product category is already known, how much does a regression add beyond the category's typical amount?

Out of scope: future spend, lifetime value, and causal effects of advertising.

## Data

Each row is one customer-product purchase record, not a proven full basket or store transaction.

| Column | Meaning in this file |
| --- | --- |
| `User_ID` | Customer identifier. Not a predictor. |
| `Product_ID` | Product identifier. |
| `Product_Category` | Category code on that record. |
| `Purchase` | Amount on that record, in undocumented monetary units. |
| `Gender`, `Age`, `Occupation`, `City_Category`, `Stay_In_Current_City_Years`, `Marital_Status` | Person attributes. Each is constant within a `User_ID`. |

`Stay_In_Current_City_Years` includes `4+`, which means four or more years, not exactly four.

The file has 550,068 records, 5,891 customers, and 3,631 products. There are no missing values and no duplicate rows.

This schema matches the public Black Friday retail dataset circulated from the Analytics Vidhya hackathon and Kaggle mirrors such as [sdolezel/black-friday](https://www.kaggle.com/datasets/sdolezel/black-friday). This copy has one `Product_Category` column. The common public file also has `Product_Category_2` and `Product_Category_3`.

Limits:

- The repository has no original data dictionary.
- The file has no currency field and no date. Some republished notes call the amount US dollars and the setting Black Friday. This project does not treat either claim as confirmed. Amounts are monetary units.
- The file name says Walmart. That does not make these official Walmart records.
- Results describe this sample only.

## Units of analysis

1. Amount on one record: `Purchase`.
2. Total observed spend for a customer: sum of `Purchase` for that `User_ID` in this file.
3. Future spend or lifetime value: not in the file, and not estimated.

Group comparisons use one row per customer, so repeated records from the same person are not treated as independent observations.

## How to run

From this folder:

```bash
pip install -r requirements.txt
```

Open `study.ipynb` and run all cells from the top. The first code cell reads `Data/walmart_data.csv`. Saved outputs match that run.

## Statistical method

Group questions use Welch intervals and tests for the difference of means. The confidence level is 90%, so the two-sided significance level is 0.10. Welch does not assume equal variances. It does assume independent customers and an approximately normal mean within each group.

Individual intervals for each group's mean are not used to decide whether two groups differ. Overlap of those intervals is not a test. A difference interval that contains zero is not proof that the groups are equal.

The six age contrasts are one family. Their p-values are Holm-adjusted. The intervals printed for those contrasts are still unadjusted 90% intervals. The comparisons were chosen on the same file, so they are exploratory.

## Data split and leakage

Customers are split 80/20 with `random_state=42`.

- Customer model: one row per `User_ID`.
- Record model: every record from a customer stays on the same side.
- The executed notebook reports 0 shared customers.
- Encoders are fit on training rows only. Unknown categories become zeros.
- `User_ID` is not a feature.
- Evaluation `Purchase` values are not used to build features or category medians.
- This split was available while the analysis was written. It is a development evaluation, not an untouched final test, and not a forecast of future sales.

## Model comparison

MAE is the average absolute miss on the held-out customers or records.

The training-mean baseline is kept. A training-median baseline is added because the median minimizes total absolute error on the sample used to compute it. That does not guarantee a lower error on held-out rows.

On one record, a third baseline predicts the training median of `Purchase` for that `Product_Category`. An unseen category would use the overall training median. Category is a fair input only when it is already known, for example after the customer has chosen the item. It is not a fair input for the total spend of someone who has not bought yet.

### Customer total observed spend

Held-out customers: 1,179. Training customers: 4,712.

| Model | MAE | Versus training mean | Versus training median |
| --- | ---: | ---: | ---: |
| Training mean | 696,879 | 0 | 7.4% worse |
| Training median | 648,844 | 6.9% lower MAE | 0 |
| Linear regression | 630,397 | 9.5% lower MAE | 2.8% lower MAE |

The regression beats both baselines. The gain over the median is small: about 18,447 monetary units of MAE. Most of the error remains. Person attributes do not measure what one customer will spend.

### One customer-product record

Held-out records: 116,592. Training records: 433,476. No shared customers.

| Model | MAE | Versus training mean | Versus category median |
| --- | ---: | ---: | ---: |
| Training mean | 4,069 | 0 | 1,833 worse |
| Training median | 3,912 | 3.9% lower MAE | 1,676 worse |
| Regression, person columns only | 4,045 | 0.6% lower MAE | 1,809 worse |
| Category training median | 2,236 | 45.1% lower MAE | 0 |
| Regression, person plus category | 2,304 | 43.4% lower MAE | 68 worse |

Person columns do not beat the training median. The category median beats the regression that also uses person columns, by about 68 monetary units of MAE. Once the category is known, the simple category rule is enough. The regression is mostly rediscovering that rule.

## Group differences

Welch 90% intervals for the difference of customer total spend. Age p-values are Holm-adjusted. Intervals are unadjusted.

| Contrast | Difference | 90% interval | Read |
| --- | ---: | --- | --- |
| Men minus women | 213,320 | 172,311 to 254,329 | Excludes zero |
| City A minus B | 20,453 | -61,450 to 102,357 | Includes zero. Not evidence that A and B match |
| City A minus C | 729,738 | 658,545 to 800,931 | Excludes zero |
| City B minus C | 709,285 | 665,012 to 753,557 | Excludes zero |
| Unmarried minus married | 37,049 | -3,830 to 77,928 | Includes zero (p about 0.14). Not evidence that the groups match |
| Age 26–35 minus each other band | 109,994 to 449,962 | All exclude zero | Holm-adjusted p-values stay below 0.10 |

Categories 1, 5, and 8 carry most observed spend. Category 1 is about 28.5% of women's spend and about 40.2% of men's spend.

## What this supports, and what it does not

Supports: in this file, men have higher total observed spend than women; age 26–35 is higher than the other age bands; cities A and B are higher than C; categories 1, 5, and 8 carry the record amounts.

Does not support: that overlapping groups are equal; that marital status does not matter in every sense; that person attributes can price a customer; that a regression with category is better than the category median; that these patterns are effects of marketing; that the amounts are confirmed dollars or official Walmart sales; that the same gaps would appear in a later period.

## Next steps that match these limits

- Confirm currency, dates, and whether category 2 and 3 were dropped from this copy.
- Freeze a final split before any further model changes. The split above is already a development check.
- If the decision is "what will this chosen item cost," start from the category median and only keep a model that beats it on a new split.
- If the decision is "what will a new customer spend before any purchase," person attributes are a weak signal. More columns from this file will not create a future time period.

## Files

```text
black-friday-purchases/
│
├── study.ipynb
│   └── Question, method, result, and interpretation, with saved outputs
│
├── Data/walmart_data.csv
│   └── Customer-product purchase records
│
├── requirements.txt
│   └── Libraries needed to run the notebook
│
└── README.md
    └── This summary
```
