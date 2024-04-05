---
title: Metrics Validation
summary: Metric validation
tags:
  - metrics
---

## Overview

As part of our ongoing effort towards data quality and verification, several anomalies have been detected over time. While we will never get perfection, we want to be able to identify when things happen. The server can try to cleanse some of the data, but that masks any potential underlying issue. There's a few things to keep in mind:

- Off By One Errors - These are minimally influential in the grand scheme of things. If there's an extra minute, or extra step, or extra connection reported infrequently, it won't really impact outliers, trends, or rankings. These are important to fix, but lower priority.
- Flipped Values - These are slightly more important, as they could have serious ramifications depending on context. However, in the one case we see these (high vs low temperature), there's no implication of the issue. Nothing is reported to the user, that doesn't impact trends, and doesn't factor into outliers.
- Significant over/under reporting - These are the most important. This includes substantial overages (for example, 100 minutes in an hour) or obviously incorrect data (4,000,000 steps in an hour).

One goal is to enable better reporting and identification of these situations beyond what the server is able to see. While it would be possible to see who sent the faulty metrics using purely server headers, that won't reveal if the issue occurs as a result of faulty BLE communication starting on the collar; fauly processing of the BLE data; faulty storage or consolidation on the app; or faulty processing prior to sending the data. Therefore, at each appropriate step, the data should be analyzed. If there is a significant data or performance implication in this, please address that.

**STATUS**: Pending discussion and implementation.

### Event Reporting

A new event with a domain of `metric_anomaly` will be created and used to report it. It is general so that the same event can be used with different `additionalData`. The `additionalData` will consist of the following fields:

- `anomalies`: An array of issues found with the specific metric
- `source`: The source, one of:
  - `android`
  - `ios`
  - `firmware`
- `process`: When in the lifecycle the anomaly was found, such as:
  - `native`: For `firmware`, created natively during data collection
  - `ble_send`: For `firmware`, noticed when sending over BLE
  - `http_send`: For all, noticed when sending to the server
  - `ble_received`: For `mobile`, noticed when received from the firmware over BLE
  - `ble_processed`: For `mobile`, noticed when processing or storing the metrics
- `fixed`: If the metric was fixed as a result of this notice. I would imagine values could be `yes`, `no`, and `na`.

For example, the call may look like this:

```json
{
  "domain": "metric_anomaly",
  "additionalData": {
    "source": "android",
    "process": "ble_received",
    "deviceId": 1,
    "anomalies": [
      {
        "metrics": ["steps"],
        "issue": "recieved 4,000,000 steps over BLE from collar",
        "fixed": "yes"
      }
    ]
  }
}
```

```json
{
  "domain": "metric_anomaly",
  "additionalData": {
    "source": "ios",
    "process": "ble_processed",
    "deviceId": 1,
    "anomalies": [
      {
        "metrics": ["wifiMinutes", "cellMinutes"],
        "issue": "wifi minutes of 45 and cell minutes of 16 > 60 minutes",
        "fixed": "no"
      },
      {
        "metrics": ["exerciseMinutes"],
        "issue": "exercise minutes were 185",
        "fixed": "yes"
      }
    ]
  }
}
```

*Note about fixing*: Future iterations may change the "fixing" thresholds. For example, we may want to only fix the minute metrics if the reported value is only a few minutes over 60, but leave it if it's egregious. For now, just following the flow should suffice.

**IMPORTANT**: No fields are required or checked for the server, so it's important to be consistent.

{{metrics_validation_algorithm}}
