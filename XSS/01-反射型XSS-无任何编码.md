# Lab：Reflected XSS into HTML context with nothing encoded

> 难度：APPRENTICE ｜ 分类：XSS ｜ 状态：📖 解析（待本人实操验证）
> 官方页面：https://portswigger.net/web-security/cross-site-scripting/reflected/lab-html-context-nothing-encoded

## 一、漏洞原理

应用把请求里的数据放进**即时响应**且未做安全处理：

```
https://insecure-website.com/search?term=gift
→ <p>You searched for: gift</p>
```

若输入被原样拼进 HTML 文本节点，浏览器解析到 `<script>` 就当作代码执行。漏洞本质是**数据被当作代码解析**，不是"输入没过滤"。反射型 XSS 需要诱导受害者点击构造好的链接，危害通常低于存储型，但能完全冒用受害者会话执行操作。

## 二、标准利用链

```
/search?term=<script>alert(1)</script>
```

在搜索框输入或在 Burp 里改 `term` 参数，页面弹窗即通过。官方建议用 `alert(document.domain)`，能顺带确认脚本执行的域。

**真正的难点是判断回显落在哪个上下文**——上下文判断错了 payload 永远不生效：

| 回显位置 | 典型 payload |
|---|---|
| HTML 文本节点 | `<script>alert(1)</script>` 或 `<img src=x onerror=alert(1)>` |
| 标签属性内（双引号） | `"><img src=x onerror=alert(1)>` |
| `<script>` 字符串内部 | `</script><script>alert(1)</script>` |

**手工测试流程**（官方方法）：先提交一个约 8 位的随机字母数字串，逐个入口找它是否回显；再确定每个回显点的上下文；然后在 Repeater 里把候选 payload 插在随机串前后，用随机串作搜索词快速定位回显位置；最后一定要**放回真实浏览器**验证脚本确实执行，Repeater 里响应看着对不代表浏览器会执行。

## 三、我的利用步骤（实操后填写）

```
（此处贴实际使用的 payload 与响应里回显的 HTML 片段）
```

- 回显上下文：`（文本节点 / 属性 / script 内）`
- 浏览器验证结果：`（填）`
- 卡住的地方与解法：`（填）`

## 四、防御视角

**按上下文做输出编码**：HTML 实体编码、属性编码、JavaScript 编码、URL 编码各自适用不同位置，不能一套全局 HTML 编码打天下（放进 JS 上下文照样能被闭合）。配合 CSP 做纵深防御，会话 Cookie 加 `HttpOnly` 降低被窃取风险。

几种常见的无效修法：

- 只做输入过滤 / 黑名单 —— `<img onerror>`、大小写混写、SVG、各种编码，绕过方式无穷无尽；
- 靠 WAF 拦 `<script>` 字符串 —— 事件型 payload 根本不含这个字符串；
- 把 CSP 当修复手段 —— CSP 是"万一 XSS 已经发生"的兜底，配 `script-src 'unsafe-inline'` 等于没配。

## 五、一句话复盘

"反射型 XSS 的本质是服务端把用户输入拼进响应 HTML，浏览器当成代码执行；利用时第一步是确定回显落在文本节点、属性还是 script 里，再选对应 payload，最后必须回浏览器验证。修复是按上下文做输出编码，输入过滤和 WAF 都只是补充。"
