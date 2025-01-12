```r
# Update the first three column names for better readability
colnames(x)[1:3] <- c("submitter_id", "overallsurvival", "deceased")

# Remove the column named "ACLY.1" from the dataframe
x <- x[, !colnames(x) %in% "ACLY.1"]

# Replace all 0 values in the "overallsurvival" column with NA (missing values)
x$overallsurvival[x$overallsurvival == 0] <- NA

# Filter out rows where "overallsurvival" is NA
x <- subset(x, !is.na(x$overallsurvival))

# Remove columns that contain any NA values
x <- x[, colSums(is.na(x)) == 0]

# Drop the "vital_status" column from the dataframe
x$vital_status <- NULL

# Identify all columns related to gene expression by excluding specific columns
expression_cols <- setdiff(colnames(x), c("submitter_id", "overallsurvival", "deceased"))

# Calculate the median for each gene expression column, ignoring NA values
medians <- sapply(expression_cols, function(col) median(x[[col]], na.rm = TRUE))

```
---
```r
# Iterate through each gene expression column in the dataframe
for (col in expression_cols) {
  
  # Create a new column name by prefixing "expr_" to the current column name
  new_col <- paste0("expr_", col)
  
  # Save the original expression values into the new column
  x[[new_col]] <- x[[col]]
  
  # Overwrite the current column with "High" or "Low" based on median threshold
  # If the original value is greater than or equal to the median, assign "High"; otherwise, assign "Low"
  x[[col]] <- ifelse(x[[new_col]] >= medians[col], "High", "Low")
}
```

---
```r
# Identify all columns related to gene expression by excluding specific metadata columns
expression_cols <- setdiff(colnames(x), c("submitter_id", "overallsurvival", "deceased"))

# Exclude columns that start with "expr_", as they are the new columns storing original values
expression_cols <- expression_cols[!startsWith(expression_cols, "expr_")]

# Exclude the main gene column (specified as 'gene_name'), if it is present
# 'gene_name' is assumed to be a variable containing the name of the gene column
expression_cols <- expression_cols[expression_cols != gene_name]

# Further filter the expression columns by ensuring each column has more than one unique level
# Convert each column to a factor and check if it has more than one level
expression_cols <- expression_cols[sapply(expression_cols, function(col) {
  length(levels(factor(x[[col]]))) > 1
})]
```
