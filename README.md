# VAERS DSA Pipeline

Three Java projects exploring data structures and algorithms applied to the CDC's [VAERS](https://vaers.hhs.gov/data.html) (Vaccine Adverse Event Reporting System) COVID-19 dataset — roughly 1.6 million records. Built for CSC 365 (Data Structures & Algorithms).

# Project 1: Sorting Algorithm Benchmark

Implements and benchmarks three sorting algorithms — Quick Sort, Merge Sort, and Insertion Sort — on VAERS COVID-19 case records, sorted by `VAERSID`.

*Pipeline
- Filtered raw VAERS CSVs down to records with a COVID-19 history
- Parsed CSV input with [Apache Commons CSV](https://commons.apache.org/proper/commons-csv/)
- Randomized record order with `Collections.shuffle()` before each run to remove pre-sorted bias
- Measured wall-clock sort time across **1,615,925** records

*Results
| Algorithm | Avg. Case Complexity | Time to Sort |
|---|---|---|
| Quick Sort | O(n log n) | 3,699 ms |
| Merge Sort | O(n log n) | 2,089 ms |
| Insertion Sort | O(n²) | 9,992,331 ms (~2.8 hrs) |

Insertion Sort's blow-up at this scale is a good demonstration of why O(n²) algorithms become impractical on million-row datasets.


# Project 2: B+ Tree Indexing

Implements a custom B+ Tree to index VAERS records by `VAERSID`, loading multiple years of data (2020–2025) produced by Project 1.

*Key Features**
- Custom node splitting (leaf and internal) with parent-child pointer management for dynamic balanced indexing
- Search by `VAERSID` in O(log n)
- Incremental loading: checks existing keys before insertion so newer/updated yearly files (e.g. 2025) don't create duplicates
- O(log n) for insert, search, and computing tree height

*Sample Run** (max degree = 1000)

| File | Load Time |
|---|---|
| VAERSCOVID2020.csv | 252 ms |
| VAERSCOVID2021.csv | 16,094 ms |
| VAERSCOVID2022.csv | 5,858 ms |
| VAERSCOVID2022.csv (2nd load) | 5,581 ms |
| VAERSCOVID2023.csv | 1,815 ms |
| VAERSCOVID2024.csv | 471 ms |
| VAERSCOVID2025.csv | 25 ms |
| **Total** | **30,096 ms** |

> Note: 2022 appears twice in the original console output — worth double-checking whether that's intentional (e.g. testing the incremental-update logic) or a duplicate run.


# Project 3: Apriori Association Rule Mining

Mines frequent symptom co-occurrence patterns from VAERS reports using the Apriori algorithm — each report is treated as a "transaction" and each reported symptom as an "item."

*Preprocessing
- Used `SYMPTOM1`–`SYMPTOM5` columns (covers all symptoms recorded per report)
- Dropped entries with null, missing, or very short (≤3 character) symptom text
- Padded missing symptom slots with empty strings to keep CSV output uniform

*Why Apriori
Chosen for its interpretable rule structure — useful for medical/adverse-event data where transparency matters — and because it works natively on categorical data without needing embeddings or distance functions.

*Parameters

| Parameter | Value | Rationale |
|---|---|---|
| Minimum Support | 0.01 | Keeps symptoms appearing in ≥1% of ~1.6M reports |
| Minimum Confidence | 0.05 | Surfaces moderate-strength rules without overfitting to noise |

*Dataset Summary
- Records processed: 1.6 million
- Transactions used: 1,603,049
- CSV merge time: 31.67 s
- Apriori mining time: 61.76 s
- Frequent 1-itemsets (≥1% support): 40+

*Top Frequent Symptoms

| Symptom | Support |
|---|---|
| SARS-CoV-2 test | 13.9% |
| COVID-19 | 11.0% |
| Pyrexia, Headache, Fatigue, Pain | <10% each |
