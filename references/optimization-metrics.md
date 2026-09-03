# 投递优化指标

在代表性真实投递中记录本参考的紧凑指标，用于和历史基线比较。只使用运行时、任务线程或现有操作记录直接提供的数据；不得为统计重新读取网页、扫描完整线程日志、解析缓存或增加浏览器操作。

## 指标结构

```json
{
  "wall_clock_seconds": null,
  "agent_active_seconds": null,
  "user_wait_seconds": null,
  "usage": {
    "root_input": null,
    "root_cached_input": null,
    "root_uncached_input": null,
    "root_output": null,
    "child_input": null,
    "child_cached_input": null,
    "child_uncached_input": null,
    "child_output": null
  },
  "computer_use": {
    "calls": 0,
    "state_reads": 0,
    "full_state_reads": 0,
    "scoped_or_diff_reads": 0,
    "screenshots": 0,
    "observed_output_chars": null,
    "avoidable_api_errors": 0,
    "recovery_retries": 0
  },
  "plugin": {
    "continue_fill_runs": 0,
    "region_fill_runs": 0,
    "flash_fill_runs": 0
  },
  "browser_tabs": {
    "opened_by_stage": 0,
    "closed_as_transient": 0,
    "retained_for_continuation": 0,
    "uncertain_ownership_not_closed": 0
  },
  "workflow": {
    "planned_user_checkpoints": 0,
    "unplanned_user_interruptions": 0,
    "user_info_rounds": 0,
    "form_gaps_found": 0,
    "form_gaps_reopened": 0,
    "omissions_found_at_final_review": 0,
    "misfills_found_at_final_review": 0,
    "official_readback_ok": false,
    "ledger_readback_ok": false
  }
}
```

无法直接取得的字段保留`null`，不能按页面数量、消息长度或墙钟时间猜算。名义 token 必须把输入、缓存输入、未缓存输入和输出分列；缓存输入仍属于名义调用量，不能当作实际计费 token，也不能据此推断费用。

## 采集责任

- Search/Form Agent 在自己的现有操作记录中累计 Computer Use、插件、错误和缺口计数，阶段结束时只回传一行指标。
- 主 Agent 记录直接可得的总墙钟、用户等待、主/子 Agent usage、人工检查点以及官网和台账结果。
- 每个共享候选人档案的`continue_fill_runs`单独核对；明确失败且没有写入页面的尝试单独计入错误，不伪装成成功运行。
- 阶段 Agent 只统计自己明确新开的标签；阶段开始前已有或归属不确定的标签不得归到可清理数量中。Search 阶段结束时记录关闭的临时检索标签和保留的目标公司官方职位页。
- 最终审核发现的遗漏或误填单独计数，用来判断调用减少是否牺牲正确性。

## 判断方式

先与同类历史任务比较，再决定是否调整默认流程。优先观察：重复状态读取、可避免 API 错误、每份共享档案的整页插件次数、主动执行时间、未缓存输入，以及最终审核遗漏/误填是否增加。

工程目标是重复状态读取和可避免 API 错误为 0、每份共享档案成功的整页填写不超过 1 次，并在正确性和官网/台账证据不下降时减少调用和耗时。没有足够同类样本时不承诺节省比例，也不据单家公司结果决定推理强度或 Agent 策略。
