---
title: Ads System for a Website
tags: [hld, ads, click-tracking, streaming]
status: draft
---

# Ads System for a Website

Reference: [Hello Interview — Ad Click Aggregator](https://www.hellointerview.com/learn/system-design/answer-keys/ad-click-aggregator).

## Requirements

**Functional**

- Users can click on the ad and be redirected to the advertiser's website.
- Advertisers should be able to query ad click metrics over time with a minimum granularity of one minute.

**Non-functional**

- Scalable enough to support **10k clicks per sec**.
- Low latency for metrics queries.
- We shouldn't lose click data — **data durability**. Click data is money.
- **Idempotency** in click tracking: we shouldn't count the same click multiple times.
- Advertisers should be able to query metrics data soon after the click.

## How do we redirect the user?

| Approach | Notes |
|---|---|
| Client-side redirect | We might miss the click. Clients can extract the direct advertiser URL (e.g. with an extension) and never call us. |
| **Server-side redirect** | Preferred. |

**Server-side redirect**

1. Clients call us.
2. We log the click.
3. We return a response with **302 (redirect)** status code.

## How do we aggregate the data?

1. **Same database (Cassandra) for storing clicks and serving metrics.**
2. **Cassandra for writes.** A Spark job runs periodically and pushes metrics data into an OLAP store.
3. **Kafka for writes.** Stream processing with Flink aggregates metrics and pushes them to the OLAP store for querying.

## How do we solve hot shards?

A popular Nike shoe ad can heavily impact some shards. In those cases create multiple partition keys for the same logical key:

```
partitionKey:1-N
```

## How do we ensure we don't lose any data?

- Kafka retention.
- Flink checkpointing in S3.

## How is reconciliation done?

Data from Kafka can be dumped into S3. Run a Spark job on top of it to aggregate metrics and compare with the OLAP store to see anomalies.

## Added: deep dives

> [!info] Added
> The sections below are not in the original doc.

- Prefer **Kafka → Flink → OLAP** when you need ~10k QPS and near-real-time advertiser queries; batch Spark (option 2) is simpler but lags by the job interval.
- Deduplicate with a click id (or hash of user/ad/time bucket) so retries don't inflate counts.
- Salted partition keys (`ad_id:1..N`) must be reduced back together at aggregate / query time.
- If the interviewer broadens scope beyond click aggregation: impression serving, auction/bidding, campaign budget pacing and creative CDN are separate services from this pipeline.
