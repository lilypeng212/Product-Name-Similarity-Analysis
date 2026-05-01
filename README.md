# Product Name Similarity Analysis

## 📌 Problem
As the e-commerce platform expanded its third-party (3P) marketplace, product names across 1P and 3P assortments often contained highly similar or duplicated descriptions. This made it difficult to identify overlapping products, evaluate assortment gaps, and improve product-level data quality.

---

## 💡 Solution
Built a product name similarity analysis workflow in R to compare 3P product names against existing 1P product assortments using fuzzy matching and Jaccard similarity.

The workflow included three stages:

### Version 1 — Baseline Similarity Matching
- Compared 3P product names with existing 1P product names using Jaccard distance
- Excluded exact matches with the same product name and product code
- Identified the closest matching 1P product for each 3P product
- Exported matched product pairs and similarity scores to Excel for review

### Version 2 — Parallelized Matching for Large-scale Data
- Rebuilt the matching logic using `stringdist()` for more flexible distance calculation
- Applied multi-core parallel processing with `future.apply`
- Added progress tracking for long-running product matching tasks
- Improved scalability for larger product datasets

### Version 3 — Category-constrained Matching
- Restricted matching within the same department and first-level category
- Reduced false matches caused by cross-category product name similarity
- Improved matching precision by comparing products within more relevant product groups
- Joined matched results back to original product attributes for further review

---

## 🔍 Highlights
- Applied fuzzy matching to identify highly similar 1P and 3P product names
- Used Jaccard similarity to support text-based product comparison
- Improved matching scalability through parallel processing
- Reduced irrelevant matches by adding department and category-level constraints
- Produced Excel outputs for business review and product data quality checks

---

## 📈 Impact
- Supported product assortment analysis by identifying overlapping or highly similar products
- Improved visibility into 1P / 3P product duplication and assortment similarity
- Reduced manual effort in product name comparison and review
- Provided a reusable workflow for product data cleaning and marketplace expansion analysis

---

## 🛠 Tools
- R
- dplyr
- stringdist
- future.apply
- progressr
- readxl / openxlsx
- Fuzzy Matching
- Product Data Cleaning

---

## ⚠️ Disclaimer
This repository uses sanitized descriptions and synthetic examples only.  
No proprietary data, client information, internal file paths, product IDs, or confidential business logic are included.
