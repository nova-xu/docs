---
title: TiDB Lightning Log Explanation
summary: Provide detailed explanation of logs generating during the importing process using TiDB Lightning.
---

# TiDB Lightning Log Explanation

This doc investigated the **5.4 version** **TiDB** **Lightning** logs of a successfully test data importing with TiDB Lightning - **Local Backend mode**, and dive deep to understand where the log comes from and what it actually represents. Users of TiDB Lightning can refer to this doc to have a better understand of the logs. We will continue working on other TiDB versions and improving the log messages.

We expected you would have basic understanding of what Lightning is, and read through high level Lightning workflow doc in [previous section](https://docs.pingcap.com/tidb/v5.4/tidb-lightning-overview) (max ~10 mins reading time). You can also refer to [glossary](https://docs.pingcap.com/tidb/v5.4/tidb-lightning-glossary) when you encounter an unfamiliar concept later in this doc.

When receiving Lightning oncall tickets, users can quickly look through the log and refer to this doc to find out if there are any anomalies, eg: warn/error etc. Users could use this doc to quickly navigate within Lightning source code to identify potential root cause and what exactly the error means for.

Note that some trivial logs are ignored. Only important logs are included in the following doc.

## Lightning Log Explanation 

```
[WARN] [config.go:800] ["currently only per-task configuration can be applied, global configuration changes can only be made on startup"] ["global config changes"="[lightning.level,lightning.file]"]
```
[config.go:800](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/config/config.go#L800):
Warn that some legal field of config file won't be overwritten, such as lightning.file.
Triggered from [br/cmd/tidb-lightning/main.go](https://github.com/pingcap/tidb/blob/v5.4.0/br/cmd/tidb-lightning/main.go#L87) , it represents loading lightning toml file.  
  
```
[INFO] [info.go:49] ["Welcome to TiDB-Lightning"] [release-version=v5.4.0] [git-hash=55f3b24c1c9f506bd652ef1d162283541e428872] [git-branch=HEAD] [go-version=go1.16.6] [utc-build-time="2022-04-21 02:07:55"] [race-enabled=false]
```
[info.go:49](https://github.com/pingcap/tidb/blob/55f3b24c1c9f506bd652ef1d162283541e428872/br/pkg/version/build/info.go#L49):
Print current TiDB Lightning version infomation.

```
[INFO] [lightning.go:233] [cfg] [cfg="{\"id\":1650510440481957437,\"lightning\":{\"table-concurrency\":6,\"index-concurrency\":2,\"region-concurrency\":8,\"io-concurrency\":5,\"check-requirements\":true,\"meta-schema-name\":\"lightning_metadata\", ...
```
[lightning.go:233](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/lightning.go#L233):
Print TiDB Lightning config information.

```
[INFO] [lightning.go:312] ["load data source start"] 
```
[lightning.go:312](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/lightning.go#L312): [Load Lightning config](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/mydump/loader.go#L105) info of data source, [scan relevant data source file info](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/mydump/loader.go#L205), and store it into s.loader.dbs for later import process.

```
[INFO] [loader.go:289] ["[loader] file is filtered by file router"] [path=metadata]
```
[loader.go:289](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/mydump/loader.go#L289): This log indicates which file is [skipped by cfg.Mydumper.FileRouters and cfg.Mydumper.DefaultFileRules](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/mydump/loader.go#L134).

```
[INFO] [lightning.go:315] ["load data source completed"] [takeTime=273.964µs] []
``` 
[lightning.go:315](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/lightning.go#L315): The previous loading process has completed.

```
[INFO] [checkpoints.go:977] ["open checkpoint file failed, going to create a new one"] [path=/tmp/tidb_lightning_checkpoint.pb] [error="open /tmp/tidb_lightning_checkpoint.pb: no such file or directory"]
```
[checkpoints.go:977](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/checkpoints/checkpoints.go#L977):
If Lightning uses file to store checkpoint, and can't find any local checkpoint file, Lightning will create a new checkpoint.

```
[INFO] [local.go:401] ["multi ingest support"]
```
[local.go:401](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/local.go#L401):
Muti ingest mode is supported which allows Lightning to ingest multiple files in one request.

```
[INFO] [restore.go:444] ["the whole procedure start"]
```
[restore.go:444](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L444):
Start the importing process.

```
[INFO] [restore.go:748] ["restore all schema start"]
```
[restore.go:748](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L748): Based on data source schema, create the needed database and table in the target TiDB cluster.

```
[INFO] [restore.go:767] ["restore all schema completed"] [takeTime=189.766729ms]  
```
[restore.go:767](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L767): Restoring schemas has completed.

```
[INFO] [check_info.go:680] ["datafile to check"] [db=sysbench] [table=sbtest1] [path=sysbench.sbtest1.000000000.sql]
```
[check_info.go:680](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/check_info.go#L680):
Before starting to restore a table, Lighting uses the first data file of the table to check if the schema is matched between the source data and the target cluster.

```
[INFO] [version.go:360] ["detect server version"] [type=TiDB] [version=5.4.0]
```
[version.go:360](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/version/version.go#L360):
Detect and print the cuurent TiDB server version. To import data in local backend mode, TiDB with version higher than 4.0 is required.

We also need to check server version for [detecting data confilcts](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/version/version.go#L224).

```
[INFO] [check_info.go:995] ["sample file start"] [table=sbtest1]
```
[check_info.go:995](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/check_info.go#L995): As part of lightning precheck, we need to estimate source data size to determine: 1. [the local disk has enough space if Lighting is in local backend mode](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/check_info.go#L462); 2. [the target cluster has enough space to store transformed kv pairs](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/check_info.go#L102). This log represents that Lightning starts to sample first source data file of each table, and transform sample source data to kv paris to estimate the size of kv pairs based on file size vs. kv pairs size ratio.

```
[INFO] [check_info.go:1080] ["Sample source data"] [table=sbtest1] [IndexRatio=1.3037832180660969] [IsSourceOrder=true]  
```
[check_info.go:1080](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/check_info.go#L1080): Sampling source data has completed.

```
[INFO] [pd.go:415] ["pause scheduler successful at beginning"] [name="[balance-region-scheduler,balance-leader-scheduler,balance-hot-region-scheduler]"] 
[INFO] [pd.go:423] ["pause configs successful at beginning"] [cfg="{\"enable-location-replacement\":\"false\",\"leader-schedule-limit\":4,\"max-merge-region-keys\":0,\"max-merge-region-size\":0,\"max-pending-peer-count\":2147483647,\"max-snapshot-count\":40,\"region-schedule-limit\":40}"]
```

[pd.go:415](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/pdutil/pd.go#L415), [pd.go:423](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/pdutil/pd.go#L423): In local backend mode, some [pd schedulers](https://docs.pingcap.com/tidb/stable/tidb-scheduling) are disabled and some [config settings are changed](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/pdutil/pd.go#L417) to make splitting region and ingesting sst more stable.

```
[INFO] [restore.go:1683] ["switch to import mode"]
```
[restore.go:1683](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L1683):
In local backend mode, before restoring the table, Lightning turns each TiKV node into import mode to speed up import process, but sacrifice its storage space. Under tidb backend mode, Lightning does not need to switch TiKV to import mode.

```
[INFO] [restore.go:1462] ["restore table start"] [table=`sysbench`.`sbtest1`]
```
[restore.go:1462](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L1462):
Start to restore table `sysbench`.`sbtest1`.  Lightning concurrently restore multiple tables based on [IndexConcurrency](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L1459) config. For each table, Lightning concurrently restore data files in the table based on [RegionConcurrency](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/mydump/region.go#L157).

```
[INFO] [table_restore.go:91] ["load engines and files start"] [table=`sysbench`.`sbtest1`]  
```
[table_restore.go:91](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L91): This log indicates start of [single table restore](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L1561). To restore a specific table, Lightning [splits table data files](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/mydump/region.go#L136) into multiple [chunks/table regions](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/mydump/region.go#L283) based on [RegionConcurrency](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/mydump/region.go#L157) concurrency number(max value would be the max logical CPU cores number).
-   If the data source is a large csv, it will [divide the csv file into different chunks as table regions](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/mydump/region.go#L279).
    
-   If it's another data source, it will [treat the whole data source file as a table region](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/mydump/region.go#L283).

After creating regions, Lightning [assigns each data file](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/mydump/region.go#L246) with an Engine ID based on Mydumper.BatchSize config, so that different engine can process data files in parallel.

```
[INFO] [region.go:241] [makeTableRegions] [filesCount=8] [MaxRegionSize=268435456] [RegionsCount=8] [BatchSize=107374182400] [cost=53.207µs]
```
[region.go:241](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/mydump/region.go#L241): Lightning prints how many table data files(`filesCount`) have been processed, and the largest chunk size(`MaxRegionSize`) of CSV file, the number of generated table regions/chunks (`RegionsCount`), and the batchSize that we use to assign different engines to process data files.

```
[INFO] [table_restore.go:129] ["load engines and files completed"] [table=`sysbench`.`sbtest1`] [enginesCnt=2] [ime=75.563µs] []
```

[table_restore.go:129](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L129): Lightning prints the processed table info, and the number of chunks and engines after making regions and assigning engine IDs.

```
[INFO] [version.go:360] ["detect server version"] [type=TiDB] [version=5.4.0]
```

[version.go:360](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/version/version.go#L360): `restoreTable` [checks the server version](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L1578) to determine the following workflow.

```
[INFO] [backend.go:346] ["open engine"] [engineTag=`sysbench`.`sbtest1`:-1] [engineUUID=3942bab1-bd60-52e2-bf53-e17aebf962c6]
```

[backend.go:346](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/backend.go#L346): At the beginning of [restore engines process](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L199), index engine was opened first. engine id -1 represents the index engine.

```
[INFO] [table_restore.go:270] ["import whole table start"] [table=`sysbench`.`sbtest1`]
```

[table_restore.go:270](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L270): Lightning prints the log before concurrently [restore different data engines](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L318).

```
[INFO] [table_restore.go:317] ["restore engine start"] [table=`sysbench`.`sbtest1`] [engineNumber=0]
```

[table_restore.go:317](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L317): Start to restore engine 0, non -1 engine id means data engine. Note that "restore enigne" and "import enigne" (appears later in the logs) refer to different processes. "restore engine" indicates the process of sending KV pairs to the allocated engine and sorting them, while "import engine" represents the process of ingesting Sorted KV pairs in the engine file to the TiKV nodes.

```
[INFO] [table_restore.go:422] ["encode kv data and write start"] [table=`sysbench`.`sbtest1`] [engineNumber=0]
```

[table_restore.go:422](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L422): Lightning starts the process of [restoring table data by chunks](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L386).

```
[INFO] [backend.go:346] ["open engine"] [engineTag=`sysbench`.`sbtest1`:0] [engineUUID=d173bb2e-b753-5da9-b72e-13a49a46f5d7]
```

[backend.go:346](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/backend.go#L346): Open data enigne with engine id = 0, which is a data engine.

```
[INFO] [restore.go:2482] ["restore file start"] [table=`sysbench`.`sbtest1`] [engineNumber=0] [fileIndex=0] [path=sysbench.sbtest1.000000000.sql:0] 
```
[restore.go:2482](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L2482): 
Note that this log may appear multiple times based on the importing table data size. Each log in this form indicates the start of restoring a chunk/region. Lightning concurrently [restores chunks](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L386) based on [regionWorkers](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L532) defined by [RegionConcurrency](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L402). For each chunk, the restoring process is as following:
1. Lightning [encodes sql into kv pairs](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L2389);
 2. Lightning [writes kv pairs into data engine and index engine](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L2179) based on whether KVs are sorted：
	- If KVs are sorted, Lightning [writes into the SST file directly](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/engine.go#L1227).
    - If KVs are unsorted, Lightning first stores them in [writeBatch](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/engine.go#L994) then [batch flushes](https://github.com/pingcap/tidb/blob/55f3b24c1c9f506bd652ef1d162283541e428872/br/pkg/lightning/backend/local/engine.go#L1156) them into the SST file.

```
[INFO] [engine.go:777] ["write data to local DB"] [size=134256327] [kvs=621576] [files=1] [sstFileSize=108984502] [file=/home/centos/tidb-lightning-temp-data/sorted-kv-dir/d173bb2e-b753-5da9-b72e-13a49a46f5d7.sst/11e65bc1-04d0-4a39-9666-cae49cd013a9.sst] [firstKey=74800000000000003F5F728000000000144577] [lastKey=74800000000000003F5F7280000000001DC17E] 
```

[engine.go:777](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/engine.go#L777): The log indicates the start of ingesting generated SST file into the pebble engine. Lightning [concurrently ingests SST files](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/local.go#L624). [CompactConcurrency](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/engine.go#L526) controlls the concurrency level.

```
[INFO] [restore.go:2492] ["restore file completed"] [table=`sysbench`.`sbtest1`] [engineNumber=0] [fileIndex=1] [path=sysbench.sbtest1.000000001.sql:0] [readDur=3.123667511s] [encodeDur=5.627497136s] [deliverDur=6.653498837s] [checksum="{cksum=6610977918434119862,size=336040251,kvs=2646056}"] [takeTime=15.474211783s] []
```

[restore.go:2492](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L2492): The log indicates that a chunk has completed restoring. The file(fileIndex=1) of table(sysbench. sbtest1) has been restored.

```
[INFO] [table_restore.go:584] ["encode kv data and write completed"] [table=`sysbench`.`sbtest1`] [engineNumber=0] [read=16] [written=2539933993] [takeTime=23.598662501s] []
[source code]
```

[table_restore.go:584](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L584): The process of restoring table data by chunks has completed. engine `engineNumber=0` has already processed `table = sysbench.sbtest1`, and all kv pairs have been writen into pebble engine.

```
[INFO] [backend.go:438] ["engine close start"] [engineTag=`sysbench`.`sbtest1`:0] [engineUUID=d173bb2e-b753-5da9-b72e-13a49a46f5d7]   
[INFO] [backend.go:440] ["engine close completed"] [engineTag=`sysbench`.`sbtest1`:0] [engineUUID=d173bb2e-b753-5da9-b72e-13a49a46f5d7] [takeTime=2.879906ms] []
```

[backend.go:438](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/backend.go#L438): The logs shows [the final stage of engine restore](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L628). The data engine that processed the table data is closed and prepared for importing.

```
[INFO] [engine.go:1455] ["compact sst"] [fileCount=8] [size=380000000] [count=10000000] [cost=3.421250443s] [file=/home/centos/tidb-lightning-temp-data/sorted-kv-dir/3942bab1-bd60-52e2-bf53-e17aebf962c6.sst/c7a62230-378f-4558-a355-b1b82ed3af95.sst]
```

[engine.go:1455](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/engine.go#L1455): Whether compact is needed is determined by [estimated compaction threshold](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L246). If compact is needed, Lightning [merge multiple small SST](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/engine.go#L1353) files into a large SST file. The log shows which sst file has been processed.

```
[INFO] [table_restore.go:319] ["restore engine completed"] [table=`sysbench`.`sbtest1`] [engineNumber=0] [takeTime=27.031916498s] []
```

[table_restore.go:319](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L319): The process of writing KV pairs to the engine has completed. The corresponding engine for this process has id = 0.

```
[INFO] [table_restore.go:927] ["import and cleanup engine start"] [engineTag=`sysbench`.`sbtest1`:0] [engineUUID=d173bb2e-b753-5da9-b72e-13a49a46f5d7]   
[INFO] [backend.go:452] ["import start"] [engineTag=`sysbench`.`sbtest1`:0] [engineUUID=d173bb2e-b753-5da9-b72e-13a49a46f5d7] [retryCnt=0]
```

[table_restore.go:927](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L927), [backend.go:452](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/backend.go#L452): Based on [TikvImporter.RegionSplitSize](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L927) config, it starts to [import](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/local.go#L1311) kv pairs stored in the engine into the target TiKV node.

```
[INFO] [local.go:1023] ["split engine key ranges"] [engine=d173bb2e-b753-5da9-b72e-13a49a46f5d7] [totalSize=2159933993] [totalCount=10000000] [firstKey=74800000000000003F5F728000000000000001] [lastKey=74800000000000003F5F728000000000989680] [ranges=22]
```

[local.go:1023](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/local.go#L1023): During [ImportEngine](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/local.go#L1331) phase, it logically splits engine data into many ranges based on [TikvImporter.RegionSplitSize](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L927) config.

```
[INFO] [local.go:1336] ["start import engine"] [uuid=d173bb2e-b753-5da9-b72e-13a49a46f5d7] [ranges=22] [count=10000000] [size=2159933993]
```

[local.go:1336](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/local.go#L1336): Lightning starts to import KV pairs in the engine by splited ranges.

```
[INFO] [local.go:1343] ["import engine unfinished ranges"] [count=22]
```

[local.go:1343](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/local.go#L1343): Unfinished ranges of engine has imported. Unfinished ranges are retrieved by:
1. Lightning [sorts the ranges and merges ranges]((https://github.com/pingcap/tidb/blob/55f3b24c1c9f506bd652ef1d162283541e428872/br/pkg/lightning/backend/local/engine.go#L890)) that overlaps with each other into a single range.
2. Then Lightning [filters out the overlaping ranges](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/engine.go#L917) and return the unfinished range to import later.

```
[INFO] [localhelper.go:89] ["split and scatter region"] [minKey=7480000000000000FF3F5F728000000000FF0000010000000000FA] [maxKey=7480000000000000FF3F5F728000000000FF9896810000000000FA] [retry=0]
```

[localhelper.go:89](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/localhelper.go#L89): Lightning starts to [split and scatter](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/localhelper.go#L65) previous unfinished ranges. minKey and maxKey refer to the min and max key of unfinished ranges.

```
[INFO] [localhelper.go:108] ["paginate scan regions"] [count=1] [start=7480000000000000FF3F5F728000000000FF0000010000000000FA] [end=7480000000000000FF3F5F728000000000FF9896810000000000FA]   
[INFO] [localhelper.go:116] ["paginate scan region finished"] [minKey=7480000000000000FF3F5F728000000000FF0000010000000000FA] [maxKey=7480000000000000FF3F5F728000000000FF9896810000000000FA] [regions=1]
```

[localhelper.go:108](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/localhelper.go#L108), [localhelper.go:116](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/localhelper.go#L108): Lightning [scans a batch of regions](https://github.com/pingcap/tidb/blob/55f3b24c1c9f506bd652ef1d162283541e428872/br/pkg/restore/split.go#L413) (limit = 128) at a time in the key range (minKey/start, maxKey/end).  

```
[INFO] [split_client.go:460] ["checking whether need to scatter"] [store=1] [max-replica=3]   
[INFO] [split_client.go:113] ["skipping scatter because the replica number isn't less than store count."]
```

[split_client.go:460](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/restore/split_client.go#L460), [split_client.go:113](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/restore/split_client.go#L113): Lightning skips the scatter regions phase because max-replica <= number of TiKV stores. Scattering regions is the process that PD schedulers distributes regions and their replicas to different TiKV stores.

```
[INFO] [localhelper.go:240] ["batch split region"] [region_id=2] [keys=23] [firstKey="dIAAAAAAAAA/X3KAAAAAAAAAAQ=="] [end="dIAAAAAAAAA/X3KAAAAAAJiWgQ=="]
```

[localhelper.go:240](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/localhelper.go#L240): Batch splitting regions has completed. At first, there is only one region containning all KV pairs. Lightning [splits regions](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/localhelper.go#L361) from a batch of keys through the following steps:

 1. Lightning [sends the split request](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/restore/split_client.go#L377) to the TiKV node.
 2. Lightning then [splits the current region](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/restore/split_client.go#L281) into left region and right region from the given key. The left region is the same as the original region except the range of keys, while the right region is a new created region.
 3. In the end, Lightning [scatters the splited regions](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/localhelper.go#L374) one by one.

```
[INFO] [localhelper.go:319] ["waiting for scattering regions done"] [skipped_keys=0] [regions=23] [take=6.505195ms]
```

[localhelper.go:319](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/localhelper.go#L319): Region scattering has completed. If the waiting time exceeds [ScatterWaitUpperInterval](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/localhelper.go#L313), it shows timeout log: waiting for scattering regions timeout.

```
[INFO] [local.go:1371] ["import engine success"] [uuid=d173bb2e-b753-5da9-b72e-13a49a46f5d7] [size=2159933993] [kvs=10000000] [importedSize=2159933993] [importedCount=10000000]   
[INFO] [backend.go:455] ["import completed"] [engineTag=`sysbench`.`sbtest1`:0] [engineUUID=d173bb2e-b753-5da9-b72e-13a49a46f5d7] [retryCnt=0] [takeTime=20.179184481s] []
```

[local.go:1371](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/local/local.go#L1371), [backend.go:455](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/backend.go#L455): Lightning has completed importing KV pairs in the specific engine to the TiKV stores.

``` 
[INFO] [backend.go:467] ["cleanup start"] [engineTag=`sysbench`.`sbtest1`:0] [engineUUID=d173bb2e-b753-5da9-b72e-13a49a46f5d7]   
[INFO] [backend.go:469] ["cleanup completed"] [engineTag=`sysbench`.`sbtest1`:0] [engineUUID=d173bb2e-b753-5da9-b72e-13a49a46f5d7] [takeTime=209.800004ms] []
```

[backend.go:467](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/backend.go#L467), [backend.go:469](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/backend/backend.go#L469): Clean up intermediate data during import phase. It will [cleanup](https://github.com/pingcap/tidb/blob/55f3b24c1c9f506bd652ef1d162283541e428872/br/pkg/lightning/backend/local/engine.go#L158) engine related meta info and db files.

```
[INFO] [table_restore.go:946] ["import and cleanup engine completed"] [engineTag=`sysbench`.`sbtest1`:0] [engineUUID=d173bb2e-b753-5da9-b72e-13a49a46f5d7] [takeTime=20.389269402s] []
```

[table_restore.go:946](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L946): Import and cleanup has completed.

```
[INFO] [table_restore.go:345] ["import whole table completed"] [table=`sysbench`.`sbtest1`] [takeTime=47.421324969s] []
```

[table_restore.go:345](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L345): Importing table data has completed. Lightning has converted all table data into KV pairs and ingested them into the TiKV clusters.

```
[INFO] [tidb.go:401] ["alter table auto_increment start"] [table=`sysbench`.`sbtest1`] [auto_increment=10000002]   
[INFO] [tidb.go:403] ["alter table auto_increment completed"] [table=`sysbench`.`sbtest1`] [auto_increment=10000002] [takeTime=82.225557ms] []
```

[tidb.go:401](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/tidb.go#L401), [tidb.go:403](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/tidb.go#L403): During [post process](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L680) phase, it will [adjust table auto increment](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L703) to avoid introducing conflicts from newly added data.

```
[INFO] [restore.go:1466] ["restore table completed"] [table=`sysbench`.`sbtest1`] [takeTime=53.280464651s] []
```

[restore.go:1466](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L1466): Table restore has completed.

```
[INFO] [restore.go:1396] ["add back PD leader&region schedulers"]    
[INFO] [pd.go:462] ["resume scheduler"] [schedulers="[balance-region-scheduler,balance-leader-scheduler,balance-hot-region-scheduler]"]    
[INFO] [pd.go:448] ["exit pause scheduler and configs successful"]    
[INFO] [pd.go:482] ["resume scheduler successful"] [scheduler=balance-region-scheduler]   
[INFO] [pd.go:573] ["restoring config"] [config="{\"enable-location-replacement\":\"true\",\"leader-schedule-limit\":4,\"max-merge-region-keys\":200000,\"max-merge-region-size\":20,\"max-pending-peer-count\":64,\"max-snapshot-count\":64,\"region-schedule-limit\":2048}"]
```

[restore.go:1396](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L1396), [pd.go:462](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/pdutil/pd.go#L462), [pd.go:448](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/pdutil/pd.go#L448), [pd.go:482](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/pdutil/pd.go#482), [pd.go:573](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/pdutil/pd.go#573): Restore PD from import mode to normal mode, resume paused PD schedulers before import, and reset PD configs.

```
[INFO] [restore.go:1244] ["cancel periodic actions"] [do=true]
```

[restore.go:1244](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L1244): Start to cancel [periodic actions](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L1087), which periodically prints the importing progress, and check  whether TiKV is still in import mode.

```
[INFO] [restore.go:1688] ["switch to normal mode"]
```

[restore.go:1688](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L1688): Switch TiKV back to the normal mode.

```
[INFO] [table_restore.go:736] ["local checksum"] [table=`sysbench`.`sbtest1`] [checksum="{cksum=9970490404295648092,size=2539933993,kvs=20000000}"]   
[INFO] [checksum.go:172] ["remote checksum start"] [table=sbtest1]   
[INFO] [checksum.go:175] ["remote checksum completed"] [table=sbtest1] [takeTime=2.817086758s] []   
[INFO] [table_restore.go:971] ["checksum pass"] [table=`sysbench`.`sbtest1`] [local="{cksum=9970490404295648092,size=2539933993,kvs=20000000}"]
```

[table_restore.go:736](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L736), [checksum.go:172](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/checksum.go#L172), [checksum.go:175](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/checksum.go#L175), [table_restore.go:971](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L971): [Compare local and remote checksum](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L736) to validate the imported data.

```
[INFO] [table_restore.go:976] ["analyze start"] [table=`sysbench`.`sbtest1`]   
[INFO] [table_restore.go:978] ["analyze completed"] [table=`sysbench`.`sbtest1`] [takeTime=26.410378251s] []
```

[table_restore.go:976](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L976), [table_restore.go:978](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/table_restore.go#L978): TiDB analyzes table to update the statistics that TiDB builds on tables and indexes. It is recommended to run `ANALYZE` after performing a large batch update or import of records, or when you notice that query execution plans are sub-optimal.

```
[INFO] [restore.go:1440] ["cleanup task metas"]
```

[restore.go:1440](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L1440): clean up task metas, table metas and schema db if needed.

```
[INFO] [restore.go:1656] ["skip full compaction"]
```

[restore.go:1656](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L1656): [fullCompact](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L1654) determines if Lightning needs to compact data based on cfg.PostRestore.Compact.

```
[INFO] [restore.go:1842] ["clean checkpoints start"] [keepAfterSuccess=remove] [taskID=1650516927467320997]   
[INFO] [restore.go:1850] ["clean checkpoints completed"] [keepAfterSuccess=remove] [taskID=1650516927467320997] [takeTime=18.543µs] []
```

[restore.go:1842](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L1842), [restore.go:1850](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L1850): Clean up checkpoints.

```
[INFO] [restore.go:473] ["the whole procedure completed"] [takeTime=1m22.804337152s] []
```

[restore.go:473](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L473): The whole importing procedure has completed.

```
[INFO] [restore.go:1143] ["everything imported, stopping periodic actions"]
```

[restore.go:1143](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L1143): Stop all [periodic actions](https://github.com/pingcap/tidb/blob/v5.4.0/br/pkg/lightning/restore/restore.go#L1087) after importing completed.