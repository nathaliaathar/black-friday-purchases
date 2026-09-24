# Black Friday purchases

This project studies Walmart Black Friday purchases. It starts from one row per purchase, then switches to one row per customer, and ends by testing what can actually be predicted.

The question is who spends more, and whether gender, age, city, and marital status are enough to say how much a customer will pay.

## Files

```text
black-friday-purchases/
│
├── study.ipynb
│   └── The analysis, in the order it was done
│
├── Data/walmart_data.csv
│   └── One row per purchase
│
├── requirements.txt
│   └── Python libraries needed to run the notebook
│
└── README.md
    └── This summary
```

`study.ipynb` is the analysis. Read the markdown above each block, run the code, then read the observation underneath it.

## How to run

From this folder:

```bash
pip install -r requirements.txt
```

Open `study.ipynb` and run the cells from top to bottom. The first code cell loads `Data/walmart_data.csv`.

## Path of the analysis

1. **Inspect each purchase.** Look at the first and last rows, the shape, the columns, the value types, and `describe`. Clean `Stay_In_Current_City_Years` by removing `+` and storing it as a number.
2. **Check quality.** Count missing values and duplicate rows.
3. **Plot the distributions.** Histograms for occupation, years in the city, marital status, and purchase amount. Bar charts for the category columns.
4. **Switch to the customer.** Sum `Purchase` by `User_ID`. A row count is not spend per person. Compute 90% confidence intervals for city, age, and marital status, and for the male mean minus the female mean.
5. **See where the money goes.** Share of spend by product category inside each gender, and a table of purchase count versus spend per customer.
6. **Predict a customer's total spend** from gender, age, city, and marital status. Product category stays out, because it is part of the spend. Test on customers the model has not seen. Compare with predicting the training mean (MAE).
7. **Predict one purchase** after the product category is known, still splitting whole customers so one person is not on both sides.

## What the data shows

There are 550,068 purchases and 5,891 customers. No missing values. No duplicate rows.

A single purchase has median about $8,047 and mean about $9,264. The mean sits above the median because of a long right tail. The minimum is $12 and the maximum is $23,961. About 0.5% of purchases sit above the upper whisker.

Product category moves a single purchase the most. Occupation barely moves it. More purchase rows belong to men, to age 26–35, and to city B. Those are counts, not spend per customer.

At customer level, using a 90% confidence interval:

- Men spend more per customer than women. The intervals do not overlap. The interval for the difference (male mean minus female mean) is about $172,442 to $254,198, entirely above zero.
- Cities A and B overlap. City C sits entirely below both. C has the most customers and a much lower total per customer.
- Age 26–35 sits above every other age band.
- Unmarried and married intervals overlap. Marital status does not separate spend.

Categories 1, 5, and 8 carry most of the spend. Category 1 is about 40% of male spend and about 28% of female spend. Women spread more evenly across 1, 5, and 8.

## What can be predicted

**Customer total spend**, using only gender, age, city, and marital status:

- Predicting the training mean misses by about $696,879 per held-out customer.
- The linear regression misses by about $630,397.
- That removes about 9.5% of the error. Most of the error remains.

The associations, versus woman, age 0–17, and city A, are about $189,901 more for men, $198,997 more for age 26–35, $25,309 more for city B, $664,703 less for city C, and $14,436 less for married customers. These are associations, not causes.

**One purchase:**

- Person columns alone remove about 0.75% of the error (about $4,061 versus a baseline of about $4,091).
- Adding product category removes about 43.6% of the error (about $2,308 left).
- Category gives the price band of that purchase. It is not a fixed shelf price, and about $2,308 of error remains. This does not estimate a new customer's total spend.

## Signals, and what not to claim

The higher-spend profile in this file is a man, aged 26–35, in city A or B. City C is a large group of much smaller customer totals. Marital status is not a useful split.

These columns do not measure how much a customer will pay. Do not read the results as “buy more advertising.” The file shows who already spends more. It does not show that more ads would raise sales.

## Stack

Python, pandas, NumPy, matplotlib, seaborn, Plotly, SciPy, and scikit-learn.
