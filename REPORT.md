# Hands-on L10: Report

**Name: Arham Hussain Inamdar**

**Student ID: 801404110**

**Email: ainamda3@charlotte.edu**

---

## Seed and commands

Seed used for `datagen.py`: 801404110

The commands you ran, in order. If you deviated from the steps in the README, say where and
why.

```bash
python3 datagen.py 801404110
docker compose up -d
docker cp datagen.py spark-master:/opt/spark/work-dir/shared/
docker exec spark-master python3 /opt/spark/work-dir/shared/datagen.py 801404110 \
  /opt/spark/work-dir/shared/stream
docker cp main.py spark-master:/opt/spark/work-dir/

docker exec -it spark-master /opt/spark/bin/spark-submit \
  --master spark://spark-master:7077 \
  /opt/spark/work-dir/main.py task1 /opt/spark/work-dir/shared

docker cp main.py spark-master:/opt/spark/work-dir/

docker exec -it spark-master /opt/spark/bin/spark-submit \
  --master spark://spark-master:7077 \
  /opt/spark/work-dir/main.py task1 /opt/spark/work-dir/shared

docker cp main.py spark-master:/opt/spark/work-dir/

docker exec -it spark-master /opt/spark/bin/spark-submit \
  --master spark://spark-master:7077 \
  /opt/spark/work-dir/main.py task1 /opt/spark/work-dir/shared

docker exec -it spark-master /opt/spark/bin/spark-submit \
  --master spark://spark-master:7077 \
  /opt/spark/work-dir/main.py task2 /opt/spark/work-dir/shared

docker exec -it spark-master /opt/spark/bin/spark-submit \
  --master spark://spark-master:7077 \
  /opt/spark/work-dir/main.py task2 /opt/spark/work-dir/shared update

docker compose down
```

---

## Results

### Task 1: ingest and parse

Paste the micro-batch summary printed at the end of the run, and the number of rows in
`shared-folder/output/task1` (`cat shared-folder/output/task1/*.csv | grep -vc trip_id`).


| Batch | Input rows | Watermark | State rows |
| ---: | ---: | --- | --- |
| 0 | 20 | - | - |
| 1 | 20 | - | - |
| 2 | 25 | - | - |
| 3 | 23 | - | - |
| 4 | 22 | - | - |
| 5 | 28 | - | - |
| 6 | 21 | - | - |
| 7 | 29 | - | - |
| 8 | 23 | - | - |
| 9 | 27 | - | - |
| 10 | 25 | - | - |
| 11 | 24 | - | - |

The CSV output had **287 rows**.


How many micro-batches ran, and why that number?

There were 12 micro-batches because there were 12 input files and maxFilesPerTrigger was set to 1. Each batch read one file. The query stopped after processing all available files. Task 1 is stateless, so there are no watermark or state values

### Task 2: trips and fares per window

Paste the windows written by the stream (`cat shared-folder/output/task2/*.csv | grep -v
window_start | sort`), the summary, and the batch table that `main.py` prints at the end.

**Append-mode CSV output**

| Window start | Window end | Trips | Total fare |
| --- | --- | ---: | ---: |
| 07:58:00 | 08:00:00 | 3 | 125.36 |
| 08:00:00 | 08:02:00 | 41 | 1753.14 |
| 08:02:00 | 08:04:00 | 52 | 2349.83 |
| 08:04:00 | 08:06:00 | 54 | 2162.26 |
| 08:06:00 | 08:08:00 | 45 | 2131.93 |

**Append-mode micro-batch summary**

| Batch | Input rows | Watermark | State rows |
| ---: | ---: | --- | ---: |
| 0 | 20 | - | 2 |
| 1 | 20 | 07:58:51 | 2 |
| 2 | 25 | 07:59:43 | 3 |
| 3 | 23 | 08:00:54 | 2 |
| 4 | 22 | 08:01:48 | 3 |
| 5 | 28 | 08:02:57 | 2 |
| 6 | 21 | 08:03:50 | 3 |
| 7 | 29 | 08:04:56 | 2 |
| 8 | 23 | 08:05:50 | 3 |
| 9 | 27 | 08:06:36 | 2 |
| 10 | 25 | 08:07:43 | 3 |
| 11 | 24 | 08:08:55 | 2 |
| 12 | 0 | 08:09:45 | 2 |

**Update-mode micro-batch summary**

| Batch | Input rows | Watermark | State rows |
| ---: | ---: | --- | ---: |
| 0 | 20 | - | 2 |
| 1 | 20 | 07:58:51 | 2 |
| 2 | 25 | 07:59:43 | 3 |
| 3 | 23 | 08:00:54 | 2 |
| 4 | 22 | 08:01:48 | 3 |
| 5 | 28 | 08:02:57 | 2 |
| 6 | 21 | 08:03:50 | 3 |
| 7 | 29 | 08:04:56 | 2 |
| 8 | 23 | 08:05:50 | 3 |
| 9 | 27 | 08:06:36 | 2 |
| 10 | 25 | 08:07:43 | 3 |
| 11 | 24 | 08:08:55 | 2 |
| 12 | 0 | 08:09:45 | 2 |

**Batch results over all files**

| Window start | Window end | Trips | Total fare |
| --- | --- | ---: | ---: |
| 07:58:00 | 08:00:00 | 4 | 179.09 |
| 08:00:00 | 08:02:00 | 45 | 1978.02 |
| 08:02:00 | 08:04:00 | 54 | 2445.74 |
| 08:04:00 | 08:06:00 | 54 | 2162.26 |
| 08:06:00 | 08:08:00 | 45 | 2131.93 |
| 08:08:00 | 08:10:00 | 45 | 1842.30 |
| 08:10:00 | 08:12:00 | 40 | 1837.50 |


---

## Questions

**1. Append versus update.** Run task 2 a second time with `update` (it prints to the
console). In which batch does each window first appear in update mode, and in which batch
is it written in append mode? Explain the difference with the watermark values from the
summary.

| Window | First update batch | Append batch | Watermark when appended |
| --- | ---: | --- | --- |
| 07:58–08:00 | 0 | 3 | 08:00:54 |
| 08:00–08:02 | 0 | 5 | 08:02:57 |
| 08:02–08:04 | 2 | 7 | 08:04:56 |
| 08:04–08:06 | 4 | 9 | 08:06:36 |
| 08:06–08:08 | 6 | 11 | 08:08:55 |
| 08:08–08:10 | 8 | Not written | - |
| 08:10–08:12 | 10 | Not written | - |

Update mode prints a window when its total changes. For example, the 08:00–08:02 window had 17 trips in batch 0 and reached 41 trips in batch 3.

Append mode waits until the watermark passes the window's end, then writes it once. The final watermark was 08:09:45, so the last two windows were not written.



**2. Where did the rows go?** Compare the stream's windows with the batch table. Which
windows have fewer trips in the stream, by how many, and why? Which windows are missing
from the stream's output altogether, and what would make Spark write them? (Look at
`datagen.py` for how the late events were made.)

| Window | Batch trips | Streaming trips | Trips dropped |
| --- | ---: | ---: | ---: |
| 07:58–08:00 | 4 | 3 | 1 |
| 08:00–08:02 | 45 | 41 | 4 |
| 08:02–08:04 | 54 | 52 | 2 |

The stream dropped 7 trips in total. These match the 7 events that the generator made 8–10 minutes late. Their windows had already passed the watermark when they arrived. The batch results include them because the batch calculation has no watermark. The 22 events that were 1–2 minutes late were kept in this run.

The 08:08–08:10 and 08:10–08:12 windows were missing from append output. Update mode showed 45 trips and 1842.30 for the first, and 40 trips and 1837.50 for the second.

Spark would write these windows if new events advanced the watermark past their end times. Just waiting would not move the event-time watermark.



**3. Checkpoint.** Run task 1 again without deleting `shared-folder/checkpoint/task1`. What
happens, and why? List the subfolders of the checkpoint and say in one line what each one
holds.

Running Task 1 again without deleting its checkpoint produced batch 12 with zero input rows. Spark already knew that all 12 files had been processed, so it did not process them again. The output stayed at 287 rows.

| Checkpoint folder | What it holds |
| --- | --- |
| `sources/` | Records of the input files selected for processing. |
| `offsets/` | Source offsets and metadata for each batch. |
| `commits/` | Records of batches that completed successfully. |
| `state/` | Saved window totals for Task 2. Task 1 does not have this folder because it is stateless. |


---

## Problems and fixes

Anything that went wrong and what resolved it. Paste the actual error message. If nothing
went wrong, say so.

No problems

## AI Use Statement

I used AI to help structure my report and extract information and enhance clarity. I also used AI to help explain code. All other ideas, thoughts, and work are mine