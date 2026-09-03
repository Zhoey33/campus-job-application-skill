# Campus Job Application Skill

面向 Codex Desktop 的端到端校招投递 Skill。它负责官网岗位检索、简历与表单处理、多志愿提交、结果回读和飞书台账维护；用户只处理少数必要的人工卡点。

## 使用方法

安装后直接在 Codex 中说：

```text
按照校招投递 Skill 的全流程，投一下 XX 公司的校招。
```

也可以在当前请求中指定届次和方向：

```text
按照校招投递 Skill 的全流程，投一下 XX 公司的 2027 届校招，找财务和审计方向的岗位。
```

公司以当前请求为准，不会保存为下次的默认公司。

## 工作流程

```text
初始化 → 登录与额度核验 → 官方岗位检索 → 岗位确认
       → 表单填写与复核 → 整组提交确认 → 逐志愿提交 → 官网与台账回读
```

### 三个人工卡点

| 卡点 | 用户操作 | 后续自动执行 |
|---|---|---|
| 登录 | 完成扫码、验证码、密码输入或浏览器权限确认 | 核验登录状态、投递上限、已投数和剩余额度 |
| 岗位确认 | 确认目标岗位和志愿顺序 | 准备整组志愿并填写、校验表单 |
| 提交确认 | 一次审核全部志愿并授权提交 | 按顺序提交、逐项回读并更新台账 |

无法从现有资料确定个人事实时，Skill 会提出具体问题；其余浏览器操作由 Skill 完成。

## 自动能力

| 模块 | 行为 |
|---|---|
| 岗位核验 | 登录后从官方招聘系统完整检索岗位，并核对届次、地点和投递额度 |
| 表单填写 | 上传 PDF、触发网站简历解析，调用“来个简历”填写并复核组件、附件和必填项 |
| 字数限制 | 先读取页面限制，忠实压缩超限文本，再按网站计数器和校验结果复核 |
| 多志愿 | 第一志愿完整填写；后续志愿复用共享候选人档案，只处理岗位差异并连续提交 |
| 内推码 | 第一志愿从小红书等公开来源查找并验证；后续志愿复用，不重复搜索 |
| 结果记录 | 每个志愿分别回读官网状态，提交后重新检查额度，并维护飞书个人台账 |

Skill 不会把第一志愿成功当作整组成功。出现新必填事实、验证码、额度或顺序变化、岗位失效或提交异常时才会暂停。

## 首次运行

### 依赖

- Codex Desktop、Computer Use 和 Chrome；
- 官方 [`lark-cli`](https://github.com/larksuite/cli) 及所需飞书 Skills；
- Chrome 扩展[“来个简历”](https://cv.playoffer.cn/)；
- 配套 Skill：[laige-resume-autofill](https://github.com/Zhoey33/laige-resume-autofill-skill)。

Skill 会检查这些依赖，并在需要时引导完成安装、登录和授权。

### 首次提供的信息

- 目标届次或毕业年份；
- 默认岗位方向和关键词；
- 飞书个人投递台账；
- 本地资料文件夹绝对路径。

请提前把 PDF 简历、证件照、成绩单、证书和其他证明材料放在同一个本地文件夹。只需提供文件夹路径，不必逐个提供文件路径；同类文件存在多个且无法判断时，Skill 才会请你选择。

首次初始化会生成本地 `context.md` 和 `references/current-profile-field-map.md`。后续运行会复用有效信息，不重复询问。

## 安装

```bash
git clone https://github.com/Zhoey33/campus-job-application-skill.git ~/.codex/skills/campus-job-application-workflow
git clone https://github.com/Zhoey33/laige-resume-autofill-skill.git ~/.codex/skills/laige-resume-autofill
```

两项安装完成后，重新启动 Codex 或刷新 Skills。

## 隐私与边界

- 最终提交必须由用户明确授权；一次授权可以覆盖当前公司已列明的整组志愿，不会跨公司或跨任务复用。
- Skill 不编造个人经历、成绩、证书、内推码、推荐人或推荐关系，也不会通过私信、加群、付费或提交个人信息换取内推码。
- `context.md`、`references/current-profile-field-map.md` 和 `references/supplemental-information.md` 已被 `.gitignore` 排除，不会上传到公开仓库。
- 不要在 Issue、提交信息或示例中粘贴简历内容、联系方式、飞书私有链接或本地路径。
