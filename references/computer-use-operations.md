# Computer Use 操作模板

控制招聘页面前读取本参考。只使用当前 Computer Use 文档公开的应用级接口，不猜测方法名、参数键或辅助功能动作。

## 取得和复用 Chrome

首次接管时使用：

```javascript
let chrome = await cua.getApp("Google Chrome");
```

`getApp`已经返回初始辅助功能状态；同一执行会话后续复用`chrome`，不重复取得应用。显示名无法解析时，从`cua.listApps()`取得 Google Chrome 的 bundle id，再用该 id 重试一次。

本文件已经给出当前流程需要的调用形式。不得在任务执行中通过`Object.keys(chrome)`、`String(chrome.drag)`、`String(chrome.scroll)`、`String(chrome.pressKey)`或类似运行时自省重新探索 API；现有模板无法完成动作时，按下方故障恢复规则换一种已记录的定位方式或停止回传。

本工作流只控制 Google Chrome 应用对象。不要改用浏览器标签对象、外部扩展桥接、CDP 或其他自动化方式。

## 观察模板

- 普通结果确认：`await chrome.getAXState()`。保留默认 diff；它只返回相对上次观察的变化。
- 确需重新建立完整基线：`await chrome.getAXState({ disableDiffing: true })`。仅用于首次接管后的必要复核、网站解析或插件批量动作导致大范围重建、截图后需要重新取得元素索引、最终交审或故障恢复。
- 纯视觉问题：`await chrome.getScreenshot()`；只有同时确实需要完整结构和图像时才使用`getAXStateAndScreenshot()`。
- 独立观察没有变化时，不立即重复观察。只有能说明缺少的是视觉信息还是完整结构时，才切换表示方式。

截图后若要再次依赖辅助功能元素索引，先取得一次完整辅助功能状态。不要从旧截图或旧状态继续使用元素索引。

## 动作模板

优先使用当前辅助功能状态中的数字元素索引：

```javascript
await chrome.click(42);
await chrome.setValue(57, "示例值");
await chrome.pressKey("Tab");
await chrome.getAXState();
```

只有元素不可访问且最新截图能够可靠定位时才使用坐标，坐标必须是二元数组：

```javascript
await chrome.click([640, 480]);
await chrome.scroll([640, 700], "down", 1);
await chrome.drag([300, 200], [700, 500]);
```

允许的方法与关键参数只有：

- `click(number | [x,y], { mouseButton?, clickCount? })`
- `setValue(elementIndex, value)`，只用于状态明确显示可编辑或可设置值的元素；否则先点击目标后用`typeText`或`paste`
- `paste(text, { format: "text" | "md" | "html" })`
- `typeText(text)`
- `pressKey(key)`
- `scroll(number | [x,y], direction, pages?)`
- `drag([x1,y1], [x2,y2])`
- `selectText(elementIndex, text, options?)`
- `performSecondaryAction(elementIndex, action)`，只调用当前辅助功能状态明确列出的 action

不存在`clickAt`。不得把坐标写成`{x, y}`，不得猜测`key`、`keys`、`textValue`等参数变体，也不得猜测未显示的 secondary action。

## 稳定区块批量操作

结果可预期、不会导航、不会弹窗、不会新增/隐藏字段且失败可从局部状态识别时，把同一区块的动作和一次结果观察放在同一 REPL 调用中：

```javascript
await chrome.setValue(21, "值一");
await chrome.click(28);
await chrome.pressKey("Tab");
await chrome.getAXState();
```

一个稳定区块通常采用“一次前置状态 → 一组动作 → 一次结果状态”。最后一个动作已经由随后的 diff 明确证明结果时，不再紧接着单独调用第二次`getAXState()`。新增经历、附件上传、网站解析、插件整体填写、会改变页面分支的选择、声明授权和最终提交不得与其他普通字段合并。

## 错误恢复

动作抛错、元素失效或页面结构意外变化时：

1. 立即停止当前批次剩余动作，保留已经成功写入的字段；
2. 获取当前区块的新 diff；若索引来源是截图或结构已大范围重建，改取一次完整状态；
3. 从新状态重新定位，只重做失败动作，不重放已经成功的动作；
4. 使用当前文档支持的另一种定位方式进行一次有区别的重试；
5. 同一动作再次失败时停止并回传准确错误、当前状态和下一安全动作。

不要连续试验多个方法名、坐标格式、参数键或旧元素索引。超时后先检查实际页面是否已经变化，再决定是否重试，不能把超时本身当作失败证据。

教育、项目、实习、家庭等重复记录中的`删除`、垃圾桶、减号和移除按钮视为破坏性控件。除非队列中已证明存在重复或错误记录且当前动作目的明确是删除，否则不得点击。编辑或展开记录时，只使用当前状态中名称或语义明确为`编辑`、记录标题或展开入口的元素；删除控件附近的旧索引、无标签坐标或视觉猜测不能用于打开记录。若记录意外消失，立即停止其他动作，将其记为`record_error`，只从同一条已确认来源恢复一次并保存、复核，恢复完成前不得继续其他模块。

多列日期/时间滚轮只有在目标项、当前项和移动方向都可判定，且一次动作后能可靠读回值时才自动操作。若只能通过坐标拖动和截图猜测位置，最多做一次有明确校准依据的尝试；一次后仍不确定，立即取消错误值并回传为`requires_local_interaction`。不得连续拖动、批量循环拖动或逐次截图搜索目标日期。
