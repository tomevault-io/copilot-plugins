## slcptac

> **Package Type**: Proteogenomic Analysis Toolkit (Protein/Phospho + Genomics + Clinical)

# SLCPTAC Documentation Rules for LLM RAG System

**Package Type**: Proteogenomic Analysis Toolkit (Protein/Phospho + Genomics + Clinical)  
**Target Users**: Cancer Researchers + Proteomics Scientists + LLM Systems + Cursor AI Agent  
**Core Purpose**: Enable LLM to SELECT the right analysis scenario and GENERATE correct proteogenomic analysis calls

**CRITICAL DISTINCTION**: SLCPTAC = CPTAC Proteomics + Phosphoproteomics (NOT TCGA)

---

## 🎯 CORE UNDERSTANDING: Proteogenomic Analysis Package

**SLCPTAC = 17 Statistical Scenarios + Phosphorylation-Centric Analysis**

```
User Query: "AKT1蛋白水平和磷酸化之间有什么关系?"
    ↓
LLM RAG Retrieval → Finds cptac_correlation()
    ↓
LLM Reads: @description + @param + @examples
    ↓
LLM Generates: result <- cptac_correlation(
                  var1 = "AKT1",
                  var1_modal = "Protein",     # Protein abundance
                  var1_cancers = "BRCA",
                  var2 = "AKT1",
                  var2_modal = "Phospho",     # Auto-detects all phospho sites
                  var2_cancers = "BRCA"
                )
    ↓
Execute → Returns: Correlation for each phospho site + Heatmap
    ↓
LLM Answers: "AKT1蛋白水平与其9个磷酸化位点正相关，
             S473位点相关性最强 (r=0.68, p<0.001)"
```

**Unique SLCPTAC Features**:
- ✅ **Proteomics**: Protein abundance (mass spectrometry)
- ✅ **Phosphoproteomics**: Site-specific phosphorylation levels
- ✅ **Auto-site detection**: Input "AKT1" → Returns all phospho sites
- ✅ **Transcriptome-Proteome integration**: Compare mRNA vs protein
- ✅ **Mutation-Phospho association**: How mutations affect phosphorylation

**Critical Difference from Other Packages**:
- ❌ NOT: Raw data analysis (like DoCCI, RunPTA)
- ✅ YES: Pre-defined statistical scenarios on curated datasets
- Users ask **research questions**, not **how to analyze data**

---

## 🚨 FIVE CORE PRINCIPLES (NON-NEGOTIABLE)

### Principle #1: TEST-FIRST (测试成功是基石)

**For statistical functions, testing = Scenario execution + Result verification**

```r
# Test template for SLTCGA/SLCPTAC functions
Rscript -e "
library(SLTCGA)  # or SLCPTAC
start <- Sys.time()

# Test with real research question
result <- tcga_correlation(
  var1 = 'TP53', var1_modal = 'RNAseq', var1_cancers = 'BRCA',
  var2 = 'MDM2', var2_modal = 'RNAseq', var2_cancers = 'BRCA',
  method = 'pearson'
)

runtime <- as.numeric(difftime(Sys.time(), start, units = 'secs'))
cat('✓ Analysis successful\n')
cat('Runtime:', runtime, 'sec\n')
cat('Correlation:', result$statistics$correlation, '\n')
cat('P-value:', result$statistics$pvalue, '\n')
cat('Plot saved:', result$plot_path, '\n')
"
```

**What to Document from Test**:
- ✅ Analysis runtime (typically 1-30 sec depending on scenario)
- ✅ Result structure (statistics, plot, data)
- ✅ Statistical values (correlation, p-value, HR, etc.)
- ✅ Plot output (file path)

**If test fails**:
- Data loading error → Check if `SL_BULK_DATA` environment variable is set
- Scenario mismatch → Check if variable types match the scenario
- STOP and report to user

---

### Principle #2: ENGLISH ONLY

- ✅ ALL roxygen documentation in English
- ✅ ALL code comments in English
- ❌ NO Chinese characters: 相关分析, 富集分析, 生存分析, etc.

---

### Principle #3: DELETE ALL NOISE

**Category 1: Data Setup (MINIMIZE)**

❌ DELETE verbose instructions:
```r
#' **Data Setup**:
#' \itemize{
#'   \item Step 1: Download TCGA data from...
#'   \item Step 2: Set environment variable SL_BULK_DATA...
#'   \item Step 3: Preprocess data using...
#' }
```

✅ KEEP minimal one-liner:
```r
#' **Data**: Requires \code{Sys.setenv(SL_BULK_DATA = "/path/to/data")}. 
#' See \code{vignette("setup")} for first-time configuration.
```

**Category 2: Statistical Jargon (KEEP if necessary for interpretation)**

✅ Keep statistical context:
```r
#' @details
#' **Interpreting Results**:
#' \itemize{
#'   \item **Pearson correlation**: Measures linear relationship (-1 to 1)
#'   \item **p-value**: Statistical significance (< 0.05 typically significant)
#'   \item **Hazard ratio**: >1 indicates increased risk, <1 protective effect
#' }
```

**Category 3: Filler Words (DELETE)**

❌ "Powerful analysis", "Comprehensive toolkit", "Advanced statistics"

---

### Principle #4: REFERENCE-BACKED

Every statistical method MUST have:
- Method paper citation
- DOI link
- Statistical test reference (if non-standard)

**Search priority**:
1. Check existing @references in the .R file
2. Search for method name in documentation
3. Use web_search: "[Method name] statistical test paper DOI"
4. If not found → Ask user

---

### Principle #5: EXECUTABLE EXAMPLES

- ✅ ALL @examples must use \donttest{} (not \dontrun{})
- ✅ Examples must use real cancer types and genes
- ✅ Show expected output structure
- ✅ Include interpretation guidance

---

## 📋 DOCUMENTATION STRUCTURE FOR STATISTICAL FUNCTIONS

### @title (Scenario-Oriented)

**Format**: `[Statistical Analysis Type] - [Data Types]`

✅ Good:
```r
#' Correlation Analysis Across Multi-Omics Data
#' Enrichment Analysis - Mutation-Driven Pathway Changes
#' Survival Analysis with Clinical and Molecular Variables
```

❌ Bad:
```r
#' TCGA Correlation  ← Too vague
#' Analyze Data  ← Not specific
```

---

### @description (3-4 sentences, 60-100 words)

**This is the LLM's decision point for scenario selection!**

**Template**:
```r
#' @description
#' [Sentence 1: What statistical question + Which scenarios]
#' Performs [statistical analysis type] across [data modalities] covering 
#' [X scenarios] from [comprehensive scenario list].
#' 
#' [Sentence 2: Data coverage]
#' Supports [N cancer types], [M omics layers], and [K variable combinations].
#'
#' [Sentence 3: Analysis output]
#' Generates [statistical metrics] with automated [visualization type] and 
#' publication-ready outputs.
#'
#' [Sentence 4: Special features]
#' **Features**: [Unique capabilities like multi-cancer, subtype analysis, etc.]
```

**Real Example**:
```r
#' @description
#' Performs proteogenomic correlation and association analysis across 7 scenarios:
#' continuous-continuous (Pearson/Spearman), categorical-continuous (Wilcoxon/
#' Kruskal-Wallis), categorical-categorical (Chi-square/Fisher). Supports 10 CPTAC 
#' cancer types with 7 omics layers (RNAseq, Protein, Phospho, Mutation, Clinical, 
#' CNV, Methylation). Automatically detects phosphorylation sites for any protein 
#' (e.g., "AKT1" → 9 sites). Generates automated scatter plots, box plots, heatmaps, 
#' and correlation networks with statistical testing.
#' **Features**: Transcriptome-proteome integration, phosphoproteomics-centric, 
#' multi-cancer comparison, mutation-phospho association.
```

**Why this works for LLM**:
- ✅ "correlation and association analysis across 7 scenarios" → Clear function purpose
- ✅ "33 TCGA cancer types" → Scope clarity
- ✅ "Pearson/Spearman", "Wilcoxon/Kruskal-Wallis" → Method specificity
- ✅ "scatter plots, box plots, heatmaps" → Output types
- ✅ "Multi-cancer comparison" → Unique features

---

### @param (Clear Scenario Parameters)

**For statistical functions, parameters = scenario configuration**

**Format**:
```r
#' @param param_name [Type]. [What it configures]. 
#'   Options: [valid values]. Default: \code{value}.
#'   [Impact on analysis scenario].
```

**Examples**:

```r
#' @param var1 Character vector. Gene/variable names to analyze.
#'   Examples: "TP53", c("TP53", "KRAS"). 
#'   Multiple variables trigger Scenario 2/3 (multi-variable analysis).

#' @param var1_modal Character. Data modality for var1.
#'   Options: "RNAseq", "Protein", "Phospho", "Mutation", "Clinical", "CNV", "Methylation".
#'   **Phospho**: Automatically detects all phosphorylation sites for given gene.
#'   Example: var1="AKT1", var1_modal="Phospho" returns ~9 phospho sites.
#'   Determines variable type for scenario selection.

#' @param var1_cancers Character vector. Cancer types to include.
#'   Examples: "BRCA", c("BRCA", "LUAD", "CCRCC").
#'   CPTAC supports 10 cancer types: BRCA, CCRCC, COAD, GBM, HNSCC, 
#'   LUAD, LUSC, OV, PDAC, UCEC.
#'   Multiple cancers enable pan-cancer comparison.
#'   **Note**: Phospho data available for 8 cancer types (not OV, COAD).

#' @param method Character. Correlation method.
#'   Options: "pearson" (linear), "spearman" (monotonic).
#'   Default: \code{"pearson"}.
#'   Spearman recommended for non-normal distributions.

#' @param var2 Character vector. Second variable(s) for correlation.
#'   If NULL, performs one-to-many analysis (Scenario 2).
#'   If provided, performs pairwise correlation (Scenario 1/3).
```

**Key points**:
- ✅ Explain how parameter affects scenario selection
- ✅ Include valid options with biological context
- ✅ Cross-reference helper functions (list_cancer_types, list_variables)
- ✅ Show impact on analysis type

---

### @references (Method + Database Citation)

**Mandatory for every statistical function**:

```r
#' @references
#' **Statistical Methods**:
#' [Authors] ([Year]). [Test name]. [Journal/Book]. \doi{[10.xxxx/xxxxx]}
#'
#' **Database**:
#' [TCGA/CPTAC] Research Network ([Year]). [Database name]. 
#' \url{[Official URL]}
```

**Real Example**:
```r
#' @references
#' **Statistical Methods**:
#' Pearson K (1895). Notes on regression and inheritance in the case of two parents.
#' Proceedings of the Royal Society of London, 58, 240-242.
#'
#' Spearman C (1904). The proof and measurement of association between two things.
#' Am J Psychol, 15(1):72-101.
#'
#' **Database**:
#' Clinical Proteomic Tumor Analysis Consortium (2020). Proteogenomic 
#' characterization of human cancer. Cell, various publications.
#' \doi{10.1016/j.cell.2020.01.026}
#'
#' Database: \url{https://proteomics.cancer.gov/programs/cptac}
```

---

### @section Performance Test (MANDATORY - from real test)

```r
#' @section Performance Test:
#' \itemize{
#'   \item **Test scenario**: TP53-MDM2 correlation in BRCA (Scenario 1)
#'   \item **Runtime**: ~3.2 sec (data loading: 2.1 sec, analysis: 1.1 sec)
#'   \item **Sample size**: 1,095 BRCA patients
#'   \item **Result**: r=0.42, p<0.001
#'   \item **Output files**: PNG plot (300 DPI) + TSV statistics + RDS data
#'   \item **Recommended**: Suitable for single-cancer or multi-cancer analyses (up to 10 cancers simultaneously)
#' }
```

**What to document**:
- ✅ Specific test scenario (which variable types, which cancer)
- ✅ Runtime breakdown (data loading vs computation)
- ✅ Sample size processed
- ✅ Example statistical results
- ✅ Output file types
- ❌ NOT: "Fast", "Efficient" (meaningless adjectives)

---

### @details (Statistical Interpretation - IMPORTANT for LLM答案)

**Only include statistical interpretation guidance**:

```r
#' @details
#' **Interpreting Results**:
#' \itemize{
#'   \item **Pearson correlation**: Measures linear relationship. 
#'     r = 1: perfect positive correlation
#'     r = 0: no linear correlation  
#'     r = -1: perfect negative correlation
#'     |r| > 0.3 generally considered moderate correlation
#'   \item **p-value**: Probability of observing this correlation by chance.
#'     p < 0.05: statistically significant (commonly used threshold)
#'     p < 0.01: highly significant
#'   \item **Sample size**: Larger samples (>100) provide more reliable estimates
#' }
#'
#' **Scenario Selection**:
#' \itemize{
#'   \item **Scenario 1**: One variable vs one variable (e.g., TP53 vs MDM2)
#'   \item **Scenario 2**: One variable vs multiple variables (e.g., TP53 vs c("MDM2", "MYC", "KRAS"))
#'   \item **Scenario 3**: Multiple vs multiple (correlation matrix)
#' }
```

---

### @return (Result Structure + Interpretation Guide)

**Critical: LLM needs to know EXACT return structure to answer user questions**

**Template**:
```r
#' @return List object with analysis results:
#' \describe{
#'   \item{\strong{statistics}}{Data frame with statistical metrics:
#'     \itemize{
#'       \item \code{correlation}: Correlation coefficient (numeric)
#'       \item \code{pvalue}: Statistical significance (numeric)
#'       \item \code{n_samples}: Sample size (integer)
#'       \item \code{method}: Test used (character)
#'     }
#'   }
#'   \item{\strong{plot}}{ggplot object - scatter plot with regression line}
#'   \item{\strong{data}}{Data frame with merged variable values}
#'   \item{\strong{plot_path}}{Character - saved plot file path}
#' }
#'
#' **How to Interpret**:
#' \enumerate{
#'   \item Check \code{result$statistics$correlation} for effect size
#'   \item Check \code{result$statistics$pvalue} for significance
#'   \item View plot: \code{print(result$plot)} or open \code{result$plot_path}
#'   \item Access raw data: \code{result$data} for custom analysis
#' }
#'
#' **What You Can Do Next**:
#' \enumerate{
#'   \item Filter significant results: \code{result$statistics[result$statistics$pvalue < 0.05, ]}
#'   \item Multi-cancer comparison: Re-run with \code{var1_cancers = c("BRCA", "LUAD", "COAD")}
#'   \item Explore related variables: Use \code{search_variables()} to find similar genes
#'   \item Enrichment analysis: \code{tcga_enrichment()} for genome-wide scan
#'   \item Survival analysis: \code{tcga_survival()} to check prognostic value
#' }
#'
#' **Alternative Analyses**:
#' \itemize{
#'   \item \code{\link{tcga_enrichment}}: For genome-wide pathway analysis
#'   \item \code{\link{tcga_survival}}: For prognostic association
#'   \item Different modality: Try CNV, Methylation, or Mutation instead
#' }
```

**Use REAL test results**:
```r
# From test output:
str(result)
# List of 4
#  $ statistics:Classes 'tbl_df', 'tbl' and 'data.frame':	1 obs. of  5 variables:
#   ..$ var1       : chr "TP53 (RNAseq, BRCA)"
#   ..$ var2       : chr "MDM2 (RNAseq, BRCA)"
#   ..$ correlation: num 0.42
#   ..$ pvalue     : num 2.1e-48
#   ..$ n_samples  : int 1095
#  $ plot      : ggplot object
#  $ data      : data.frame with 1095 rows
#  $ plot_path : chr "sltcga_output/correlation_BRCA_TP53_RNAseq_vs_MDM2_RNAseq.png"
```

---

### @examples (Research Question-Driven)

**CRITICAL**: Examples should show research questions, not just syntax

```r
#' @examples
#' \donttest{
#' # ===========================================================================
#' # Example 1: Basic Correlation (TESTED - 3.2 sec, r=0.42, p<0.001)
#' # ===========================================================================
#' # Research Question: Is TP53 expression correlated with MDM2 expression 
#' # in breast cancer?
#' # Expected: Positive correlation (MDM2 is TP53 regulator)
#' 
#' result <- tcga_correlation(
#'   var1 = "TP53", var1_modal = "RNAseq", var1_cancers = "BRCA",
#'   var2 = "MDM2", var2_modal = "RNAseq", var2_cancers = "BRCA",
#'   method = "pearson"
#' )
#' 
#' # Verify result structure
#' result$statistics
#' #   var1           var2           correlation pvalue    n_samples
#' #   TP53 (RNAseq)  MDM2 (RNAseq)  0.42        <0.001    1095
#' 
#' # View plot
#' print(result$plot)  # Shows scatter plot with regression line
#' 
#' # Interpret
#' cat("TP53 and MDM2 show moderate positive correlation (r=0.42)\n")
#' cat("This is statistically significant (p<0.001)\n")
#' 
#' # ===========================================================================
#' # Example 2: Multi-Gene Comparison (Show parameter variation)
#' # ===========================================================================
#' # Research Question: Which genes correlate most strongly with TP53?
#' 
#' result <- tcga_correlation(
#'   var1 = "TP53", var1_modal = "RNAseq", var1_cancers = "BRCA",
#'   var2 = c("MDM2", "MYC", "CDKN1A"), var2_modal = "RNAseq", 
#'   var2_cancers = "BRCA"
#' )
#' 
#' # Compare correlations
#' result$statistics[order(-result$statistics$correlation), ]
#' 
#' # ===========================================================================
#' # Example 3: Multi-Cancer Analysis
#' # ===========================================================================
#' # Research Question: Is TP53-MDM2 correlation consistent across cancer types?
#' 
#' result <- tcga_correlation(
#'   var1 = "TP53", var1_modal = "RNAseq",
#'   var1_cancers = c("BRCA", "LUAD", "COAD"),
#'   var2 = "MDM2", var2_modal = "RNAseq",
#'   var2_cancers = c("BRCA", "LUAD", "COAD")
#' )
#' 
#' # Compare across cancers
#' result$statistics
#' 
#' # ===========================================================================
#' # Example 4: Cross-Modality Analysis
#' # ===========================================================================
#' # Research Question: Does TP53 methylation silence its expression?
#' 
#' result <- tcga_correlation(
#'   var1 = "TP53", var1_modal = "Methylation", var1_cancers = "BRCA",
#'   var2 = "TP53", var2_modal = "RNAseq", var2_cancers = "BRCA",
#'   method = "spearman"  # Use Spearman for methylation (non-normal)
#' )
#' 
#' # Expect negative correlation if methylation silences expression
#' result$statistics$correlation  # Should be negative
#' 
#' # ===========================================================================
#' # Next Steps
#' # ===========================================================================
#' # For complete workflows:
#' # - Genome-wide scan: tcga_enrichment(var1="TP53", analysis_type="genome")
#' # - Survival impact: tcga_survival(var1="TP53", surv_type="OS")
#' # - Explore variables: search_variables("TP53")
#' # - List cancer types: list_cancer_types()
#' }
```

**Why this is better**:
- ✅ Starts with research question (what user actually wants to know)
- ✅ Shows expected biological outcome
- ✅ Includes result interpretation
- ✅ Demonstrates parameter variations (different scenarios)
- ✅ Brief pointers to related analyses

---

### @seealso (Workflow-Oriented)

```r
#' @seealso
#' **Core Analysis Functions**:
#' \itemize{
#'   \item \code{\link{tcga_correlation}}: Correlation/association analysis (7 scenarios)
#'   \item \code{\link{tcga_enrichment}}: Pathway enrichment analysis (8 scenarios)
#'   \item \code{\link{tcga_survival}}: Survival analysis (2 scenarios)
#' }
#'
#' **Data Exploration**:
#' \itemize{
#'   \item \code{\link{list_cancer_types}}: View all 33 cancer types and 32 subtypes
#'   \item \code{\link{list_variables}}: Browse available genes, clinical variables
#'   \item \code{\link{search_variables}}: Search for specific genes or patterns
#'   \item \code{\link{list_immune_cells}}: View immune cell types (22 cell types)
#' }
#'
#' **Helper Functions**:
#' \itemize{
#'   \item \code{\link{tcga_load_modality}}: Manual data loading
#'   \item \code{\link{get_variable_groups}}: Get categorical variable groups
#' }
#'
#' **Database**: \url{https://www.cancer.gov/tcga}
```

---

### @section User Queries (Task-Oriented + Research Questions)

**CRITICAL**: These queries will be embedded for RAG retrieval. Focus on biological research questions!

**Query Pattern Philosophy**:
```
❌ Technical: "How to perform correlation analysis in TCGA?"
✅ Research:  "What is the correlation between TP53 and MDM2 expression?"

❌ Method-focused: "How to use tcga_correlation()?"
✅ Question-focused: "Are TP53 mutations associated with higher TMB?"

❌ Database-centric: "Which function analyzes TCGA data?"
✅ Biology-centric: "Does TP53 methylation correlate with its mRNA expression?"
```

**Query Types (Generate 25-35 queries)**:

```r
#' @section User Queries:
#' \itemize{
#'   \item What is the correlation between TP53 mRNA and protein levels?
#'   \item Does TP53 protein abundance correlate with its mRNA expression?
#'   \item What are the phosphorylation sites of AKT1 protein?
#'   \item Does AKT1 protein level correlate with its phosphorylation?
#'   \item Which AKT1 phosphorylation sites correlate with protein abundance?
#'   \item Is PIK3CA mutation associated with AKT1 phosphorylation?
#'   \item Does EGFR mutation affect EGFR protein phosphorylation?
#'   \item What phosphorylation events are affected by TP53 mutation?
#'   \item Is mRNA-protein correlation consistent across cancer types?
#'   \item Which proteins show poor mRNA-protein correlation?
#'   \item Does MTOR protein correlate with its downstream phosphorylation?
#'   \item Are AKT1 and MTOR phosphorylation sites correlated?
#'   \item What phosphorylation changes occur in PIK3CA mutant tumors?
#'   \item Does tumor stage correlate with protein phosphorylation?
#'   \item Is there cross-talk between AKT1 and ERK phosphorylation?
#'   \item Which phosphorylation sites predict survival?
#'   \item Does RPS6 phosphorylation indicate mTOR pathway activation?
#'   \item Are KRAS and EGFR mutations mutually exclusive?
#'   \item What proteins correlate with TP53 protein levels?
#'   \item Does ERBB2 protein correlate with its mRNA in breast cancer?
#'   \item Which phosphorylation sites are affected by kinase mutations?
#'   \item Is protein stability related to mRNA-protein discordance?
#'   \item What pathway proteins show coordinated phosphorylation?
#'   \item Does age correlate with global phosphorylation levels?
#'   \item Are there gender differences in protein expression?
#'   \item Which proteins drive survival in pancreatic cancer?
#'   \item Does VHL mutation affect HIF1A protein levels?
#'   \item What protein-phospho patterns distinguish cancer subtypes?
#'   \item Is STAT3 phosphorylation correlated with immune signatures?
#'   \item Which phosphorylation sites are druggable targets?
#'   \item Does protein phosphorylation predict treatment response?
#'   \item What proteins show post-translational regulation?
#'   \item Are phosphorylation networks rewired in mutant tumors?
#' }
```

**Why 25-35 queries**:
- Covers research question diversity
- Includes cross-modality questions (methylation-expression, CNV-expression)
- Shows multi-cancer scenarios
- Mentions survival and enrichment (for cross-function linking)
- Uses specific gene examples (TP53, EGFR, PIK3CA)

---

## 🤖 AI AGENT WORKFLOW FOR STATISTICAL PACKAGES

### Phase 1: Code Understanding

```
For Statistical Functions:
1. What statistical question does it answer?
2. Which scenarios (1-17) are covered?
3. What are the data modalities supported?
4. What variable type combinations trigger which scenario?
5. What statistical tests are used?
6. What visualization is generated?

Example for tcga_correlation():
- Question: Association between variables
- Scenarios: 1-7 (continuous-continuous, categorical-continuous, categorical-categorical)
- Modalities: RNAseq, Mutation, CNV, Methylation, miRNA, Clinical, ImmuneCell, Signature
- Tests: Pearson, Spearman, Wilcoxon, Kruskal-Wallis, Chi-square, Fisher
- Visualization: Scatter, Box, Bar, Heatmap
```

---

### Phase 2: Testing (CRITICAL - MUST PASS)

**Test Script Template**:

```r
# Test: tcga_correlation()
library(SLTCGA)

cat("=== Testing tcga_correlation ===\n")

# Test 1: Basic correlation (Scenario 1)
cat("\nTest 1: TP53-MDM2 correlation in BRCA...\n")
start <- Sys.time()
result1 <- tcga_correlation(
  var1 = "TP53", var1_modal = "RNAseq", var1_cancers = "BRCA",
  var2 = "MDM2", var2_modal = "RNAseq", var2_cancers = "BRCA",
  method = "pearson"
)
t1 <- difftime(Sys.time(), start, units = "secs")

cat("✓ Success!\n")
cat("  Runtime:", round(t1, 2), "sec\n")
cat("  Correlation:", round(result1$statistics$correlation, 3), "\n")
cat("  P-value:", format(result1$statistics$pvalue, digits=3), "\n")
cat("  Sample size:", result1$statistics$n_samples, "\n")
cat("  Plot saved:", result1$plot_path, "\n")

# Test 2: Multi-gene analysis (Scenario 2)
cat("\nTest 2: TP53 vs multiple genes...\n")
start <- Sys.time()
result2 <- tcga_correlation(
  var1 = "TP53", var1_modal = "RNAseq", var1_cancers = "BRCA",
  var2 = c("MDM2", "MYC"), var2_modal = "RNAseq", var2_cancers = "BRCA"
)
t2 <- difftime(Sys.time(), start, units = "secs")

cat("✓ Success!\n")
cat("  Runtime:", round(t2, 2), "sec\n")
cat("  Results:", nrow(result2$statistics), "correlations\n")

# Test 3: Cross-modality (methylation-expression)
cat("\nTest 3: TP53 methylation vs expression...\n")
start <- Sys.time()
result3 <- tcga_correlation(
  var1 = "TP53", var1_modal = "Methylation", var1_cancers = "BRCA",
  var2 = "TP53", var2_modal = "RNAseq", var2_cancers = "BRCA",
  method = "spearman"
)
t3 <- difftime(Sys.time(), start, units = "secs")

cat("✓ Success!\n")
cat("  Runtime:", round(t3, 2), "sec\n")
cat("  Correlation:", round(result3$statistics$correlation, 3), 
    "(expect negative)\n")

cat("\n=== All tests passed ===\n")
```

**What to save from test**:
- ✅ Runtime for each scenario
- ✅ Statistical values (correlation, p-value, sample size)
- ✅ Output file paths
- ✅ Result structure (for @return documentation)

---

### Phase 3: Documentation Writing

**Use this checklist**:

```
□ @title: [Analysis Type] - [Data Types]
□ @description: 4 sentences (Analysis + Data + Output + Features)
□ @param: All parameters (Type + Options + Scenario impact)
□ @return: Exact structure (statistics + plot + data + paths) from test
□ @section Performance Test: Runtime + Sample size + Results (real test)
□ @section User Queries: 25-35 research question queries
□ @examples: Research questions + Expected outcomes + Interpretation
□ @seealso: Core functions + Exploration + Helpers + Database
□ @references: Statistical tests + Database citation
□ @details: Statistical interpretation + Scenario selection guide
```

---

### Phase 4: Validation

**Run these checks**:

```bash
# Check 1: No Chinese characters
grep -n "[\u4e00-\u9fff]" R/<file>.R

# Check 2: Has @references with statistical method
grep -A 8 "@references" R/<file>.R | grep -i "pearson\|spearman\|wilcoxon\|kruskal"

# Check 3: Has @section Performance Test
grep "@section Performance Test" R/<file>.R

# Check 4: Has @section User Queries with research questions
grep -A 30 "@section User Queries" R/<file>.R | wc -l  # Should be 30+ lines

# Check 5: Examples use \donttest
grep -A 2 "@examples" R/<file>.R | grep "donttest"

# Check 6: Test script exists and runs
Rscript tests/test_<function>.R
```

---

### Phase 5: Report (中文输出)

```
✅ 文档优化完成：<function_name>

【函数类型】：统计分析函数 (Scenario <X-Y>)
【支持场景】：<场景列表>
【测试结果】：
  - Scenario 1测试：<X> 秒，r=<correlation>, p=<pvalue>
  - Scenario 2测试：<Y> 秒，<N>个相关性
  - 样本量：<N>个样本

【关键文档】：
  - @title: <Analysis Type> - <Data Types>
  - @description: <4句话总结>
  - @param: <N>个参数（场景导向）
  - @return: statistics + plot + data + paths（真实结构）
  - @section Performance Test: <真实测试结果>
  - @section User Queries: <N>个研究问题
  - @references: <Statistical tests + Database>

【验证检查】：
  ✓ 无中文字符
  ✓ 有 @references（统计方法 + 数据库）
  ✓ 有 @section Performance Test（真实测试）
  ✓ 有 @section User Queries（25-35个）
  ✓ @examples 用 \donttest{} 包裹
  ✓ 测试成功通过
```

---

## 📊 QUALITY CHECKLIST

**After writing documentation, verify**:

### Testing (CRITICAL)
- [ ] Function test executed successfully
- [ ] Runtime documented with scenario details
- [ ] Statistical results included (correlation, p-value, etc.)
- [ ] Output files verified (plot, data, statistics)

### Content Quality (REQUIRED)
- [ ] No Chinese characters
- [ ] No setup noise (≤1 sentence if needed)
- [ ] Has @references with statistical test + database
- [ ] Has @section Performance Test with real results
- [ ] @description is 3-4 sentences (60-100 words)

### Structure (REQUIRED)
- [ ] @title: [Analysis Type] - [Data Types]
- [ ] @description includes: Analysis + Data + Output + Features
- [ ] @param: All parameters with scenario impact
- [ ] @return: Exact structure (statistics + plot + data)
- [ ] @section User Queries: 25-35 research questions
- [ ] @examples: Research questions + Interpretation
- [ ] @seealso: Core + Exploration + Helpers
- [ ] @details: Statistical interpretation

### LLM RAG Readiness (CRITICAL)
- [ ] @description clearly describes which scenarios
- [ ] @section User Queries covers research questions (not method queries)
- [ ] @examples show biological interpretation
- [ ] @return tells LLM how to extract answers
- [ ] @details explains statistical results

---

## 🎯 REMEMBER: THE RAG WORKFLOW

**Users ask research questions, not statistical methods!**

```
User: "TP53表达和TMB之间有什么关系?"
      ❌ NOT: "如何用tcga_correlation分析相关性?"
      ❌ NOT: "哪个函数可以算相关?"
    ↓
┌─────────────────────────────────────────┐
│ RAG Retrieval                          │
├─────────────────────────────────────────┤
│ Embedding 1: @title + @description     │
│   → Finds: tcga_correlation()          │
│                                         │
│ Embedding 2: @section User Queries     │
│   → Match: "TP53 expression and TMB"   │
│                                         │
│ Return: tcga_correlation() docs        │
└─────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────┐
│ LLM Code Generation                    │
├─────────────────────────────────────────┤
│ Reads: @param (var1, var1_modal...)    │
│ Reads: @examples (pattern)              │
│                                         │
│ Generates:                              │
│ result <- tcga_correlation(             │
│   var1 = "TP53",                        │
│   var1_modal = "RNAseq",                │
│   var1_cancers = "BRCA",                │
│   var2 = "TMB",                         │
│   var2_modal = "Signature",             │
│   method = "spearman"                   │
│ )                                       │
└─────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────┐
│ Execution + Answer Generation          │
├─────────────────────────────────────────┤
│ Reads: @return structure                │
│ Reads: @details interpretation          │
│                                         │
│ LLM answers:                            │
│ "TP53表达与TMB呈正相关 (r=0.31,        │
│  p<0.001)。这表明TP53高表达的肿瘤      │
│  往往有更高的突变负荷。"                │
│                                         │
│ Reads: "What You Can Do Next"          │
│ Suggests: tcga_survival() for          │
│ prognostic value                        │
└─────────────────────────────────────────┘
```

**核心原则**：
1. 测试必须成功（真实统计结果）
2. 英文 only
3. @description 必须说明覆盖哪些场景
4. @section User Queries 必须是研究问题（不是方法问题）
5. @details 必须解释统计结果含义

---

**End of SLTCGA/SLCPTAC Documentation Rules**

---
> Source: [SolvingLab/SLCPTAC](https://github.com/SolvingLab/SLCPTAC) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
