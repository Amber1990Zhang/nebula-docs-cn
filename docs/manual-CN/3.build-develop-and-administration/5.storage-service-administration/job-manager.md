# 作业管理（flush 和 compact）

作业特指在存储层运行的一些长任务。比如 `compact` 和 `flush`等。管理指对作业进行管理。比如让作业排队执行、查看作业状态、停止作业、恢复作业等。

## 命令列表

### SUBMIT JOB COMPACT

`SUBMIT JOB COMPACT` 命令触发长耗时的 `RocksDB compact` 操作。示例返回结果如下：

```ngql
nebula> SUBMIT JOB COMPACT;
==============
| New Job Id |
==============
| 40         |
--------------
```

修改默认 compact 线程数量请参考[这里](../../3.build-develop-and-administration/3.configurations/5.storage-config.md)。

### SUBMIT JOB FLUSH

`SUBMIT JOB FLUSH` 命令将内存中的 RocksDB memfile 写入到硬盘中。

```ngql
nebula> SUBMIT JOB FLUSH;
==============
| New Job Id |
==============
| 2          |
--------------
```

### SHOW JOB

#### 返回单个作业信息

命令 `SHOW JOB <job_id>` 用于返回对应 ID 作业及其所有任务。作业到达 Meta 层后，Meta 会将作业分成多个任务并发送至 storage 层。

```ngql
nebula> SHOW JOB 40;
=====================================================================================
| Job Id(TaskId) | Command(Dest) | Status   | Start Time        | Stop Time         |
=====================================================================================
| 40             | flush nba     | finished | 12/17/19 17:21:30 | 12/17/19 17:21:30 |
-------------------------------------------------------------------------------------
| 40-0           | 192.168.8.5   | finished | 12/17/19 17:21:30 | 12/17/19 17:21:30 |
-------------------------------------------------------------------------------------
```

执行上述命令将返回 1 到多行结果，取决于 space 所在的 storaged 个数。

返回结果说明：

- `40` 为当前作业 ID
- `flush nba` 表示在 nba 图空间上执行了 flush 操作
- `finished` 表示运行已结束，且成功。其他可能的状态有 Queue、running、failed、stopped
- `12/17/19 17:21:30` 表示开始时间，初始为空(Queue)，当且仅当状态变为 running 时，才会设这个值
- `12/17/19 17:21:30` 表示结束时间，如果为 Queue 或者 running 状态，这里会为空，当状态变为 finished、failed、stopped 时会设置此值
- `40-0` 表示当前作业 ID 是 40，任务 ID 是 0
- `192.168.8.5` 表示运行在 192.168.8.5 这台机器上
- `finished` 表示运行已结束，且成功。其他可能的状态有 Queue、running、failed、stopped
- `12/17/19 17:21:30` 表示开始时间，因为任务初始即为 running 状态，所以这里永不为空
- `12/17/19 17:21:30` 表示结束时间，如果为 running 状态，这里会为空，当状态变为 finished、failed、stopped 时会设置此值

**注意：** 作业状态有五种，分为是 QUEUE、RUNNING、FINISHED、FAILED、STOPPED。状态机转换见以下说明：

```ngql
Queue -- running -- finished -- removed
     \          \                /
      \          \ -- failed -- /
       \          \            /
        \ ---------- stopped -/
```

#### 返回所有作业信息

命令 `SHOW JOBS` 用于列出所有未过期的作业信息。默认作业过期时长为一周。用户可通过 `job_expired_secs` 参数更改过期时长。

```ngql
nebula> SHOW JOBS;
=============================================================================
| Job Id | Command       | Status   | Start Time        | Stop Time         |
=============================================================================
| 22     | flush test2   | failed   | 12/06/19 14:46:22 | 12/06/19 14:46:22 |
-----------------------------------------------------------------------------
| 23     | compact test2 | stopped  | 12/06/19 15:07:09 | 12/06/19 15:07:33 |
-----------------------------------------------------------------------------
| 24     | compact test2 | stopped  | 12/06/19 15:07:11 | 12/06/19 15:07:20 |
-----------------------------------------------------------------------------
| 25     | compact test2 | stopped  | 12/06/19 15:07:13 | 12/06/19 15:07:24 |
-----------------------------------------------------------------------------
```

返回结果说明见上节[返回单个作业信息](#返回单个作业信息)。

### STOP JOB

命令 `STOP JOB` 用于在停止未完成的作业。

```ngql
nebula> STOP JOB 22;
=========================
| STOP Result         |
=========================
| stop 1 jobs 2 tasks |
-------------------------
```

### RECOVER JOB

命令 `RECOVER JOB` 用于重新执行失败的作业，并返回 recover 的作业数目。

```ngql
nebula> RECOVER JOB;
=====================
| Recovered job num |
=====================
| 5 job recovered   |
---------------------
```

## FAQ

`SUBMIT JOB` 使用 HTTP 端口。请检查 Storage 之间的 HTTP 端口是否正常。你可以使用如下命令调试。

```bash
curl "http://{storaged-ip}:12000/admin?space={test}&op=compact"
```
