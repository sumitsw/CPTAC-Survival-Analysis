# CPTAC Survival Analysis Documentation

This script performs survival analysis for the ACLY gene expression data, stratified by clinical and pathological features such as gender, tumor stage, tumor grade, lymph node involvement, and metastasis. It generates Kaplan-Meier survival plots for different subgroups and saves the results as CSV files and plots.

## Dependencies

The following R packages are required:

- `ggplot2`: For data visualization.
- `survival`: To create survival objects and perform survival analysis.
- `survminer`: For enhanced survival plots.
- `gtsummary`: For summary statistics.
- `broom`: For model tidying.

Install missing packages using:
```r
install.packages(c("ggplot2", "survival", "survminer", "gtsummary", "broom"))
```

## Workflow

### 1. Load and Preprocess Data

```r
# Load libraries
library(ggplot2)
library(survival)
library(survminer)
library(gtsummary)
library(broom)

# Load data
data <- read.csv(file.choose(), header = TRUE)

# Filter out rows with missing OS.event
data <- subset(data, !is.na(data$OS.event))

# Create a binary expression variable based on median ACLY value
median_tpm <- median(data$ACLY)
data$expression <- ifelse(data$ACLY > median_tpm, "High", "Low")

# Define output directory
output_dir <- "results/CPTAC/Survival_Analysis"
```

### 2. Define the `generate_survival_plot` Function

This function:

- Generates Kaplan-Meier plots using `ggsurvplot`.
- Annotates plots with log-rank p-values, hazard ratios (HR), and other statistics.
- Saves the plot as a PNG file and the statistics as a CSV file.

```r
generate_survival_plot <- function(fit, filter_data, plot_name) {
  # Create Kaplan-Meier plot
  ggsurv <- ggsurvplot(
    fit,
    data = filter_data,
    surv.median.line = "hv",
    pval = TRUE,
    xlab = "Time (Days)",
    ylab = "Survival Probability",
    break.time.by = 500,
    title = paste(plot_name),
    legend.title = "",
    legend.labs = c("High", "Low"),
    font.title = c(20, "bold"),
    font.x = c(18, "bold"),
    font.y = c(18, "bold"),
    font.tickslab = c(16, "plain", "darkgreen"),
    palette = "UCSCGB",
    surv.plot.height = 2,
    ggtheme = theme_survminer(base_size = 16, font.legend = c(16, "plain", "black"))
  )

  # Calculate statistics
  logrank_test <- survdiff(surv_obj ~ expression, data = filter_data)
  logrank_p <- logrank_test$pvalue

  cox_model <- coxph(surv_obj ~ expression, data = filter_data)
  HR <- exp(coef(cox_model))
  p_value <- summary(cox_model)$coefficients[,"Pr(>|z|)"]
  n <- cox_model$n
  events <- cox_model$nevent

  group_counts <- table(filter_data$expression)
  n_high <- group_counts["High"]
  n_low <- group_counts["Low"]

  # Annotate plot with statistics
  ggsurv$plot <- ggsurv$plot +
    ggplot2::annotate(
      "text",
      x = Inf, y = Inf,
      vjust = 1, hjust = 1,
      label = paste(
        "Logrank P =", round(logrank_p, 3),
        "\nHR =", round(HR, 2), 
        "\np(HR) =", round(p_value, 3), 
        "\nn =", n, 
        "\nEvents =", events,
        "\nn(High) =", n_high, 
        "\nn(Low) =", n_low
      ),
      size = 3.5
    ) + 
    scale_x_continuous(expand = c(0, 0), limits = c(0, NA)) +
    scale_y_continuous(expand = c(0, 0), limits = c(0, 1)) +
    coord_cartesian(xlim = c(0, 2200))

  # Save plot and results
  ggsave(filename = paste0(plot_name, ".png"), plot = last_plot(), device = "png")
  print(ggsurv)

  results <- data.frame(
    Observations = n,
    Events = events,
    n_High = n_high,
    n_Low = n_low,
    LogRank_p_value = round(logrank_p, 3),
    HR = round(HR, 3),
    HR_p_value = round(p_value, 3)
  )
  output_file <- paste0(plot_name, ".csv")
  write.csv(results, file = output_file, row.names = FALSE)
}
```

### 3. Overall Survival Analysis

Generate survival plots for the entire dataset:
```r
surv_obj <- Surv(time = data$OS.time, event = data$OS.event)
fit <- survfit(surv_obj ~ data$expression)
generate_survival_plot(fit, data, "ACLY Overall Survival")
```

### 4. Subgroup Analyses

#### Gender
```r
# Male
filter_data <- data[which(data$sex == 'Male'), ]
surv_obj <- Surv(time = filter_data$OS.time, event = filter_data$OS.event)
fit <- survfit(surv_obj ~ expression, data = filter_data)
generate_survival_plot(fit, filter_data, "Gender - Male")

# Female
filter_data <- data[which(data$sex == 'Female'), ]
surv_obj <- Surv(time = filter_data$OS.time, event = filter_data$OS.event)
fit <- survfit(surv_obj ~ expression, data = filter_data)
generate_survival_plot(fit, filter_data, "Gender - Female")
```

#### Tumor Stage
```r
stages <- c("Stage I", "Stage II", "Stage III", "Stage IV")
for (stage in stages) {
  filter_data <- data[which(data$tumor_stage_pathological == stage), ]
  surv_obj <- Surv(time = filter_data$OS.time, event = filter_data$OS.event)
  fit <- survfit(surv_obj ~ expression, data = filter_data)
  generate_survival_plot(fit, filter_data, stage)
}
```

#### Tumor Grade
```r
# Example: T4 Stage
filter_data <- data[data$pathologic_staging_primary_tumor_pt %in% c("pT4", "pT4a", "pTIVA"), ]
surv_obj <- Surv(time = filter_data$OS.time, event = filter_data$OS.event)
fit <- survfit(surv_obj ~ expression, data = filter_data)
generate_survival_plot(fit, filter_data, "T4")

filter_data <- data[grepl("pT2", data$pathologic_staging_primary_tumor_pt), ]
surv_obj <- Surv(time = filter_data$OS.time, event = filter_data$OS.event )
fit <- survfit(surv_obj ~ expression, data = filter_data)
generate_survival_plot(fit, filter_data, "T2")

filter_data <- data[grepl("pT3", data$pathologic_staging_primary_tumor_pt), ]
surv_obj <- Surv(time = filter_data$OS.time, event = filter_data$OS.event )
fit <- survfit(surv_obj ~ expression, data = filter_data)
generate_survival_plot(fit, filter_data, "T3")

filter_data <- data[grepl("pT1", data$pathologic_staging_primary_tumor_pt), ]
surv_obj <- Surv(time = filter_data$OS.time, event = filter_data$OS.event )
fit <- survfit(surv_obj ~ expression, data = filter_data)
generate_survival_plot(fit, filter_data, "T1")
```
### Regional Lymph Node Stage
```r
filter_data <- data[grepl("pN3", data$pathologic_staging_regional_lymph_nodes_pn), ]
surv_obj <- Surv(time = filter_data$OS.time, event = filter_data$OS.event )
fit <- survfit(surv_obj ~ expression, data = filter_data)
generate_survival_plot(fit, filter_data, "N3")

filter_data <- data[grepl("pN2", data$pathologic_staging_regional_lymph_nodes_pn), ]
surv_obj <- Surv(time = filter_data$OS.time, event = filter_data$OS.event )
fit <- survfit(surv_obj ~ expression, data = filter_data)
generate_survival_plot(fit, filter_data, "N2")

filter_data <- data[grepl("pN1", data$pathologic_staging_regional_lymph_nodes_pn), ]
surv_obj <- Surv(time = filter_data$OS.time, event = filter_data$OS.event )
fit <- survfit(surv_obj ~ expression, data = filter_data)
generate_survival_plot(fit, filter_data, "N1")

filter_data <- data[grepl("pN0", data$pathologic_staging_regional_lymph_nodes_pn), ]
surv_obj <- Surv(time = filter_data$OS.time, event = filter_data$OS.event )
fit <- survfit(surv_obj ~ expression, data = filter_data)
generate_survival_plot(fit, filter_data, "N0")
```
### Metastasis
```r
filter_data <- data[grepl("cM0", data$clinical_staging_distant_metastasis_cm), ]
surv_obj <- Surv(time = filter_data$OS.time, event = filter_data$OS.event )
fit <- survfit(surv_obj ~ expression, data = filter_data)
generate_survival_plot(fit, filter_data, "M0")
```

## Output

- Plots are saved as PNG files in the specified directory.
- Summary statistics are saved as CSV files in the same directory.
