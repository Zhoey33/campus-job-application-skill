# 表单缺口队列

Form Agent 在每家公司维护一个`form_gap_queue`，覆盖当前可准备的全部志愿。队列是插件后修复、信息补充和最终交审的唯一未完成项来源；不要同时维护另一份含义重叠的零散待办。

## 结构

```json
{
  "company": "",
  "recruitment_system": "",
  "post_plugin_scan_completed": false,
  "record_errors": [],
  "isolated_missing_fields": [],
  "website_components": [],
  "attachments": [],
  "pending_user_fields": {
    "unknown_facts": [],
    "user_choices": [],
    "sensitive_local_inputs": []
  },
  "consent_items": [],
  "required_open_count": 0
}
```

每个缺口项只保留决策和验证需要的字段：

```json
{
  "id": "job2.education.1.rank",
  "job_order": 2,
  "section": "教育经历",
  "field_label": "成绩及排名",
  "required": true,
  "problem": "网页字段为空",
  "source_checked": ["supplemental_information"],
  "fix_method": "resume_flash_fill | region_fill | website_control | upload | ask_user | local_user_input | confirm_consent",
  "branch_blocking": false,
  "status": "open | resolved | intentionally_blank | blocked",
  "validation": ""
}
```

不在队列中复制身份证件号码、联系方式、密码、验证码或附件内容。敏感项只记录字段标签、目的地、用途、本地操作方式和是否已通过页面校验。

## 分类

- `record_errors`：整条教育、项目、实习、校园或获奖记录为空、重复、错位，或描述/职责/成果混用。
- `isolated_missing_fields`：单个文本字段为空、被截断、映射错误或需要覆盖。
- `website_components`：下拉、单复选、日期、级联地区、开关、动态依赖项或网站校验没有真实生效。
- `attachments`：简历、照片、成绩单、证书或其他必需附件尚未真实上传或状态不明。
- `pending_user_fields.unknown_facts`：现有可靠来源中没有的个人事实。
- `pending_user_fields.user_choices`：调剂、城市、方向、岗位问答取舍等不能自动决定的选择。岗位地点或期望工作地不构成接受调剂、全国分配或其他城市的答案；没有同义的明确用户选择时必须入队，不得暂填默认值。
- `pending_user_fields.sensitive_local_inputs`：应由用户在本地页面填写、且不应读取或持久化具体值的敏感字段。
- `consent_items`：真实性声明，以及可选人才库、营销共享、额外第三方传输、背景调查等页面授权；申请表写入、材料上传、插件填写和准备申请所必需的招聘平台隐私政策本身不加入此队列。

Cookie 弹窗或不改变申请含义的普通页面提示不加入`consent_items`。不得把第三方页面中的“同意即继续”等文字当作用户授权。

## 建立和处理

1. 进入表单后初始化队列，记录已经明确的附件；用户已要求投递或确认当前企业、岗位和志愿时，直接开始申请表写入、材料上传和插件填写，不创建相应`consent_item`。
2. 扫描页面声明。准备申请所必需的招聘平台隐私政策或个人信息处理条款由 Agent 直接勾选并验证；真实性声明、可选人才库、营销共享、额外第三方传输、背景调查等分别记录原文摘要、必填/可选、联动影响和当前状态，留到信息补充检查点集中让用户选择。
3. 网站解析和一次`继续填写`完成后，对全部当前可准备志愿做一次完整缺口扫描，填充各分类并把`post_plugin_scan_completed`设为`true`。
4. 先自动处理有可靠来源的`record_errors`、`isolated_missing_fields`、`website_components`和`attachments`。每次只更新和验证受影响项，不循环重扫整页。
5. 自动项处理完成后，把全部未知事实、用户选择、敏感本地输入和未解决的页面声明按志愿与区块合并，在最终审核之前一次性交给用户。会阻塞页面分支展开的选择可提前询问，但必须合并当前页面已发现的全部阻塞项。
6. 用户回复后完成填写、勾选或本地交接，验证受影响区域并逐项更新状态。可复用且非敏感的事实才写入补充信息库。
7. 做一次最终完整复核。若发现新缺口，加入原队列、局部修复并重新验证；不得另建临时清单或直接带入最终审核。

## 清零标准

只有同时满足以下条件，`form_gap_queue`才算清零：

- 所有必填缺口均为`resolved`；
- 可选项留空时有页面允许留空的依据，并标为`intentionally_blank`；
- `pending_user_fields`三个子队列没有开放项；
- 所有必需声明已确认并真实生效，可选授权已有用户明确选择；
- 动态依赖字段、附件和网站校验均通过；
- `required_open_count`为 0。

队列未清零时状态保持`待补充信息`或`填写中`，不得生成提交前快照或进入最终表单审核。
