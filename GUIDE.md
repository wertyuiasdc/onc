# 🧭 User's Guide: Clinical Intelligence Agent Backend

**Welcome!** This repository contains the end-to-end backend for a Secure Clinical AI.
Follow this "Lab Manual" to step through the architecture from **Data Ingestion** to **AI Safety**.

> **Prerequisite:** Ensure you are running this in **GitHub Codespaces** (recommended) or have Python 3.10+ and VS Code installed locally.

---

## 🛠️ Step 0: Environment Setup
Before we touch data, let's make sure our tools are sharp.

1.  **In VS Code open the Terminal:**
    * Press `Ctrl + ~` (Windows) or `Cmd + ~` (Mac).
2.  **Install Dependencies:**
    Run this command to install the required Python libraries:
    ```bash
    pip install -r requirements.txt
    ```
3.  **Configure Secrets (Crucial):**
    * Rename `.env.template` to `.env`.
    * Paste your Azure credentials (Workspace ID, Connection Strings) into `.env` if you already know the values.
    * *Note: Without this, some cells in the notebooks may fail or run in "Simulation Mode" or fail to connect to the cloud.*

---

## 🥉 Step 1: Data Ingestion (The Bronze Layer)
**Goal:** We need to fetch raw oncology data from the public internet (cBioPortal) and land it securely in our Private Cloud (OneLake).

1.  **Open the Notebook:**
    👉 [**Click to Open `01_data_ingestion/1_ingestion_pipeline.ipynb`**](./01_data_ingestion/1_ingestion_pipeline.ipynb)

2.  **What to do:**
    * Read the "Concept Guides" in the Markdown cells.
    * Run the code cells to see how we verify the Azure connection (`az login`) and stream the download.
    * *Output:* You will see a progress bar (`tqdm`) as files are uploaded to the `Raw/` folder in OneLake.

---

## 🥈 Step 2: Transformation (The Silver Layer)
**Goal:** Raw data is messy. We use **PySpark** to clean it and structure it into Delta Tables.

* *Note:* This step typically runs inside the **Microsoft Fabric** SaaS environment, not locally in VS Code.
* **Inspect the Code:**
    👉 [**View `fabric_code/bronze_to_silver_etl.py`**](./fabric_code/bronze_to_silver_etl.py)
* **Key Takeaway:** Notice how we use `mssparkutils` to mount the lakehouse and clean column names before saving to Delta format.

---

## 📊 Step 3: Analysis & Validation (The Gold Standard)
**Goal:** Now that we have clean data, we must prove it is "Organic" (not artificially capped) before training AI on it.

1.  **Open the Notebook:**
    👉 [**Click to Open `02_backend_logic/2_data_analysis.ipynb`**](./02_backend_logic/2_data_analysis.ipynb)

2.  **What to do:**
    * Click **Run All**.
    * This notebook connects via **ODBC/SQL** to the Silver Delta Tables we created in Step 2.
    * **The "Aha!" Moment:** Scroll to **Step 4**. Look at the histogram. You should see a *smooth tail* (patients surviving 80+ months), proving there is no artificial "60-month wall."

---

## 🛡️ Step 4: Safety Guardrails (The Shield)
**Goal:** Finally, we test the Middleware that sits between the User and the AI to prevent data leaks.

1.  **Run the Test Suite:**
    Paste this command into your terminal to run the automated compliance checks:
    ```bash
    python -m safety_middleware.test_guardrails
    ```

2.  **Verify the Output:**
    You should see the system blocking unsafe prompts:
    * `Scanning: '...dosage for Aspirin?'` → **✅ PASSED**
    * `Scanning: '...patient TCGA-OR-A5J1...'` → **⚠️ BLOCKING: PII Detected**
    * `Scanning: '...hack the mainframe?'` → **⛔ BLOCKING: Content Violation**

---
*Project by **IAYF Consulting** | [View on GitHub](https://github.com/CarnegieJ/onc-clinical-intel-agent)*