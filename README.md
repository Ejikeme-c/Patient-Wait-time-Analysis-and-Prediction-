# Patient Wait Time Analysis and Prediction

I analysed 9,216 patient records to explore how gender, age, race, referral department and admission status relate to patient wait time. I then engineered additional features and built regression models to predict wait time in minutes.

The project covers data cleaning, exploratory data analysis, feature engineering, regression modelling and a report for a nontechnical audience.

**Main finding:** The available features explained very little of the variation in wait time. The selected regression model performed almost the same as predicting the average wait for every patient.

## Project objectives

- Prepare the patient data for analysis.
- Compare wait times across demographic and referral groups.
- Create features from admission dates, times and patient information.
- Predict wait time using regression and an independent test set.
- Check for overfitting and communicate the results clearly.

## Dataset

The input dataset is from **Kaggle** and is provided as `Kaggle_HealthData.csv`. It contains **9,216 records and 11 original columns**, including patient identifiers, admission dates and times, demographic information, referrals, admission status, satisfaction scores and wait times.

The outcome is **`Patient Waittime`**, measured in **minutes**. Observed waits range from **10 to 60 minutes**, with an average of **35.26 minutes** and a median of **35 minutes**.

The original Kaggle dataset URL and licence were not supplied with the file.

## Project files

The notebooks expect their input CSV files in the same directory. The layout below keeps the workflow easy to run from the repository root.

| File or folder | Purpose |
| --- | --- |
| [Health_Data_Cleaning.ipynb](Health_Data_Cleaning.ipynb) | Cleans the original dataset and exports the cleaned CSV. |
| [Health_Data_EDA_Feature_Engineering.ipynb](Health_Data_EDA_Feature_Engineering.ipynb) | Explores wait-time relationships, creates charts and engineers features. |
| [Health_Data_Regression.ipynb](Health_Data_Regression.ipynb) | Trains and evaluates regression models and checks for overfitting. |
| `Kaggle_HealthData.csv` | Original input dataset. |
| [Kaggle_HealthData_Cleaned.csv](Kaggle_HealthData_Cleaned.csv) | Output of the cleaning notebook. |
| [HealthData_Engineered.csv](HealthData_Engineered.csv) | Enriched data used by the regression notebook. |
| [HealthData_Predictors.csv](HealthData_Predictors.csv) | Illustrative encoded feature matrix exported during EDA. |
| [HealthData_Target.csv](HealthData_Target.csv) | Wait-time outcome exported during EDA. |
| `eda_visualizations/` | Six PNG figures created by the EDA notebook. |
| [EDA_Visualizations.zip](EDA_Visualizations.zip) | Downloadable collection of the EDA figures. |
| [Patient_Wait_Time_Report.pdf](Patient_Wait_Time_Report.pdf) | First-person report explaining the findings and model results. |

## Tools

I used Python, pandas, NumPy, Matplotlib, seaborn, scikit-learn and Jupyter Notebook. The regression workflow was run with Python 3.12 and scikit-learn 1.8.0.

## Getting started

### 1. Set up the environment

From the repository directory, create a virtual environment:

```bash
python -m venv .venv
```

Activate it on Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

Or on macOS/Linux:

```bash
source .venv/bin/activate
```

Install the notebook dependencies:

```bash
python -m pip install pandas numpy matplotlib seaborn "scikit-learn>=1.2" notebook
```

The minimum scikit-learn version supports the `sparse_output` option used by the encoder. Package versions can affect exact numerical results.

### 2. Add the input data

Place `Kaggle_HealthData.csv` beside the notebooks. Keep the generated CSV files in that directory as well.

### 3. Run the notebooks in order

```bash
jupyter notebook
```

Run all cells in each notebook in this sequence:

1. **Cleaning:** produces `Kaggle_HealthData_Cleaned.csv`.
2. **EDA and feature engineering:** produces `HealthData_Engineered.csv`, the illustrative predictor and target files, and the visualizations.
3. **Regression:** reads `HealthData_Engineered.csv` and displays model evaluation results, learning curves and prediction plots.

Each EDA visualization has a separate download cell. Run the cell and click the link to download its PNG. The final EDA export cell also creates a ZIP containing all six figures.

## Data preparation

In the cleaning notebook, I:

- Converted admission dates from day/month/year into datetime values.
- Converted admission times into time values displayed in 24-hour format.
- Removed hyphens from patient IDs while keeping them as strings.
- Renamed `Merged` to `pt name`.
- Counted missing values for every column.
- Removed `Patient Satisfaction Score` because it was not needed for the analysis and had **6,699 missing values, or 72.69%**.

In the EDA notebook, I labelled the **17 `Femaleemale` entries as `unknown`**. The recorded gender was unclear, so I did not assume a gender. I preserved the source label in `Gender original`.

I retained the literal `None` referral category as **no referral**, rather than treating it as a missing value. No patient records were removed during these cleaning steps.

CSV files store dates and times as text. The cleaned export uses `YYYY-MM-DD` for dates and `HH:MM:SS` for times. The notebooks restore the appropriate types when loading the data and read patient IDs as strings to preserve leading zeros.

## Exploratory findings

I used group summaries, boxplots, a wait-time distribution plot, age scatter plots and correlations to explore the data.

| Factor | Main finding |
| --- | --- |
| Gender | Average waits were approximately 35.1 minutes for Female, 35.4 for Male and 35.4 for unknown. The unknown group contained only 17 records. |
| Age | Age had almost no linear association with wait time. Pearson correlation was approximately -0.0056. |
| Race | Group averages ranged from approximately 34.6 to 35.7 minutes. |
| Referral | Average waits ranged from approximately 34.7 minutes for Renal to 36.8 minutes for Neurology. |
| Admission status | Average waits were approximately 35.0 minutes for Admission and 35.6 minutes for Not Admission. |

Each categorical comparison, including age groups, accounted for **less than 0.1% of the observed variation in wait time** on its own. Group distributions overlapped substantially. These descriptive comparisons do not establish cause and effect.

### Example visualizations

![Wait time by referral department](eda_visualizations/05_referral_wait.png)

![Age and patient wait time](eda_visualizations/03_age_wait.png)

The images are generated by the EDA notebook. They are also available in `EDA_Visualizations.zip`.

## Feature engineering

I created the following features:

| Feature | Purpose |
| --- | --- |
| Age group | Groups ages into 0-17, 18-34, 35-49, 50-64 and 65+. |
| Admission hour, weekday, month and year | Represents when the patient was admitted. |
| Weekend admission | Identifies Saturday and Sunday admissions. |
| Hour sine/cosine | Represents the repeating 24-hour cycle, keeping late-night and early-morning hours close together. |
| Weekday sine/cosine | Represents the repeating seven-day cycle. |
| Has referral | Distinguishes a recorded department referral from no referral. |
| Admitted | Encodes Admission as 1 and Not Admission as 0 for retrospective analysis. |
| One-hot categories | Represents gender, race, referral department and age group with separate 0/1 columns. |

I kept patient IDs, names and original gender labels out of the model. I also excluded admission status and `Admitted` because I could not confirm whether they were known before the patient finished waiting. No predictor was derived from wait time itself.

## Regression workflow

I split the data into **80% training and 20% testing**, using `random_state=42`:

- **Training:** 7,372 records.
- **Testing:** 1,844 records.

I compared ordinary linear regression with Ridge regression using five-fold cross-validation on the training set. The selected model was **Ridge regression with `alpha=100000`**.

The regression notebook reads `HealthData_Engineered.csv` rather than the pre-encoded EDA predictor file. Imputation, scaling and one-hot encoding are fitted inside a pipeline within each training fold. This keeps validation and test information out of preprocessing estimates.

I evaluated the selected model on the held-out test set and compared it with a baseline that always predicts the training-set average wait time.

## Model results

| Model | Dataset | MAE (mins) | RMSE (mins) | R² |
| --- | --- | ---: | ---: | ---: |
| Mean baseline | Training | 12.801 | 14.761 | 0.00000 |
| Mean baseline | Test | 12.595 | 14.634 | -0.00275 |
| Selected Ridge regression | Training | 12.799 | 14.759 | 0.00023 |
| Selected Ridge regression | Test | 12.592 | 14.632 | -0.00253 |

- **MAE** is the average size of the prediction errors, in minutes.
- **RMSE** also measures error in minutes but gives more weight to larger mistakes.
- **R²** describes how well the model explains differences in the outcome. Values near zero indicate little explanatory power. Negative values indicate performance worse than predicting the evaluated set's mean.

The regression model improved test RMSE over the training-mean baseline by only about **0.011%**. Its predictions stayed close to the overall average instead of capturing the wide differences in actual patient waits.

## Overfitting checks

I checked for overfitting by comparing training and test errors, reviewing cross-validation scores and plotting learning curves using only training data.

The selected model's training RMSE was **14.759 minutes**, compared with **14.632 minutes** on the test set. I found no large gap suggesting substantial overfitting. However, similar errors did not mean the model was useful. Performance remained close to the mean baseline, indicating limited predictive signal in the available features.

## Limitations and conclusion

This analysis used one random train/test split and did not test performance on future admissions. The descriptive group comparisons did not adjust for all possible influences, and some groups had relatively few records.

I could describe average wait times, but I could not reliably predict individual waits from these features. Further analysis could examine urgency, staffing levels and patient volume if those data become available. These factors were not included in this dataset.

The [PDF report](Patient_Wait_Time_Report.pdf) presents the findings with charts and explanations for a nontechnical audience.
