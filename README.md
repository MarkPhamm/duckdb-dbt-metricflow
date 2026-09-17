# duckdb-dbt-metricflow

Testing MetricFlow, dbt exposures, and dbt on DuckDB.

This branch runs the sample project on **dbt v2.0.4** (GA). DuckDB is supported as a **CLI-only** adapter in v2.

## Set up (dbt v2)

Copy the example profile, then install one or both v2 distributions in isolated environments. Both binaries are named `dbt`, so they cannot share a venv.

```bash
cp profiles-example.yml profiles.yml
mkdir -p database

# Recommended distribution (formerly Fusion)
uv venv .venvs/dbt --python 3.12
uv pip install --python .venvs/dbt/bin/python dbt

# Apache-only distribution (formerly Core v2)
uv venv .venvs/dbt-oss --python 3.12
uv pip install --python .venvs/dbt-oss/bin/python dbt-oss

# MetricFlow still uses a Python adapter to query DuckDB
uv venv .venvs/mf --python 3.12
uv pip install --python .venvs/mf/bin/python 'dbt-metricflow[dbt-duckdb]'
```

Use a relative DuckDB path (`./database/mf_tutorial.duckdb`) in `profiles.yml`. Then:

```bash
export DBT_PROFILES_DIR="$PWD"

.venvs/dbt/bin/dbt --version          # dbt 2.0.4
.venvs/dbt-oss/bin/dbt --version      # dbt-oss 2.0.4

.venvs/dbt/bin/dbt deps
.venvs/dbt/bin/dbt debug
.venvs/dbt/bin/dbt build
.venvs/dbt/bin/dbt show --select transactions --limit 5
.venvs/dbt/bin/dbt lint
.venvs/dbt/bin/dbt docs generate
```

`dbt-oss` can parse/debug/build the same project. Native `dbt lint` and `dbt sl` live in the full `dbt` distribution. `dbt sl validate` needs a dbt platform project (`dbt_cloud.yml`); locally, use MetricFlow instead:

```bash
.venvs/mf/bin/mf validate-configs
.venvs/mf/bin/mf list metrics
.venvs/mf/bin/mf query --metrics transactions --group-by metric_time --order metric_time
```

Semantic models were migrated to the latest YAML spec (`semantic_model` nested under each model). `dbt parse` may warn `Skipping semantic manifest validation due to: No dbt_cloud.yml config`; that is expected without a dbt platform login.

<img width="1563" height="1128" alt="image" src="https://github.com/user-attachments/assets/7d28a936-43e1-443d-a766-85b14903707a" />

## 1. Metricflow


🤓 Please run the following steps:

1. Run `dbt debug` to test the connection
2. Run `dbt build` to seed tables and produce artifacts.

3. Try validating your data model:

   ```bash
   mf validate-configs
   ```

4. Check out your metrics:

   ```bash
   mf list metrics

    ✔ 🎉 Successfully parsed manifest from dbt project
    ✔ 🎉 Successfully validated the semantics of built manifest (ERRORS: 0, FUTURE_ERRORS: 0, WARNINGS: 2)
    ✔ 🎉 Successfully validated semantic models against data warehouse (ERRORS: 0, FUTURE_ERRORS: 0, WARNINGS: 0)
    ✔ 🎉 Successfully validated dimensions against data warehouse (ERRORS: 0, FUTURE_ERRORS: 0, WARNINGS: 0)
    ✔ 🎉 Successfully validated entities against data warehouse (ERRORS: 0, FUTURE_ERRORS: 0, WARNINGS: 0)
    ✔ 🎉 Successfully validated measures against data warehouse (ERRORS: 0, FUTURE_ERRORS: 0, WARNINGS: 0)
    ✔ 🎉 Successfully validated metrics against data warehouse (ERRORS: 0, FUTURE_ERRORS: 0, WARNINGS: 0)
    (.venv) ┌─(~/personal/project/duckdb-dbt-metricflow)───
    ```

5. Check out dimensions for your metric:

   ```bash
   mf list dimensions --metrics transactions

   ✔ 🌱 We ve found 13 metrics.
    The list below shows metrics in the format of "metric_name: list of available dimensions"
    • alterations: customer__country__region, customer__customer_country, customer__ds, metric_time, transaction__ds and 2 more
    • cancellation_rate: customer__country__region, customer__customer_country, customer__ds, metric_time, transaction__ds and 2 more
    • cancellations: customer__country__region, customer__customer_country, customer__ds, metric_time, transaction__ds and 2 more
    • cancellations_mx: customer__country__region, customer__customer_country, customer__ds, metric_time, transaction__ds and 2 more
    • cumulative_transactions: customer__country__region, customer__customer_country, customer__ds, metric_time, transaction__ds and 2 more
    • cumulative_transactions_in_last_7_days: customer__country__region, customer__customer_country, customer__ds, metric_time, transaction__ds and 2 more
    • new_customers: country__region, customer__customer_country, customer__ds, metric_time
    • quick_buy_amount_usd: customer__country__region, customer__customer_country, customer__ds, metric_time, transaction__ds and 2 more
    • quick_buy_transactions: customer__country__region, customer__customer_country, customer__ds, metric_time, transaction__ds and 2 more
    • revenue_usd: customer__country__region, customer__customer_country, customer__ds, metric_time, transaction__ds and 2 more
    • transaction_amount: customer__country__region, customer__customer_country, customer__ds, metric_time, transaction__ds and 2 more
    • transaction_usd_na: customer__country__region, customer__customer_country, customer__ds, metric_time, transaction__ds and 2 more
    • transactions: customer__country__region, customer__customer_country, customer__ds, metric_time, transaction__ds and 2 more
   ```

6. Query your first metric:

   ```bash
   mf query --metrics transactions --group-by metric_time --order metric_time
   
   ✔ Success 🦄 - query completed after 0.03 seconds
    metric_time__day       transactions
    -------------------  --------------
    2022-03-07T00:00:00               2
    2022-03-08T00:00:00               2
    2022-03-09T00:00:00               1
    2022-03-10T00:00:00               2
    2022-03-11T00:00:00               1
    2022-03-12T00:00:00               1
    2022-03-13T00:00:00               1
    2022-03-14T00:00:00               1
    2022-03-15T00:00:00               3
    2022-03-16T00:00:00               2
    2022-03-17T00:00:00               1
    2022-03-18T00:00:00               1
    2022-03-21T00:00:00               3
    2022-03-22T00:00:00               3
    2022-03-23T00:00:00               2
    2022-03-25T00:00:00               3
    2022-03-26T00:00:00               2
    2022-03-27T00:00:00               2
    2022-03-28T00:00:00               1
    2022-03-29T00:00:00               2
    2022-03-30T00:00:00               4
    2022-03-31T00:00:00               5
    2022-04-01T00:00:00               1
    2022-04-02T00:00:00               2
    2022-04-03T00:00:00               1
    2022-04-04T00:00:00               1
   ```

7. Show the SQL MetricFlow generates:

   ```bash
   mf query --metrics transactions --group-by metric_time --order metric_time --explain
   ```

   ```sql
   ✔ Success 🦄 - query completed after 0.02 seconds
    🔎 SQL (remove --explain to see data or add --show-dataflow-plan to see the generated dataflow plan):

    SELECT
    metric_time__day
    , SUM(transactions) AS transactions
    FROM (
    SELECT
        DATE_TRUNC('day', ds) AS metric_time__day
        , 1 AS transactions
    FROM "mf_tutorial"."main"."transactions" transactions_src_10000
    ) subq_2
    GROUP BY
    metric_time__day
    ORDER BY metric_time__day
    '''

8. Visualize the plan (if you have graphviz installed - see README):

   ```bash
   mf query --metrics transactions --group-by metric_time --order metric_time --explain --display-plans
   ```

9. Add another dimension:

   ```bash
   mf query --metrics transactions --group-by metric_time,customer__customer_country --order metric_time
   ```

10. Add a coarser time granularity:

    ```bash
    mf query --metrics transactions --group-by metric_time__week --order metric_time__week
    ✔ Success 🦄 - query completed after 0.04 seconds
    metric_time__day     customer__customer_country      transactions
    -------------------  ----------------------------  --------------
    2022-03-07T00:00:00  FR                                         1
    2022-03-07T00:00:00  CA                                         1
    2022-03-08T00:00:00  CA                                         1
    2022-03-08T00:00:00  MX                                         1
    2022-03-09T00:00:00  CA                                         1
    2022-03-10T00:00:00  CA                                         1
    2022-03-10T00:00:00  US                                         1
    2022-03-11T00:00:00  US                                         1
    2022-03-12T00:00:00  US                                         1
    2022-03-13T00:00:00  US                                         1
    2022-03-14T00:00:00  US                                         1
    2022-03-15T00:00:00  US                                         2
    2022-03-15T00:00:00  FR                                         1
    2022-03-16T00:00:00  CA                                         1
    2022-03-16T00:00:00  FR                                         1
    2022-03-17T00:00:00  CA                                         1
    2022-03-18T00:00:00  MX                                         1
    2022-03-21T00:00:00  CA                                         1
    2022-03-21T00:00:00  FR                                         1
    2022-03-21T00:00:00  US                                         1
    2022-03-22T00:00:00  CA                                         1
    2022-03-22T00:00:00  MX                                         1
    2022-03-22T00:00:00  FR                                         1
    2022-03-23T00:00:00  CA                                         1
    2022-03-23T00:00:00  FR                                         1
    2022-03-25T00:00:00  CA                                         1
    2022-03-25T00:00:00  FR                                         2
    2022-03-26T00:00:00  US                                         1
    2022-03-26T00:00:00  FR                                         1
    2022-03-27T00:00:00  US                                         2
    2022-03-28T00:00:00  CA                                         1
    2022-03-29T00:00:00  FR                                         2
    2022-03-30T00:00:00  US                                         1
    2022-03-30T00:00:00  CA                                         1
    2022-03-30T00:00:00  MX                                         2
    2022-03-31T00:00:00  US                                         1
    2022-03-31T00:00:00  MX                                         2
    2022-03-31T00:00:00  CA                                         2
    2022-04-01T00:00:00  US                                         1
    2022-04-02T00:00:00  CA                                         1
    2022-04-02T00:00:00  MX                                         1
    2022-04-03T00:00:00  MX                                         1
    2022-04-04T00:00:00  MX                                         1
    ```

11. Try a more complicated query:

    ```bash
    mf query \
      --metrics transactions,transaction_usd_na \
      --group-by metric_time,transaction__is_large \
      --order metric_time \
      --start-time 2022-03-20 --end-time 2022-04-01
    
    ✔ Success 🦄 - query completed after 0.08 seconds
    metric_time__day     transaction__is_large      transactions    transaction_usd_na
    -------------------  -----------------------  --------------  --------------------
    2022-03-21T00:00:00  True                                  3                639.87
    2022-03-22T00:00:00  True                                  3                522.68
    2022-03-23T00:00:00  True                                  2                215.14
    2022-03-25T00:00:00  True                                  3                 66.6
    2022-03-26T00:00:00  True                                  2                179.91
    2022-03-27T00:00:00  True                                  2                316.98
    2022-03-28T00:00:00  True                                  1                266.9
    2022-03-29T00:00:00  True                                  2
    2022-03-30T00:00:00  True                                  4                534.25
    2022-03-31T00:00:00  True                                  5               1170.96
    2022-04-01T00:00:00  True                                  1                100.4
    ```

12. Before integrating metrics into your project, read up on [adding a time spine](https://docs.getdbt.com/docs/build/metricflow-time-spine?version=1.10).

If you found MetricFlow to be helpful, consider adding a [Github star](https://github.com/dbt-labs/metricflow) to promote the project.

## 2. dbt exposure

`dbt exposure` is a feature that allows you to document and track downstream use cases of your dbt project, such as dashboards, ML models, or applications that depend on your dbt models. It helps maintain visibility into how your data is being used across the organization.

**File Location:** `models/exposures.yml`

```
exposures.yml
├── exposures: (root key)
    ├── exposure 1: executive_dashboard
    │   ├── name: "executive_dashboard"
    │   ├── description: "Executive dashboard showing key business metrics and KPIs"
    │   ├── type: "dashboard"
    │   ├── url: "https://company.com/dashboards/executive - demo url"
    │   ├── maturity: "high"
    │   ├── owner:
    │   │   ├── name: "Data Team"
    │   │   └── email: "data-team@company.com"
    │   ├── depends_on:
    │   │   ├── ref('transactions')
    │   │   ├── ref('customers')
    │   │   └── ref('countries')
    │   └── tags: ['executive', 'dashboard', 'kpi']
    │
    ├── exposure 2: transaction_analytics_report
    │   ├── name: "transaction_analytics_report"
    │   ├── description: "Detailed transaction analytics report for business analysts"
    │   ├── type: "notebook"
    │   ├── url: "https://company.com/notebooks/transaction-analytics"
    │   ├── maturity: "medium"
    │   ├── owner:
    │   │   ├── name: "Analytics Team"
    │   │   └── email: "analytics@company.com"
    │   ├── depends_on:
    │   │   ├── ref('transactions')
    │   │   └── ref('customers')
    │   └── tags: ['analytics', 'transactions', 'report']
    │
    └── ... (6 more exposures with similar structure)
```

### **Core Properties:**

1. **`name`** - Unique identifier for the exposure
2. **`description`** - Human-readable description of what the exposure does
3. **`type`** - Category of downstream tool:
   * `dashboard` - BI dashboards (Tableau, Looker, etc.)
   * `application` - Web apps or APIs
   * `notebook` - Jupyter notebooks or analysis tools
   * `report` - Static reports or documents

4. **`url`** - Link to the actual downstream tool/report
5. **`maturity`** - Development stage:
   * `high` - Production-ready, stable
   * `medium` - In development, some stability
   * `low` - Experimental, early stage

### **Relationship Properties:**

6. **`owner`** - Responsible team/person:
   * `name` - Team or person name
   * `email` - Contact information

7. **`depends_on`** - Which dbt models this exposure uses:
   * `ref('model_name')` - References to your dbt models
   * Creates data lineage tracking

8. **`tags`** - Categorization for filtering/organization

Dbt exposures help you:

* **Track downstream usage** of your data models
* **Maintain data lineage** from source to consumption
* **Identify impact** when models change
* **Document ownership** and responsibility
* **Organize by maturity** and team

This creates a complete picture of how your dbt models are consumed across the organization!

### **Generating and Viewing Documentation**

To generate and view dbt documentation locally:

1. **Compile your project:**

   ```bash
   dbt compile
   ```

   This creates the compiled SQL and manifest files.

2. **Generate the documentation:**

   ```bash
   dbt docs generate
   ```

   This creates the catalog and website artifacts.

3. **Start the docs server:**

   ```bash
   dbt docs serve --port 8081 
   ```

   This launches the documentation site at <http://localhost:8081>

The documentation includes your exposures, models, sources, and their relationships!
