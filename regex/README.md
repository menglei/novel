# 酒馆配套正则：隐藏生图标签

适用于本插件默认标签 `image###提示词###`。请先安装包含隐藏占位标记兼容处理的本次插件构建，再导入规则。旧构建不识别占位标记，可能把图片放进隐藏元素，导致图片不可见。

## 导入操作

1. 打开酒馆的「扩展」→「正则 / Regex」。
2. 点击「导入」，选择同目录的 `novelai-hide-image-tags.json`，导入为全局规则。所有角色都可使用；只给当前角色使用时，可选择局部/角色规则并允许该角色的正则。
3. 检查规则已启用，作用范围为「AI 输出」，勾选「仅影响显示」（新版英文为 Alter Chat Display）和「在编辑时运行」。不勾选「仅影响提示词 / Alter Outgoing Prompt」。最小/最大深度留空，宏替换选择「不替换」。
4. 重新加载聊天。正文中只显示故事和图片，标签摘要、工具栏、编辑框和任务状态均移入图片管理窗口。单击图片即可编辑标签、生成或重绘、下载、切换历史版本和查看参数。尚未生成或已隐藏的图片可从悬浮球 →「正文与图片」→「图片管理」进入。

如果流程设置开启了「把图片标题和标签思考插回正文」，还可导入 `novelai-hide-tag-thinking.json`，把它排在生图标签规则之前，隐藏 `<Tag_think>…</Tag_think>` 和 `<imgthink>…</imgthink>`。该规则只处理完整闭合的区块。

## 手动填写主规则

查找正则（连同两端 `/` 和末尾 `g` 一起复制）：

```regex
/image###(?:(?!image###)[\s\S])*?###/g
```

替换为：

```html
<textarea data-nai-regex-anchor hidden>{{match}}</textarea>
```

`[\s\S]` 支持多行提示词，`g` 支持同一条消息的多张图。中间的排除条件避免未闭合的前一个标签吞掉下一个完整标签。隐藏 textarea 保留标签原文和所在位置，插件会把整个占位元素替换成图片控件，因此图片不会继承隐藏状态。未闭合的标签保持原样，便于排查格式错误。

**「仅影响显示」必须勾选。** 酒馆默认的普通替换会修改消息原文；如果删掉原文中的标签，插件就无法识别它。也不要把主规则的替换内容留空：虽然原文能保留，图片在渲染页面中的位置会丢失。

默认只针对 AI 消息；本插件不扫描用户消息。若自己修改了插件的开始/结束标记，请同步修改主规则，转义自定义标记中的正则特殊字符。酒馆「编码 HTML 标签」应关闭，否则占位 HTML 会以文字显示。已有其他删除生图标签的显示规则时，请先停用，避免互相干扰。

导入规则只影响聊天显示，原始消息、发送给 LLM 的历史以及插件正文编辑器保留原标签。想查看原始 tag，可单击图片打开管理窗口，或点酒馆消息编辑按钮。

## 研究依据

- 本插件 `src/tags/scanner.ts` 默认识别 `image###…###`，`src/host/sillytavern.ts` 从 `chat[].mes` 读取原始消息。
- 酒馆 1.19.0 的 `public/scripts/extensions/regex/engine.js`：`markdownOnly: true` 只在聊天显示格式化时应用；`placement: [2]` 对应 AI 输出；`{{match}}` 返回完整匹配文本。
- 酒馆 `public/scripts/extensions/regex/index.js` 支持直接导入单条规则 JSON；上述文件使用其导出字段格式。
- 本次 `src/ui/messages.ts` 支持隐藏占位元素，阅读视图只渲染图片；`src/ui/image-manager.ts` 在独立窗口内提供原有操作，任务刷新时保留窗口与草稿。

验证记录见 `evidence/regex-display-verification.json`。浏览器验证使用本机酒馆的正则引擎、Showdown、DOMPurify 与隔离的模拟宿主，不调用收费接口；覆盖多行、多标签、权重、正文位置、生成、重绘、刷新和原文保留。复现时先执行 `node tests/browser/build.mjs` 并启动 `node tests/browser/server.mjs`，再运行 `node scripts/browser-regex-test.mjs`；可用 `ST_SOURCE_PATH` 指定酒馆源码路径，用 `NAI_TEST_URL` 指定测试服务器地址。
