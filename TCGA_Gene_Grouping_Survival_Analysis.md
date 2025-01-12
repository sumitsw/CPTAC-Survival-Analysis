### Purpose of the Code:
This code performs data preparation for survival analysis by:
1. Loading a CSV file containing survival and gene expression data.
2. Cleaning and transforming the dataset, including handling missing values and renaming columns.
3. Identifying gene expression columns for further analysis.
4. Computing the median expression value for each gene to categorize data in future steps.

```r
# Load the necessary libraries for survival analysis and visualization
library(survival)
library(survminer)

# Step 1: Prompt the user to select a CSV file and load the data into a dataframe
file_path <- file.choose()  # Opens a file dialog to select the CSV file
x <- read.csv(file_path, header = TRUE)  # Load the selected file with column headers

# Step 2: Define the name of the primary gene of interest
gene_name <- "ACLY"

# Step 3: Rename the first three columns for better readability
# Assumes the first three columns correspond to:
# - "submitter_id": Unique identifier for each subject
# - "overallsurvival": Overall survival time
# - "deceased": Status of whether the subject is deceased (1) or not (0)
colnames(x)[1:3] <- c("submitter_id", "overallsurvival", "deceased")

# Step 4: Remove the column "ACLY.1" if it exists in the dataset
# This avoids duplication or irrelevant data
x <- x[, !colnames(x) %in% "ACLY.1"]

# Step 5: Replace "0" values in the "overallsurvival" column with NA
# A survival time of 0 is likely invalid or represents missing data
x$overallsurvival[x$overallsurvival == 0] <- NA

# Step 6: Remove rows where "overallsurvival" is NA
# These rows cannot contribute to survival analysis
x <- subset(x, !is.na(x$overallsurvival))

# Step 7: Remove columns containing any NA values
# Ensures the dataset is complete for analysis
x <- x[, colSums(is.na(x)) == 0]

# Step 8: Drop the "vital_status" column
# This column is assumed to be unnecessary for the current analysis
x$vital_status <- NULL

# Step 9: Identify gene expression columns for analysis
# Exclude non-expression columns like "submitter_id", "overallsurvival", and "deceased"
expression_cols <- setdiff(colnames(x), c("submitter_id", "overallsurvival", "deceased"))

# Step 10: Compute the median expression value for each gene expression column
# The median will later be used to categorize expression levels as "High" or "Low"
medians <- sapply(expression_cols, function(col) median(x[[col]], na.rm = TRUE))
```
---
### Purpose of the Code:
This code categorizes each gene expression column into "High" or "Low" based on its median value. The original numeric expression values are preserved in new columns for reference.

### Code with Detailed Comments:
```r
# Step 1: Iterate through each gene expression column
for (col in expression_cols) {
  
  # Step 2: Generate a new column name to store the original expression values
  # Prefixes the original column name with "expr_"
  # For example, if the column is "ACLY", the new column will be "expr_ACLY"
  new_col <- paste0("expr_", col)
  
  # Step 3: Save the original numeric expression values into the new column
  # This ensures that the original data is retained for reference or further analysis
  x[[new_col]] <- x[[col]]
  
  # Step 4: Categorize the original expression values into "High" or "Low"
  # Compare each value in the original column to its corresponding median:
  # - Assign "High" if the value is greater than or equal to the median
  # - Assign "Low" if the value is less than the median
  # Replace the original column with these categorical values
  x[[col]] <- ifelse(x[[new_col]] >= medians[col], "High", "Low")
}
```

---
### Purpose of the Code:
This code identifies valid gene expression columns by excluding non-expression columns, ensuring no duplicates or irrelevant columns are included, and retaining only those with more than one unique value.
### Code with Comments:
```r
# Step 1: Identify potential gene expression columns by excluding specific non-expression columns
# Exclude "submitter_id", "overallsurvival", and "deceased" as they are not gene expression data
expression_cols <- setdiff(colnames(x), c("submitter_id", "overallsurvival", "deceased"))

# Step 2: Remove columns that start with "expr_"
# These columns contain the original expression values saved in earlier steps
expression_cols <- expression_cols[!startsWith(expression_cols, "expr_")]

# Step 3: Exclude the main gene of interest (stored in 'gene_name')
# Prevents the primary gene from being included in analyses where it might cause bias
expression_cols <- expression_cols[expression_cols != gene_name]

# Step 4: Retain columns with more than one unique value
# Filter columns where the number of unique levels is greater than 1
# Ensures that only non-constant columns are included
expression_cols <- expression_cols[sapply(expression_cols, function(col) {
  length(levels(factor(x[[col]]))) > 1
})]
```
