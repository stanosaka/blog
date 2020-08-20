---
title: log
tags: log analytics
---
Logs versus Metrics
| Logs                         | Metrics                            |   |
| ----                         | :------                            |   |
| Less structured              | More structured                    |   |
| Verbose descriptions         | Quantitative data points           |   |
| Triggered by events          | Collected at regular time interval |   |
| Best for root-cause analysis | Best for direct numberic analysis  |   |

- Logs are semi-structured, defined according to the preferences of the individual developers that created them.
- Metrics are quantitative assessments of specific variables, typically gathered at specific time intervals, unlike logs, which are riggered by external events.

**Collect->ETL->Index->Store->Search->Correlate->Visualize->Analyze->Report**

Pipeline for assembling and driving value from log data

Common operations performed on log data include the following:
> Collect: From its dispersed sources, log data must be aggregated, parsed, and scrubbed, such as inserting defaults for missing values and discarding irrelevant entries.
> ETL (Extract, Transform, Load): Data preparation can include being cleaned of bad entries, reformattted, normalized, and enriched with elements of other datasets.
> Index: To accelerate queries, the value of indexing all or a portion of the log data must be balanced against the compute overhead required to do so.
> Store: Potentially massive sets of log data must be stored efficiently, using infrastructure built to deliver performance that scales out smoothly and cost effectively.
> Search: The large scale of the log data in a typcial implementation places extreme demands on the ability to perform flexible, fast, sophisticated queries against it.
> Correlate: The relationships among various data sources must be identified and correlated before the significance of the underlying data points can be uncovered.
> Visualize: Treating log entr
