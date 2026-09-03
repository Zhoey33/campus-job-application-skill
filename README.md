# campus-job-application-workflow

一个面向 Codex Desktop 的中文校招投递 Skill，覆盖首次环境初始化、官方岗位检索、志愿确认、来个简历插件填表、附件与表单复核，以及飞书个人投递台账维护。

## 首次使用

Skill 会在第一次运行时：

1. 检查并在需要时安装官方`lark-cli`及其配套 Skills；
2. 检查 Chrome 是否安装“来个简历”，缺失时从[来个简历官网](https://cv.playoffer.cn/)进入官方安装流程；
3. 让用户完成插件登录、同步和简历档案准备；
4. 一次性取得目标届次、岗位方向、飞书个人台账和本地资料文件夹绝对路径；
5. 创建本地`context.md`并生成`references/current-profile-field-map.md`。

后续运行会复用这些信息，只在用户更改、路径失效或插件档案切换时更新。公司始终从当前任务消息中取得，不会持久化为默认公司。

## 准备本地资料文件夹

首次使用前，请把网申可能用到的资料集中放进同一个本地文件夹，例如：

- 当前使用的 PDF 简历；
- 证件照；
- 成绩单；
- 英语、计算机或职业资格证书；
- 获奖证明及招聘网站可能要求的其他附件。

然后只需要把这个文件夹的绝对路径告诉 Codex，不必逐个提供文件路径。Skill 会根据文件名和格式建立附件清单；如果同类文件有多个且无法确定使用哪一个，Codex 才会列出候选文件名请你选择。

## 隐私

以下运行时文件已被`.gitignore`排除，不应提交到 GitHub：

- `context.md`
- `references/current-profile-field-map.md`
- `references/supplemental-information.md`

公开仓库只包含空白模板。不要在 Issue、提交信息或示例中粘贴个人简历、联系方式、飞书私有链接或本地路径。

## 依赖

- Codex Desktop 与 Computer Use；
- Chrome；
- 官方[`lark-cli`](https://github.com/larksuite/cli)；
- Chrome 扩展“来个简历”；
- 配套 Skill：`laige-resume-autofill`。

## 安装

将本目录放入 Codex Skills 目录，并保持目录名为`campus-job-application-workflow`。同时安装`laige-resume-autofill`。重新启动 Codex 或刷新 Skills 后即可使用。

## 在 Codex 中使用

安装完成后，不需要记住 Skill 的英文名称或输入命令。直接在 Codex 中说：

```text
按照校招投递 Skill 的全流程，投一下 XX 公司的校招。
```

Codex 会自动调用本 Skill。第一次使用时会先完成环境检查、插件设置和默认信息初始化；后续使用会复用本地上下文，并从当前消息中的`XX 公司`确定本次任务范围。

也可以在同一句话中覆盖默认届次或岗位方向，例如：

```text
按照校招投递 Skill 的全流程，投一下 XX 公司的 2027 届校招，找财务和审计方向的岗位。
```

首次初始化中的依赖安装、浏览器扩展权限确认、扫码登录和飞书授权仍受本机与宿主的权限规则约束。最终投递必须由用户对当前公司和岗位明确授权。
