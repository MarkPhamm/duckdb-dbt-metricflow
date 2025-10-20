# duckdb-dbt-metricflow

Testing metricflow on duckdb - dbt

🤓 Please run the following steps:

1. Switch to the root directory of the generated sample project (e.g. `cd mf_tutorial_project`).
   This enables use of the tutorial project and associated connection profile in later steps.
   Run `cd database && pwd` to get the full path to your database directory.
   Update the path in `profiles-example.yml` to point to your DuckDB database location.

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
