# Analytic Views Benchmark Results

## About

These results compare agent configurations on the [100-question Analytic Views
benchmark dataset](../data/questions/tasks_unit_test.jsonl). We have confirmed that the accuracy
is very similar with the complete dataset. The benchmark measures whether 
each agent can produce executable Oracle Analytic View SQL and match the expected answer rows.

> **Snapshot note:** The most recent agent results in this document were published at commit [`efd34ba`](https://github.com/yassirsalmi/nl2analytics-view-sql-benchmark/commit/efd34bad62e55b8ec67509794cf991726e522609). These values are completed historical runs against the benchmark as it existed at that time. Golden SQL queries, Analytic View definitions, and benchmark questions have changed since then, so the values should not be interpreted as current or partially updated results. Refreshed agent runs are a work in progress.

The `AV skill and tools` setup is the exported [agent spec version](analytic_view_agent_spec.json). 
It includes Analytic View-specific instructions plus the AV tools exposed by the packaged agent.

The `vanilla` setup is a [very basic NL2SQL agent](vanilla_agent_spec.json). It has only the
`get_schema_overview`, `get_table_schemas`, and `run_sql` tools:
no AV skill and no AV-specialized tools.

The checked-in Agent Spec artifacts used for the published runs are versioned as
follows:

| Agent Spec artifact | Component plugin version | Agent Spec version |
| ------------------- | ------------------------ | ------------------ |
| [`analytic_view_agent_spec.json`](analytic_view_agent_spec.json) | `26.1.2` | `25.4.1` |
| [`vanilla_agent_spec.json`](vanilla_agent_spec.json) | `26.1.2` | `25.4.1` |

Model names ending in `-medium` or `-low` refer to the reasoning level used for
that run. For example, `ocigenai.gpt-5.5-medium` is the model configured with
medium reasoning, while `ocigenai.gpt-5.5-low` is the same model family
configured with low reasoning. Compare rows by the full model name, because the
reasoning level is part of the model configuration and affects both quality and
runtime behavior.

## Evaluator Methodology

All rows were evaluated with the same standalone evaluator model:
`ocigenai.gpt-5.5-low`.

For each question, the benchmark executes both the golden SQL and the generated
SQL, then passes the generated answer, extracted generated SQL, generated result
set, golden SQL, and golden result set to the evaluator. The evaluator is asked
to judge whether the generated SQL answers the same question at the same grain,
using the executed result sets as primary evidence. Answers that do not use analytic views are marked incorrect. The `# Correct` column is generated from the evaluator's `Correct: true|false`
judgment.

## Summary

`# Correct` is the number of correctly answered questions out of 100. All other result columns are averages per question. 
The exported Agent Spec agents were kept as simple as possible, so their token counts and tool-call totals may differ from the results reported below.
These results were collected with two benchmark runtime setups:

- The first setup included an agent thought tool, counted its calls in the total tool-call count, and exposed additional run sql parameters such as a query-explanation parameter.
- The second setup did not include an agent thought tool and did not expose the additional run sql parameters.

## Analytic views with agent thought tool and with additional run sql parameters

| Model                        | Setup                         | # Correct | Exec Time (s) | Input Tokens | Output Tokens | Tool Calls | Run SQL tool calls | SQL Errors | SQL Error % |
| ---------------------------- | ----------------------------- | --------: | ------------: | -----------: | ------------: | ---------: | --------: | ---------: | ----------: |
| `ocigenai.gpt-5.6-sol-medium` | vanilla                      |      53.0 |        52.899 |        13771 |          2496 |        7.5 |       5.2 |       0.09 |         0.9 |
| `ocigenai.gpt-5.6-sol-medium` | AV skill and tools           |      85.0 |        46.943 |        30322 |          2126 |        6.3 |       3.0 |       0.03 |         0.6 |
| `ocigenai.gpt-5.6-sol-low`    | vanilla                      |      44.0 |        40.286 |        11109 |          1939 |        6.2 |       4.0 |       0.02 |         0.2 |
| `ocigenai.gpt-5.6-sol-low`    | AV skill and tools           |      88.0 |        41.722 |        29050 |          1742 |        6.1 |       2.8 |       0.06 |         1.9 |
| `ocigenai.gpt-5.5-medium`    | vanilla                       |      68.0 |       150.740 |       112326 |          4855 |       27.3 |      19.6 |       5.15 |        25.7 |
| `ocigenai.gpt-5.5-medium`    | AV skill and tools            |      89.0 |        78.251 |        34173 |          2188 |        6.7 |       2.1 |       0.00 |         0.0 |
| `ocigenai.gpt-5.5-low`       | vanilla                       |      48.0 |       139.019 |        68657 |          2946 |       19.4 |      14.8 |       4.52 |        29.3 |
| `ocigenai.gpt-5.5-low`       | AV skill and tools            |      84.0 |        64.646 |        24661 |          1459 |        5.4 |       1.3 |       0.02 |         0.8 |
| `ocigenai.gpt-5.4-mini-medium` | vanilla                     |       4.0 |       510.565 |        44961 |          4129 |       12.3 |       8.8 |       1.87 |        22.4 |
| `ocigenai.gpt-5.4-mini-medium` | AV skill and tools          |      64.0 |        56.051 |        30474 |          3257 |        6.7 |       2.4 |       0.46 |        16.6 |
| `ocigenai.gpt-5.4-nano-medium` | vanilla                     |       1.0 |       555.584 |       102261 |          5117 |       22.6 |      10.4 |       2.27 |        18.6 |
| `ocigenai.gpt-5.4-nano-medium` | AV skill and tools          |      41.0 |       114.964 |        99991 |          6015 |       15.6 |       4.2 |       1.29 |        28.0 |
| `ocigenai.gpt-oss-120b`      | vanilla                       |       0.0 |       233.253 |        41120 |          1029 |       16.2 |       5.0 |       0.68 |        10.8 |
| `ocigenai.gpt-oss-120b`      | AV skill and tools            |      14.0 |        47.626 |        40929 |          1009 |        4.6 |       2.5 |       0.50 |         8.2 |
| `ocigenai.gemini-2.5-pro`    | vanilla                       |       1.0 |        87.648 |        11660 |           885 |        4.8 |       2.2 |       1.25 |        45.0 |
| `ocigenai.gemini-2.5-pro`    | AV skill and tools            |      46.0 |        48.883 |        19820 |          1026 |        3.4 |       1.2 |       0.47 |        25.4 |
| `ocigenai.command-a-03-2025` | vanilla                       |       0.0 |       125.033 |        25612 |          1244 |        5.9 |       4.8 |       0.42 |        11.6 |
| `ocigenai.command-a-03-2025` | AV skill and tools            |       2.0 |       133.730 |        25723 |          1117 |        4.5 |       3.3 |       0.22 |         5.5 |

### Vanilla

| Model                        | # Correct | Exec Time (s) | Input Tokens | Output Tokens | Tool Calls | Run SQL tool calls | SQL Errors | SQL Error % |
| ---------------------------- | --------: | ------------: | -----------: | ------------: | ---------: | --------: | ---------: | ----------: |
| `ocigenai.gpt-5.6-sol-medium` |     53.0 |        52.899 |        13771 |          2496 |        7.5 |       5.2 |       0.09 |         0.9 |
| `ocigenai.gpt-5.6-sol-low`    |     44.0 |        40.286 |        11109 |          1939 |        6.2 |       4.0 |       0.02 |         0.2 |
| `ocigenai.gpt-5.5-medium`    |      68.0 |       150.740 |       112326 |          4855 |       27.3 |      19.6 |       5.15 |        25.7 |
| `ocigenai.gpt-5.5-low`       |      48.0 |       139.019 |        68657 |          2946 |       19.4 |      14.8 |       4.52 |        29.3 |
| `ocigenai.gpt-5.4-mini-medium` |     4.0 |       510.565 |        44961 |          4129 |       12.3 |       8.8 |       1.87 |        22.4 |
| `ocigenai.gpt-5.4-nano-medium` |     1.0 |       555.584 |       102261 |          5117 |       22.6 |      10.4 |       2.27 |        18.6 |
| `ocigenai.gpt-oss-120b`      |       0.0 |       233.253 |        41120 |          1029 |       16.2 |       5.0 |       0.68 |        10.8 |
| `ocigenai.gemini-2.5-pro`    |       1.0 |        87.648 |        11660 |           885 |        4.8 |       2.2 |       1.25 |        45.0 |
| `ocigenai.command-a-03-2025` |       0.0 |       125.033 |        25612 |          1244 |        5.9 |       4.8 |       0.42 |        11.6 |

### AV skill and tools

| Model                        | # Correct | Exec Time (s) | Input Tokens | Output Tokens | Tool Calls | Run SQL tool calls | SQL Errors | SQL Error % |
| ---------------------------- | --------: | ------------: | -----------: | ------------: | ---------: | --------: | ---------: | ----------: |
| `ocigenai.gpt-5.6-sol-medium` |     85.0 |        46.943 |        30322 |          2126 |        6.3 |       3.0 |       0.03 |         0.6 |
| `ocigenai.gpt-5.6-sol-low`    |     88.0 |        41.722 |        29050 |          1742 |        6.1 |       2.8 |       0.06 |         1.9 |
| `ocigenai.gpt-5.5-medium`    |      89.0 |        78.251 |        34173 |          2188 |        6.7 |       2.1 |       0.00 |         0.0 |
| `ocigenai.gpt-5.5-low`       |      84.0 |        64.646 |        24661 |          1459 |        5.4 |       1.3 |       0.02 |         0.8 |
| `ocigenai.gpt-5.4-mini-medium` |    64.0 |        56.051 |        30474 |          3257 |        6.7 |       2.4 |       0.46 |        16.6 |
| `ocigenai.gpt-5.4-nano-medium` |    41.0 |       114.964 |        99991 |          6015 |       15.6 |       4.2 |       1.29 |        28.0 |
| `ocigenai.gpt-oss-120b`      |      14.0 |        47.626 |        40929 |          1009 |        4.6 |       2.5 |       0.50 |         8.2 |
| `ocigenai.gemini-2.5-pro`    |      46.0 |        48.883 |        19820 |          1026 |        3.4 |       1.2 |       0.47 |        25.4 |
| `ocigenai.command-a-03-2025` |       2.0 |       133.730 |        25723 |          1117 |        4.5 |       3.3 |       0.22 |         5.5 |

## Analytic views without agent thought and without additional run sql parameters

| Model                        | Setup                         | # Correct | Exec Time (s) | Input Tokens | Output Tokens | Tool Calls | Run SQL tool calls | SQL Errors | SQL Error % |
| ---------------------------- | ----------------------------- | --------: | ------------: | -----------: | ------------: | ---------: | -----------------: | ---------: | -----------: |
| `ocigenai.gpt-5.5-medium`    | AV skill and tools            |      84.0 |       117.012063 |        33334 |          1943 |        7.6 |               1.8 |       0.01 |         0.3 |
| `ocigenai.gpt-5.5-medium`    | vanilla                       |      48.0 |        57.273826 |        14124 |          2128 |        8.4 |               4.0 |       0.01 |         0.3 |
| `ocigenai.gpt-5.5-low`       | AV skill and tools            |      82.0 |       101.439097 |        26498 |          1318 |        6.5 |               1.4 |       0.09 |         1.3 |
| `ocigenai.gpt-5.5-low`       | vanilla                       |      48.0 |        77.376164 |         9104 |          1339 |        6.3 |               2.5 |       0.03 |         1.0 |
| `ocigenai.gpt-5.4-mini-medium` | AV skill and tools          |      64.0 |        82.797925 |        45567 |          3188 |       10.6 |               2.5 |       0.36 |        11.4 |
| `ocigenai.gpt-5.4-mini-medium` | vanilla                     |      39.0 |        48.150953 |        17790 |          2503 |        8.7 |               3.6 |       0.03 |         0.6 |
| `ocigenai.gpt-5.4-nano-medium` | AV skill and tools         |      31.0 |       261.096978 |       141521 |          7442 |       20.9 |               7.8 |       2.74 |        31.4 |
| `ocigenai.gpt-5.4-nano-medium` | vanilla                    |      14.0 |        83.160137 |        47195 |          3985 |       13.8 |               6.3 |       0.53 |         4.9 |
| `ocigenai.gpt-oss-120b`      | AV skill and tools            |       5.0 |        50.104383 |        42190 |          1086 |        6.7 |               3.5 |       0.82 |        11.6 |
| `ocigenai.gpt-oss-120b`      | vanilla                       |       7.0 |        22.814289 |        13378 |           870 |        6.2 |               4.2 |       0.21 |         1.9 |
| `ocigenai.command-a-03-2025` | AV skill and tools            |       0.0 |        90.215933 |        20266 |           801 |        4.0 |               1.9 |       0.18 |         5.3 |
| `ocigenai.command-a-03-2025` | vanilla                       |       9.0 |       101.560786 |        13561 |          1083 |        6.1 |               3.9 |       0.06 |         1.6 |
| `ocigenai.gemini-2.5-pro`    | AV skill and tools            |       0.0 |         0.000000 |            0 |             0 |        0.0 |               0.0 |       0.00 |         0.0 |
| `ocigenai.gemini-2.5-pro`    | vanilla                       |       0.0 |         0.000000 |            0 |             0 |        0.0 |               0.0 |       0.00 |         0.0 |

### Vanilla

| Model                        | # Correct | Exec Time (s) | Input Tokens | Output Tokens | Tool Calls | Run SQL tool calls | SQL Errors | SQL Error % |
| ---------------------------- | --------: | ------------: | -----------: | ------------: | ---------: | -----------------: | ---------: | -----------: |
| `ocigenai.gpt-5.5-medium`    |      48.0 |        57.273826 |        14124 |          2128 |        8.4 |               4.0 |       0.01 |         0.3 |
| `ocigenai.gpt-5.5-low`       |      48.0 |        77.376164 |         9104 |          1339 |        6.3 |               2.5 |       0.03 |         1.0 |
| `ocigenai.gpt-5.4-mini-medium` |      39.0 |        48.150953 |        17790 |          2503 |        8.7 |               3.6 |       0.03 |         0.6 |
| `ocigenai.gpt-5.4-nano-medium` |      14.0 |        83.160137 |        47195 |          3985 |       13.8 |               6.3 |       0.53 |         4.9 |
| `ocigenai.gpt-oss-120b`      |       7.0 |        22.814289 |        13378 |           870 |        6.2 |               4.2 |       0.21 |         1.9 |
| `ocigenai.command-a-03-2025` |       9.0 |       101.560786 |        13561 |          1083 |        6.1 |               3.9 |       0.06 |         1.6 |
| `ocigenai.gemini-2.5-pro`    |       0.0 |         0.000000 |            0 |             0 |        0.0 |               0.0 |       0.00 |         0.0 |

### AV skill and tools

| Model                        | # Correct | Exec Time (s) | Input Tokens | Output Tokens | Tool Calls | Run SQL tool calls | SQL Errors | SQL Error % |
| ---------------------------- | --------: | ------------: | ----------- | ------------: | ---------: | -----------------: | ---------: | -----------: |
| `ocigenai.gpt-5.5-medium`    |      84.0 |       117.012063 |        33334 |          1943 |        7.6 |               1.8 |       0.01 |         0.3 |
| `ocigenai.gpt-5.5-low`       |      82.0 |       101.439097 |        26498 |          1318 |        6.5 |               1.4 |       0.09 |         1.3 |
| `ocigenai.gpt-5.4-mini-medium` |      64.0 |        82.797925 |        45567 |          3188 |        6.7 |               2.5 |       0.36 |        11.4 |
| `ocigenai.gpt-5.4-nano-medium` |      31.0 |       261.096978 |       141521 |          7442 |       20.9 |               7.8 |       2.74 |        31.4 |
| `ocigenai.gpt-oss-120b`      |       5.0 |        50.104383 |        42190 |          1086 |        6.7 |               3.5 |       0.82 |        11.6 |
| `ocigenai.command-a-03-2025` |       0.0 |        90.215933 |        20266 |           801 |        4.0 |               1.9 |       0.18 |         5.3 |
| `ocigenai.gemini-2.5-pro`    |       0.0 |         0.000000 |            0 |             0 |        0.0 |               0.0 |       0.00 |         0.0 |
