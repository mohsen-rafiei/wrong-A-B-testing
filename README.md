# Nested A/B Testing Example Dataset

This repository contains a **synthetic dataset** created for teaching and demonstrating a common statistical mistake in A/B testing: using the wrong analysis method when treatment is assigned at the group level.

## Scenario

Imagine an educational experiment with:

- **75 students** spread across **10 classes**
- Each **class** (not each student) is randomly assigned to one of two versions of a learning platform:
  - **Design A** (control)
  - **Design B** (treatment)
- We record **`TimeOnPage`** for each student as a measure of engagement

Design B initially looks better, but that difference is driven by just two high-performing classes in the treatment group. This setup illustrates how ignoring data structure can lead to **false positives**.

## Files

- `data.csv`: The main dataset

##  Statistical Comparison

### Welch’s t-test (Incorrect)

Treats each student as independent. Violates the assumption of independence because students are grouped by class.

**Result:**  
Significant difference between A and B  
**p-value = 0.00007**

### Mixed-Effects Model (Correct)

Accounts for class-level assignment using a random intercept for `GroupID`.

**Result:**  
No significant difference between A and B  
**p-value = 0.109**

## Key Insight

If you ignore group-level structure, you may:
- Launch features based on false signals
- Waste engineering, design, and marketing resources
- Miss the real drivers of engagement hiding within specific subgroups

Instead of assuming treatment worked, this dataset encourages deeper analysis. What happened in Classes G003 and G007? What can we learn from them?

## R Code Example

```r
# Load necessary packages
library(tidyverse)
library(lme4)
library(lmerTest)

# Load the data
data <- read.csv("data.csv")
data$Condition <- as.factor(data$Condition)
data$GroupID <- as.factor(data$GroupID)

# Welch's t-test (incorrect approach)
t_test_result <- t.test(TimeOnPage ~ Condition, data = data)
print("Welch's t-test result:")
print(t_test_result)

# Linear mixed-effects model (correct approach)
mixed_model <- lmer(TimeOnPage ~ Condition + (1 | GroupID), data = data)
summary(mixed_model)
