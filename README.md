# Synthetic-Structured Data Generation: Preserving statistical distribution of source data.

## Project Overview  
This project demonstrates a workflow for generating high-quality synthetic data that mirrors the statistical properties of real-world tabular data (using Lending Club loan data as an example). The workflow leverages clustering-based seed instructions and the Synthetic Data Studio (SDS) API to produce realistic, privacy-compliant synthetic datasets. The process is divided into four core steps:  

0. **Setup & Library Installation**  
1. **Data Preparation & Seed Instruction Generation**  
2. **Synthetic Data Generation & Filtering**  
3. **Synthetic Data Evaluation & Distribution Comparison**  

---

## Step 0: Setup & Library Installation  
**Overview**  
Installs required Python libraries for data analysis and synthetic data generation. 
Note [Synthetic Data Studio](https://github.com/cloudera/CAI_AMP_Synthetic_Data_Studio) also needs to be installed as well by the user.

**Purpose**  
- Ensure all dependencies are available for subsequent steps  

**Key Components**  
- Installs libraries: `hvplot`, `scikit-learn`, `seaborn`, `scipy`, `matplotlib`  

**Output**  
- Ready-to-use environment for data analysis and synthetic data workflows  

**Note**
[Synthetic Data Studio](https://github.com/cloudera/CAI_AMP_Synthetic_Data_Studio) needs to be installed in the same workspace by the user.

---

## Step 1: Data Preparation & Seed Instruction Generation  
**Overview**  
Processes the original Lending Club loan dataset to generate statistical profiles for clustering-based synthetic data generation.  

**Purpose**  
- Clean and preprocess raw data  
- Create statistical seed instructions for synthetic data generation  
- Generate cluster-based statistical profiles for distribution matching  

**Key Components**  
1. **Data Cleaning**  
   - Removes redundant features (`emp_title`, `emp_length`, `title`)  
   - Handles missing values and categorical encoding  
   - Extracts zip code from address field  
2. **Clustering**  
   - Uses KMeans (2 clusters) on key financial metrics  
   - Creates statistical profiles for each cluster (mean, std, skew, kurtosis)  
3. **Seed Instruction Generation**  
   - Captures correlations between variables  
   - Preserves business rules (grade-subgrade alignment, mortgage consistency)  

**Variables to Customize**  
- `SamplingFactor`: Controls output size (default `20`)  
- `NumOfClusters`: Number of clusters for statistical profiling (default `2`)  
- `CorrelationKeys`: Variables to analyze for correlation patterns

**Output**  
- `SeedsInstructions.json`: Cluster-based statistical profiles for synthetic data generation  

---

## Step 2: Synthetic Data Generation & Filtering  
**Overview**  
Uses the seed instructions to generate synthetic data via an API and applies quality filters.  

**Purpose**  
- Generate synthetic data matching the original dataset's statistical properties  
- Filter low-quality outputs based on formatting, cross-column consistency, and realism  

**Key Components**  
1. **Synthetic Data Generation**  
   - Uses SDS API with temperature `1.0` for diversity  
   - Enforces formatting rules (date formats, decimal precision)  
   - Applies business rules (grade-subgrade alignment, mortgage consistency)  
2. **Filtering & Validation**  
   - Removes malformed data (1.29% bad formatting)  
   - Finds installment errors and corrects them using the EMI formula that depends on the term, loan_amount, and int_rate.
   - LLM-based evaluation (score threshold `8/10`)  

**Variables to Customize**  
- `temperature`: Controls diversity (default `1.0`)  
- `Threshold`: Quality score threshold (default `8`)  
- `example_path`: Path to example data for pattern learning  
- `url`: point to the SDS freeform synthesis API
- `url_eval`: point to the SDS freeform evaluation API

**Output**  
- `data_tab_separated.csv`: Filtered synthetic data matching original distributions  

---

## Step 3: Synthetic Data Evaluation & Distribution Comparison  
**Overview**  
Compares synthetic data to original data using statistical and visual analysis.  

**Purpose**  
- Validate that synthetic data preserves key statistical properties  
- Identify discrepancies in distributions and correlations  

**Key Components**  
1. **Statistical Comparison**  
   - Mean/standard deviation comparison for continuous variables  
   - KL divergence analysis for categorical variables  
2. **Visual Analysis**  
   - Box plots for loan amount, interest rate, and installment  
   - Histograms for credit lines, public records, and bankruptcies  
3. **Correlation Analysis**  
   - Compares correlations between variables (e.g., `mort_acc` vs. `loan_amnt`)  

**Variables to Customize**  
- `ContinuousVariables`: Features to analyze for statistical comparison  
- `AllCategorical`: Categorical variables for KL divergence analysis  

**Output**  
- Visualizations and statistical tables comparing synthetic vs. original data  
- `data_tab_separated.csv`: Final synthetic dataset for downstream use  

---

## Setup & Installation  
1. **Environment requirements:**  
   - Python 3.10+  
   - Access to SDS API endpoints  

2. **Tested models:**  
   - LLM: `meta/llama-3.1-70b-instruct` and `AWS bedrock models` for generation  
   - Evaluation LLM: `meta/llama-3.1-70b-instruct` and `AWS bedrock models` 

---

## Usage Guide  
1. **Run Step 0** to install dependencies.  
2. **Execute Step 1** to preprocess data and generate seed instructions.  
3. **Proceed to Step 2** to generate and filter synthetic data.  
4. **Run Step 3** to evaluate synthetic data quality.  

---

## Advanced Usage Guide  
1. **Custom input data:**  
   - Replace `data_tab_separated-large_synthetic.csv` with your dataset  
   - Update feature engineering logic to match your data schema  

2. **Custom model choice:**  
   - Modify API requests in Step 2 to use different LLMs (e.g., AWS Bedrock)  

3. **Custom evaluation criteria:**  
   - Adjust `Threshold` in Step 2 to balance quality and quantity  
   - Add custom rules to the LLM evaluation prompt  

---

## Expected Outputs  
- **Step 1**: `SeedsInstructions.json` with cluster-based statistical profiles  
- **Step 2**: `data_tab_separated.csv` containing high-quality synthetic data  
- **Step 3**: Visualizations and statistical tables comparing synthetic vs. original data  

---

## Testing Correct Execution  
- Run all steps with default parameters  
- Synthetic data should approximately match original distributions as shown in the last notebook.

---

## Known Issues  
- Address formatting may not perfectly match seed instructions (1-2% of samples)  
- Infrequent categories (e.g., "OWN" home ownership) may be underrepresented in synthetic data  
- Requires API access to SDS endpoints for synthetic data generation

