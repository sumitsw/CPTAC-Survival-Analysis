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
This code categorizes each gene expression column into "High" or "Low" based on its median value. The original numeric expression values are preserved in new columns for reference.
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
This code identifies valid gene expression columns by excluding non-expression columns, ensuring no duplicates or irrelevant columns are included, and retaining only those with more than one unique value.
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
---
This code performs survival analysis and statistical comparisons for each gene in `expression_cols`. It visualizes survival curves, calculates hazard ratios (HR), log-rank test p-values, and other summary statistics, and saves results and plots for each gene comparison.

```r
# Loop through each gene in the expression columns
for (gene_col in expression_cols) {
  
  # Print the gene being processed
  cat("\n================Processing Gene: ", gene_col,"==========================\n")
  
  # Convert the main gene and the current gene to factors with levels "High" and "Low"
  x[[gene_name]] <- factor(x[[gene_name]], levels = c("High", "Low"))
  x[[gene_col]] <- factor(x[[gene_col]], levels = c("High", "Low"))
  
  # Create a survival object for analysis
  surv_obj <- Surv(x$overallsurvival, x$deceased)
  
  # Fit a survival model with both genes as stratification variables
  fit_model <- survfit(surv_obj ~ x[[gene_name]] + x[[gene_col]], data = x)
  
  # Check if the model has four strata (all combinations of "High" and "Low")
  if (length(fit_model$strata) == 4) {
    
    # Generate and customize survival plot
    ggsurv <- ggsurvplot(
      fit_model,
      data = x,
      xlab = "Time (Days)",         # Label for x-axis
      ylab = "Survival Probability",# Label for y-axis
      break.time.by = 500,          # Break time intervals by 500 days
      legend = c(0.85, 0.94),       # Position of legend on the plot
      legend.title = "",            # No title for legend
      legend.labs = c(              # Labels for legend with gene combinations
        paste(gene_name, "=High,", gene_col, "=High"),
        paste(gene_name, "=High,", gene_col, "=Low"),
        paste(gene_name, "=Low,", gene_col, "=High"),
        paste(gene_name, "=Low,", gene_col, "=Low")
      ),
      font.x = c(18, "bold"),       # Font style for x-axis label
      font.y = c(18, "bold"),       # Font style for y-axis label
      font.legend = c(8, "black"),  # Font style for legend
      font.tickslab = c(16, "black"), # Font style for tick labels
      palette = c("red", "darkblue", "green", "violet"), # Custom colors
      ggtheme = theme_survminer(base_size = 16, font.legend = c(16, "plain", "black"))
    )
    # Further adjust plot scales and limits
    ggsurv$plot <- ggsurv$plot +
      scale_x_continuous(expand = c(0, 0), limits = c(0, NA)) +
      scale_y_continuous(expand = c(0, 0), limits = c(0, 1.05)) +
      coord_cartesian(xlim = c(0, 6700))
    
    # Save the survival plot as a PNG file
    ggsave(filename = paste0(gene_name, "_", gene_col, ".png"), 
           plot = last_plot(), 
           device = "png", 
           width = 7, 
           height = 5.7)
    
    # Print the survival plot
    print(ggsurv)
  }
  
  # Create a grouping variable with combinations of the two genes
  x$group <- paste0(gene_name, "=", x[[gene_name]], ", ", gene_col, "=", x[[gene_col]])
  
  # Define the levels for the new group column
  levels(x$group) <- c(
    paste0(gene_name,"=High, ", gene_col,"=High"),
    paste0(gene_name,"=High, ", gene_col,"=Low"),
    paste0(gene_name,"=Low, ", gene_col,"=High"),
    paste0(gene_name,"=Low, ", gene_col,"=Low")
  )
  
  # Extract unique groups
  groups <- levels(x$group)
  
  # Initialize a data frame to store results
  results <- data.frame(Group1 = character(),
                        Group2 = character(),
                        p_value = numeric(),
                        stringsAsFactors = FALSE)
  
  # Compare each pair of groups using log-rank tests
  for (i in 1:(length(groups)-1)) {
    for (j in (i+1):length(groups)) {
      
      # Subset data for the two groups being compared
      group1_data <- subset(x, group == groups[i])
      group2_data <- subset(x, group == groups[j])
      
      # Fit survival models for each group
      fit_group1 <- survfit(Surv(group1_data$overallsurvival, group1_data$deceased) ~ 1, data = group1_data)
      fit_group2 <- survfit(Surv(group2_data$overallsurvival, group2_data$deceased) ~ 1, data = group2_data)
      
      # Extract median survival, number of observations, and events for each group
      group1_median <- summary(fit_group1)$table["median"]
      group2_median <- summary(fit_group2)$table["median"]
      group1_obs <- summary(fit_group1)$table["records"]
      group2_obs <- summary(fit_group2)$table["records"]
      group1_events <- summary(fit_group1)$table["events"]
      group2_events <- summary(fit_group2)$table["events"]
      
      # Combine data for the two groups
      combined_data <- rbind(group1_data, group2_data)
      surv_obj_combined <- Surv(combined_data$overallsurvival, combined_data$deceased)
      
      # Perform log-rank test
      logrank_test <- survdiff(surv_obj_combined ~ combined_data$group)
      p_value <- logrank_test$pvalue
      
      # Fit Cox proportional hazards model
      cox_model_combined <- coxph(surv_obj_combined ~ group, data = combined_data)
      cox_summary_combined <- summary(cox_model_combined)
      HR_p_values <- cox_summary_combined$coefficients[, "Pr(>|z|)"]
      hazard_ratios <- exp(cox_summary_combined$coefficients[, "coef"])
      
      # Store the results in the data frame
      results <- rbind(results, data.frame(Group1 = groups[i],
                                           Group2 = groups[j],
                                           G1_obs = group1_obs,
                                           G2_obs = group2_obs,
                                           G1_Median = group1_median,
                                           G2_Median = group2_median,
                                           G1_Events = group1_events,
                                           G2_Events = group2_events,
                                           p_value = round(p_value, 3),
                                           HR = round(hazard_ratios, 3),
                                           HR_P_Value = round(HR_p_values, 3)))
    }
    # Replace NA values with empty strings in results
    results[is.na(results)] <- ""
    
    # Save the results to a CSV file
    output_file <- paste0(gene_name, "_", gene_col, ".csv")
    write.csv(results, file = output_file, row.names = FALSE)
  }
}
```
