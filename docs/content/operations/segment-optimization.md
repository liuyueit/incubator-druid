---
layout: doc_page
---

# Segment size optimization

In Druid, it's important to optimize the segment size because

  1. Druid stores data in segments. If you're using the [best-effort roll-up](../design/index.html#roll-up-modes) mode,
  increasing the segment size might introduce further aggregation which reduces the dataSource size.
  2. When a query is submitted, that query is distributed to all historicals and realtimes
  which hold the input segments of the query. Each node has a processing threads pool and use one thread per segment to
  process it. If the segment size is too large, data might not be well distributed over the
  whole cluster, thereby decreasing the degree of parallelism. If the segment size is too small,
  each processing thread processes too small data. This might reduce the processing speed of other queries as well as
  the input query itself because the processing threads are shared for executing all queries.

It would be best if you can optimize the segment size at ingestion time, but sometimes it's not easy
especially for the streaming ingestion because the amount of data ingested might vary over time. In this case,
you can roughly set the segment size at ingestion time and optimize it later. You have two options:

  - Turning on the [automatic compaction of coordinators](../design/coordinator.html#compacting-segments).
  The coordinator periodically submits [compaction tasks](../ingestion/tasks.html#compaction-task) to re-index small segments.
  - Running periodic Hadoop batch ingestion jobs and using a `dataSource`
  inputSpec to read from the segments generated by the Kafka indexing tasks. This might be helpful if you want to compact a lot of segments in parallel.
  Details on how to do this can be found under ['Updating Existing Data'](../ingestion/update-existing-data.html).