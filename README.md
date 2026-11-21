# T20 International Cricket Analytics

The [main notebook](T20_Big_Data_Analytics.ipynb) is the implementation. It includes the code, worked examples, figures, printed rules and measured results. Open it with Jupyter to review the saved outputs.

## Dataset

The six original CSVs are in `data/raw/`. Bhuvanesh Prasad published them on Kaggle under the [Apache 2.0 license](data/raw/LICENSE). The assigned Kaggle URL still ends in `2005-to-2023`. Its current title is 2005–2024, and the CSVs contain dated matches through **7 May 2024**. The [dataset manifest](outputs/tables/dataset_manifest.csv) records sizes and file hashes. [Assigned Kaggle dataset](https://www.kaggle.com/datasets/bhuvaneshprasad/all-t20-internationals-dataset-2005-to-2023)

If you need to restore the originals, download the ZIP from the dataset page and extract its six CSVs directly into `data/raw/`.

## Run the notebook

Use CPython 3.11 and OpenJDK 17. From the project root in PowerShell, run the commands below. Set `JAVA_HOME` if Java is missing from `PATH`. If `.venv` already has the project dependencies, run the last command.

```powershell
py -3.11 -m venv .venv
& '.\.venv\Scripts\python.exe' -m pip install -r requirements.txt
& '.\.venv\Scripts\jupyter-execute.exe' --inplace --timeout=1800 T20_Big_Data_Analytics.ipynb
```

For interactive review, open the notebook in your Jupyter editor.

From a macOS or Linux shell, use `python3.11 -m venv .venv` and the corresponding `.venv/bin/python` and `.venv/bin/jupyter-execute` commands. Run from the project root so the relative paths resolve.

## Where to find results

| Path | Contents |
| --- | --- |
| `data/raw/` | The six original Kaggle CSVs |
| `data/processed/` | Cleaned tables, consolidated innings, transactions, classification records and clustered player profiles |
| `outputs/figures/` | Sixteen descriptive PNGs, plus model, clustering and runtime figures |
| `outputs/tables/` | Data audits, statistics, itemsets, rules, model metrics, cluster profiles and individual benchmark runs |

The four added displays show India–Pakistan first-innings scores in a stem-and-leaf plot, first-innings totals in a quantile plot, corresponding innings percentiles in a Q–Q plot, and one dot per match for wickets lost at the 2022 Men's T20 World Cup. Each plot has a short numerical observation in the notebook. [Han, Kamber and Pei, *Data Mining: Concepts and Techniques*, 3rd ed., Chapter 2](https://www.sciencedirect.com/book/monograph/9780123814791/data-mining-concepts-and-techniques)

## How to interpret the results

All frequent itemsets are closed at 10%, 20% and 30% support, so CHARM compresses the pattern count by **0%** at each cutoff. The independent closure check verified this property of the chosen transactions. Spark SQL, Python DIC and Apriori agree on the original support counts.

The chase model uses information available at the first-innings break. The evaluation cohort was selected retrospectively using completed-match information to identify standard decisive games. A live set of matches can include games that later become shortened, abandoned or tied, so the reported metrics apply to the selected cohort. Entropy remains the selected tree because the validation results tied and the predefined tie-break favored entropy. The later test period supplies the reported metrics.

## Benchmarks

Spark SQL counts repeated transaction masks once per block and caches their counts. It computes support with built-in JVM expressions, so each checkpoint can avoid recreating a Python broadcast and running a Python task on each partition. The original RDD counter is rerun once in the same session for comparison. [RDD and SQL timings](outputs/tables/spark_dic_backend_comparison.csv)

The main benchmark reports medians of two runs. It goes up to 73,728 transactions for DIC and 58,752 fit rows plus 14,720 prediction rows for the Decision Tree. These larger inputs repeat records for timing. The evaluation metrics use the original held-out matches. Spark ran with four local worker threads and shared the machine's CPU and memory. [Current benchmark](outputs/tables/python_spark_benchmarks.csv) · [Previous benchmark](outputs/tables/python_spark_benchmarks_before_review.csv)

On the 4,608 original transactions, warm Spark DIC fell from **49.69 seconds with RDDs to 2.88 seconds with SQL**. End-to-end time fell from 64.55 to 6.83 seconds. Both counters used 15 checkpoints and returned the same supports. Spark SQL cached 244 block-and-mask pairs. [RDD and SQL timings](outputs/tables/spark_dic_backend_comparison.csv)

Spark took longer than local Python at every measured size. Warm speedup (Python time divided by Spark time) rose from 0.017× on the original DIC workload to 0.310× at 16×. The replicated DIC workload reuses the same masks, so the cached table stays small as the transaction count grows. For the tree, warm speedup rose from 0.011× to 0.225× at 32×. At that largest tree size, Spark spent 11.83 seconds on input preparation and 1.15 seconds on fitting and prediction. Input conversion, scheduling and output collection stay in the measured costs. These timings describe one machine and workloads with repeated records. [Benchmark medians](outputs/tables/python_spark_benchmarks.csv)

## Execution and verification

The full notebook completed all 60 code cells from a fresh kernel in about 4 minutes 24 seconds using Python 3.11.14 and PySpark 3.5.8. The saved notebook retains the outputs. Verification checked the six raw files, ten processed files, all 26 plots and their observations, the existing analytical results, and all new benchmark medians. Spark SQL DIC matched Python's supports and checkpoint statistics at all three tested thresholds.
