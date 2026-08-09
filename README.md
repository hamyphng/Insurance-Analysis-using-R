# Health Insurance Charges Analysis


>  All figures in this README are rendered from the included `insurance.csv` and stored in `figures/`.


> Exploratory data analysis and linear regression study of the factors associated with annual health insurance charges.

## Overview

This project analyzes **1,338 individual health insurance records** to understand which demographic and lifestyle characteristics are most strongly associated with annual medical charges. The dataset contains age, sex, BMI, number of children, smoking status, region, and annual insurance charges.

The analysis progresses from descriptive statistics and visualization to pairwise relationships, simple linear regression, and multiple linear regression.

### Objectives

- Describe and explore the variables in the insurance dataset.
- Visualize the distributions of quantitative and categorical variables.
- Identify factors significantly associated with annual medical charges.
- Examine relationships between predictors and charges.
- Build and interpret simple linear regression models.
- Build a multiple linear regression model and simplify it by removing predictors with limited statistical contribution.

---

## Dataset

The dataset contains **1,338 observations** and **7 variables**, with **no missing values**.

| Variable | Type | Description |
|---|---|---|
| `age` | Quantitative | Age of the insured individual |
| `sex` | Categorical | Sex of the insured individual |
| `bmi` | Quantitative | Body Mass Index (kg/m²) |
| `children` | Quantitative / discrete | Number of dependent children |
| `smoker` | Categorical | Smoking status (`yes` / `no`) |
| `region` | Categorical | Region of residence |
| `charges` | Quantitative | Annual medical charges in USD |

The response variable is **`charges`**.

---

## Project Structure

```text
.
├── insurance.csv
├── README.md
├── analysis.R / analysis.Rmd
└── figures/
    ├── children_distribution.png
    ├── age_distribution.png
    ├── bmi_distribution.png
    ├── charges_distribution.png
    ├── sex_distribution.png
    ├── smoker_distribution.png
    ├── region_distribution.png
    ├── scatterplot_matrix.png
    ├── charges_vs_age.png
    ├── charges_vs_bmi.png
    ├── charges_vs_children.png
    └── charges_by_categories.png
```

> The `figures/` paths above are the recommended filenames if plots are exported for GitHub display.

---

## 1. Data Loading and Inspection

```r
data <- read.csv("insurance.csv")

str(data)
dim(data)
head(data)
sum(is.na(data))

attach(data)
summary(data)
```

### Dataset summary

- Rows: **1,338**
- Columns: **7**
- Missing values: **0**
- Mean annual charge: approximately **$13,270**
- Median annual charge: approximately **$9,382**
- Age range: **18–64 years**
- Mean BMI: approximately **30.7 kg/m²**

The large difference between the mean and median charges indicates a strongly right-skewed response distribution.

---

# 2. Exploratory Data Analysis

## 2.1 Number of Children

```r
summary(children)

par(mfrow = c(1, 2))

barplot(
  table(children),
  col = "#ced1f8",
  main = "Number of Children",
  xlab = "Children",
  ylab = "Count",
  ylim = c(0, 600)
)

boxplot(
  children,
  col = "#ced1f8",
  main = "Boxplot of Children",
  xlab = "Children"
)
```

![Distribution and boxplot of number of children](figures/children_distribution.png)

Most individuals have no dependent children: **574 of 1,338 (~43%)**. The median is 1 and the mean is approximately 1.09.

---

## 2.2 Age Distribution

```r
summary(age)

par(mfrow = c(1, 2))

hist(
  age,
  nclass = 30,
  col = "lightblue",
  border = "white",
  main = "Distribution of Age",
  xlab = "Age (years)",
  ylab = "Frequency"
)

boxplot(
  age,
  col = "lightblue",
  main = "Boxplot of Age",
  xlab = "Age (years)"
)
```

![Age histogram and boxplot](figures/age_distribution.png)

Age is distributed fairly uniformly between 18 and 64 years. The mean and median are both close to 39 years, with no strong evidence of extreme age outliers.

---

## 2.3 BMI Distribution

```r
summary(bmi)

par(mfrow = c(1, 2))

hist(
  bmi,
  nclass = 30,
  col = "lightpink",
  border = "white",
  main = "Distribution of BMI",
  xlab = "BMI (kg/m²)",
  ylab = "Frequency"
)

boxplot(
  bmi,
  col = "lightpink",
  main = "Boxplot of BMI",
  xlab = "BMI (kg/m²)"
)
```

![BMI histogram and boxplot](figures/bmi_distribution.png)

BMI is approximately normally distributed around **30.7 kg/m²**, although several high-BMI observations appear above roughly 45 kg/m².

---

## 2.4 Charges Distribution

```r
summary(charges)

par(mfrow = c(1, 2))

hist(
  charges,
  nclass = 50,
  col = "#f9e1a8",
  border = "white",
  main = "Distribution of Charges (USD)",
  xlab = "Annual charges (USD)",
  ylab = "Frequency"
)

boxplot(
  charges,
  col = "#f9e1a8",
  main = "Boxplot of Charges (USD)",
  xlab = "Annual charges (USD)"
)
```

![Charges histogram and boxplot](figures/charges_distribution.png)

Insurance charges are strongly right-skewed. Most observations are concentrated at lower costs, while a smaller group has substantially higher annual charges. The original analysis also identifies a secondary concentration around **$30,000–$40,000**.

---

# 3. Categorical Variables

## 3.1 Sex

```r
barplot(
  table(sex),
  col = c("#f9c5c7", "#b6d8f7"),
  main = "Distribution of Sex",
  xlab = "Sex",
  ylab = "Count",
  ylim = c(0, 800)
)
```

![Distribution of sex](figures/sex_distribution.png)

The sample is almost perfectly balanced:

- Male: **676 (50.5%)**
- Female: **662 (49.5%)**

---

## 3.2 Smoking Status

```r
barplot(
  table(smoker),
  col = c("#cde9dc", "#ffdab4"),
  main = "Distribution of Smoker Status",
  xlab = "Smoker",
  ylab = "Count",
  ylim = c(0, 1200)
)
```

![Distribution of smoking status](figures/smoker_distribution.png)

Smoking status is imbalanced:

- Non-smokers: **1,064 (79.5%)**
- Smokers: **274 (20.5%)**

Despite representing only about one fifth of the dataset, smokers account for a major share of the variation in medical charges.

---

## 3.3 Region

```r
barplot(
  table(region),
  col = c("#ffb3ba", "#ffdfba", "#ffffba", "#baffc9"),
  main = "Distribution of Region",
  xlab = "Region",
  ylab = "Count",
  ylim = c(0, 400)
)
```

![Distribution by region](figures/region_distribution.png)

The four regions are relatively balanced. The southeast has the largest number of observations (**364**), while the remaining regions each contain roughly 325 observations.

---

# 4. Relationships Between Variables

## 4.1 Scatterplot Matrix

```r
quant_var <- data[, c("age", "bmi", "children", "charges")]

pairs(
  quant_var,
  main = "Scatterplot Matrix",
  pch = 19,
  col = "lightblue"
)
```

![Scatterplot matrix](figures/scatterplot_matrix.png)

The pairwise plots indicate:

- A positive association between **age** and charges.
- A positive but weaker association between **BMI** and charges.
- Little obvious relationship between **children** and charges.

---

## 4.2 Correlation Analysis

```r
cor_matrix <- cor(quant_var)
cor_matrix
```

Charges are positively correlated with age and BMI, while their correlation with the number of children is weak.

---

# 5. Simple Linear Regression

## 5.1 Charges vs Age

Model:

$$
charges = \beta_0 + \beta_1 age
$$

```r
model_age <- lm(charges ~ age, data)
summary(model_age)

plot(
  age,
  charges,
  main = "Charges vs Age",
  xlab = "Age",
  ylab = "Charges",
  pch = 19,
  col = "lightblue"
)

abline(model_age, col = "red", lwd = 2)
```

![Charges versus age](figures/charges_vs_age.png)

### Result

- Estimated slope: approximately **$257.7/year of age**
- `p < 0.001`
- $R^2 \approx 0.089$

Age is statistically significant, but alone explains only about **8.9%** of the variability in charges.

---

## 5.2 Charges vs BMI

Model:

$$
charges = \beta_0 + \beta_1 bmi
$$

```r
model_bmi <- lm(charges ~ bmi, data)
summary(model_bmi)

plot(
  bmi,
  charges,
  main = "Charges vs BMI",
  xlab = "BMI",
  ylab = "Charges",
  pch = 19,
  col = "lightblue"
)

abline(model_bmi, col = "red", lwd = 2)
```

![Charges versus BMI](figures/charges_vs_bmi.png)

### Result

- Estimated increase: approximately **$394 per BMI unit**
- `p < 0.001`
- $R^2 \approx 0.039$

BMI has a significant positive relationship with charges, but explains only about **3.9%** of the variance by itself.

---

## 5.3 Charges vs Number of Children

Model:

$$
charges = \beta_0 + \beta_1 children
$$

```r
model_children <- lm(charges ~ children, data)
summary(model_children)

plot(
  children,
  charges,
  main = "Charges vs Children",
  xlab = "Children",
  ylab = "Charges",
  pch = 19,
  col = "lightblue"
)

abline(model_children, col = "red", lwd = 2)
```

![Charges versus number of children](figures/charges_vs_children.png)

### Result

- Estimated slope: approximately **$683 per child**
- $R^2 \approx 0.0046$

Children is the weakest of the three quantitative predictors when considered by itself.

---

# 6. Charges Across Categorical Groups

```r
par(mfrow = c(1, 3))

boxplot(
  charges ~ sex,
  data = data,
  main = "Charges by Sex",
  xlab = "Sex",
  ylab = "Charges (USD)",
  col = c("#f9c5c7", "#b6d8f7")
)

boxplot(
  charges ~ smoker,
  data = data,
  main = "Charges by Smoker",
  xlab = "Smoker",
  ylab = "Charges (USD)",
  col = c("#cde9dc", "#ffdab4")
)

boxplot(
  charges ~ region,
  data = data,
  main = "Charges by Region",
  xlab = "Region",
  ylab = "Charges (USD)",
  col = c("#ffb3ba", "#ffdfba", "#ffffba", "#baffc9")
)
```

![Charges by sex, smoker status, and region](figures/charges_by_categories.png)

### Interpretation

- **Sex:** charge distributions are similar between males and females.
- **Smoking:** smokers have dramatically higher charges than non-smokers.
- **Region:** differences are comparatively small, although the southeast shows somewhat greater variability.

---

# 7. Smoking as a Predictor

Model:

$$
charges = \beta_0 + \beta_1 smoker
$$

```r
model_smoker <- lm(charges ~ smoker, data)
summary(model_smoker)
```

### Simple smoking model

| Metric | Approximate result |
|---|---:|
| Non-smoker mean / intercept | $8,434 |
| Smoker coefficient | +$23,616 |
| $R^2$ | 0.619 |

Smoking alone explains approximately **62% of the variance** in annual medical charges, making it by far the strongest simple predictor examined in this project.

---

# 8. Multiple Linear Regression

## 8.1 Full Model

$$
charges =
\beta_0 +
\beta_1 age +
\beta_2 bmi +
\beta_3 children +
\beta_4 sex +
\beta_5 smoker +
\beta_6 region
$$

```r
model_full <- lm(
  charges ~ age + bmi + children + sex + smoker + region,
  data
)

summary(model_full)
```

### Model performance

| Metric | Result |
|---|---:|
| $R^2$ | **0.7509** |
| Adjusted $R^2$ | **0.7494** |
| Overall model | `p < 0.001` |

The full model explains approximately **75% of the variance in insurance charges**.

### Estimated coefficients

| Predictor | Coefficient | Interpretation |
|---|---:|---|
| Age | +256.9 | Each additional year adds about $257 |
| Male | -131.3 | About $131 lower than female, holding other variables constant |
| BMI | +339.2 | Each BMI unit adds about $339 |
| Children | +475.5 | Each additional child adds about $476 |
| Smoker | +23,848.5 | Smokers pay about $23,849 more |
| Northwest | -353 | Difference relative to reference region |
| Southeast | -1,035 | Difference relative to reference region |
| Southwest | -960 | Difference relative to reference region |

Age, BMI, children, and smoking status are significant in the reported full model. Sex is not statistically significant, and regional effects are limited after controlling for the other variables.

---

# 9. Reduced Regression Model

Because sex and region add limited explanatory value, the reduced model retains:

- Age
- BMI
- Number of children
- Smoking status

```r
model_reduced <- lm(
  charges ~ age + bmi + children + smoker,
  data
)

summary(model_reduced)
```

The reduced model has an **Adjusted $R^2 \approx 0.7489$**, almost identical to the full model.

## Final equation

$$
\widehat{Charges}
=
-12,102.77
+257.85(Age)
+321.85(BMI)
+473.50(Children)
+23,811.40(Smoker)
$$

where:

```text
Smoker = 1  if the person smokes
Smoker = 0  otherwise
```

---

# 10. Prediction Example

Consider a 40-year-old person with:

```text
Age      = 40
BMI      = 30
Children = 2
```

### Non-smoker

$$
-12,102.77
+257.85(40)
+321.85(30)
+473.50(2)
\approx \$8,814
$$

### Smoker

$$
-12,102.77
+257.85(40)
+321.85(30)
+473.50(2)
+23,811.40
\approx \$32,625
$$

For otherwise identical characteristics, smoking increases the predicted annual charge by approximately **$23,811**.

---

# 11. Key Findings

### 1. Smoking is the dominant predictor

Smoking status alone explains approximately **62%** of the observed variance in charges. In the reduced multiple regression model, a smoker is predicted to incur roughly **$23.8k more per year** than an otherwise similar non-smoker.

### 2. Age has a significant positive effect

Older individuals tend to have higher medical charges. In the final model, each additional year of age increases predicted charges by approximately **$258**.

### 3. BMI has a significant positive effect

Each additional BMI unit increases predicted charges by approximately **$322** in the reduced model.

### 4. Number of children has a smaller effect

The number of children is weak when considered alone, but contributes positively in the multiple regression model.

### 5. Sex and region contribute little after adjustment

Once age, BMI, children, and smoking are controlled for, sex and region provide limited additional explanatory power.

### 6. A compact model performs nearly as well as the full model

| Model | Predictors | Adjusted $R^2$ |
|---|---|---:|
| Full | Age, BMI, children, sex, smoker, region | **0.7494** |
| Reduced | Age, BMI, children, smoker | **0.7489** |

The reduced model therefore provides nearly the same explanatory performance with fewer predictors.

---

# 12. Main Results at a Glance

| Analysis | Main result |
|---|---|
| Dataset size | 1,338 observations |
| Missing values | 0 |
| Mean charges | ~$13,270 |
| Median charges | ~$9,382 |
| Strongest simple predictor | Smoking status |
| Smoker-only $R^2$ | ~0.619 |
| Age-only $R^2$ | ~0.089 |
| BMI-only $R^2$ | ~0.039 |
| Children-only $R^2$ | ~0.0046 |
| Full-model $R^2$ | 0.7509 |
| Full-model adjusted $R^2$ | 0.7494 |
| Reduced-model adjusted $R^2$ | 0.7489 |
| Smoker effect in reduced model | +$23,811/year |

---

# 13. Strengths

- Includes both quantitative and categorical predictors.
- Contains no missing observations.
- Sample size of 1,338 is sufficient for the regression analyses used here.
- Analysis progresses systematically from descriptive EDA to simple and multiple regression.
- The reduced model is easier to interpret while retaining nearly all of the explanatory performance of the full model.

---

# 14. Limitations

- This is **observational data**, so the estimated associations should not be interpreted automatically as causal effects.
- Potentially relevant variables such as medical history, income, exercise, diet, and insurance-plan characteristics are absent.
- The linear regression specification may not capture nonlinearities or interactions.
- The charge distribution is strongly right-skewed, which may affect linear-model assumptions.
- Results may not generalize to other populations or healthcare systems.

---

# 15. Reproducing the Analysis

## Requirements

The project uses base R functionality, so no additional visualization or regression package is required for the analysis shown above.

Recommended:

```text
R >= 4.x
RStudio
```

## Run

1. Place `insurance.csv` in the project root.
2. Open the analysis script or R Markdown file.
3. Run the sections sequentially.
4. Export plots into `figures/` using the filenames shown in this README if you want the charts to render directly on GitHub.

Example:

```r
png("figures/charges_vs_age.png", width = 1000, height = 700)

plot(
  age,
  charges,
  main = "Charges vs Age",
  xlab = "Age",
  ylab = "Charges",
  pch = 19,
  col = "lightblue"
)

abline(model_age, col = "red", lwd = 2)

dev.off()
```

---

# 16. Conclusion

The analysis shows that annual health insurance charges are most strongly associated with **smoking status**, followed by **age** and **BMI**. Smoking is particularly influential: it explains a large share of charge variation even before other variables are considered.

A reduced linear model using only **age, BMI, number of children, and smoking status** achieves almost the same adjusted $R^2$ as the full model. This makes the reduced model a more parsimonious summary of the relationships observed in the dataset.

---

## Notes

This README summarizes the exploratory analysis and regression results from the project report. Numerical results should be regenerated from `insurance.csv` when reproducing or extending the project.
