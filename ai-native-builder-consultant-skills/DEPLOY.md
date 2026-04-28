# 部署指南

把这套 Skills 库 + 三个前端界面部署到你自己的 GitHub Pages，让任何人都能在线访问。

---

## 5 分钟快速部署

### 步骤 1：创建 GitHub 仓库

```bash
# 在 GitHub 上创建一个空仓库，比如：
# https://github.com/MedocMay/ai-native-builder-consultant-skills

# 本地解压发布包
unzip ai-native-builder-consultant-skills-v3.1.zip
cd ai-native-builder-consultant-skills

# 初始化 git，关联远端
git init
git add .
git commit -m "Initial release: AI Native Builder Consultant Skills v3.1"
git branch -M main
git remote add origin https://github.com/MedocMay/ai-native-builder-consultant-skills.git
git push -u origin main
```

### 步骤 2：开启 GitHub Pages

在仓库主页：

1. 点击 **Settings** → 左侧边栏 **Pages**
2. **Source** 选 `Deploy from a branch`
3. **Branch** 选 `main` · 文件夹选 `/docs`
4. 点 **Save**

等 1-2 分钟，GitHub 会显示部署地址：

```
https://medocmay.github.io/ai-native-builder-consultant-skills/
```

打开这个地址就能看到入口页（`docs/index.html`）。

### 步骤 3：自定义你的信息

编辑这几个文件，把示例信息换成你自己的：

**`docs/index.html`** —— 把 GitHub 链接、品牌名换成你的：
```html
<a href="https://github.com/MedocMay/ai-native-builder-consultant-skills" target="_blank">GitHub ↗</a>
```

**`README.md`** —— 第一行的项目介绍可以个性化

**`docs/chat.html`** —— 默认的 4 个 a16z 场景可以替换为你自己关注的场景：
```html
<button class="starter" data-text="...">
  <div class="starter-tag">▸ YOUR TAG</div>
  <div class="starter-text">你的场景标题</div>
</button>
```

---

## 三个界面的关系

| 路径 | 适合谁 | 用途 |
|------|--------|------|
| `/` (index.html) | 第一次访问的人 | 选择入口 |
| `/chat.html` | 想快速体验咨询的潜在客户 | 跑完 4 步咨询 → 悬赏支付 |
| `/explore.html` | 想学习/参考的工程师/PM | 浏览全部 skills 内容 |
| `/workbench.html` | 咨询师日常使用 | 类 Claude Code 桌面版工作台 |

---

## 关于 Live API 模式

`chat.html` 提供 DEMO（脚本演示）和 LIVE API（真实 Claude）两种模式。

### LIVE API 模式工作原理

1. 用户在浏览器里输入自己的 Anthropic API Key
2. Key 只保存在 **浏览器 localStorage**，不会上传任何服务器
3. fetch 请求**直接从浏览器发到** `api.anthropic.com`，附加 `anthropic-dangerous-direct-browser-access: true` header
4. Skills 内容作为 system prompt 注入（按当前 SOP 阶段选择 3-5 个相关 skills）

### ⚠ 重要安全说明

**这种"浏览器直连 API"的方式适合个人测试和团队内部使用，但不适合作为公开服务部署。** 原因：

- 用户必须有自己的 API Key（你不能替他们出钱）
- 浏览器直连模式被 Anthropic 标记为 dangerous
- 如果你想替用户出 API 费用，必须用自己的后端代理调用（用 Vercel Edge Function / Cloudflare Workers / 自己的 Node 服务都可以）

### 升级为后端代理（可选）

如果你要为公开用户提供服务，参考这个最小代理：

```javascript
// vercel/api/claude.js (Vercel Serverless Function)
export default async function handler(req, res) {
  const response = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': process.env.ANTHROPIC_API_KEY, // 你的 key 在 Vercel env vars 里
      'anthropic-version': '2023-06-01',
    },
    body: JSON.stringify(req.body),
  });
  res.status(response.status);
  // 流式转发
  for await (const chunk of response.body) res.write(chunk);
  res.end();
}
```

然后改 `chat.html` 里的 fetch URL 从 `https://api.anthropic.com/v1/messages` 改成 `/api/claude`，并删掉 `x-api-key` header 与配置 UI。

---

## 自定义 Skills

### 添加你自己的 skill

1. 在对应分类目录下创建文件夹，比如 `skills/agent-design/my-new-skill/`
2. 创建 `SKILL.md`，参考 [CONTRIBUTING.md](CONTRIBUTING.md) 的格式规范
3. 在 `CONSULTING-WORKFLOW.md` 里声明它在 SOP 哪一步使用
4. 重新生成 `chat.html` 中嵌入的 skills 数据（需要重跑数据嵌入脚本，或手工编辑 base64 数据 — 见下方）

### 重新嵌入 skills 数据到 HTML

`docs/chat.html` 把所有 28 个 skills 内容 base64 编码后嵌入了 HTML 里（约 437KB）。如果你修改/新增了 skills，需要重新嵌入。

简化版本：直接编辑 chat.html，搜索 `const SKILLS_B64 = `，可以看到那一长串 base64。用这段 Python 脚本重新生成：

```python
import os, json, re, base64

skills = []
for category in os.listdir('skills'):
    cat_dir = f'skills/{category}'
    if not os.path.isdir(cat_dir): continue
    for skill_name in os.listdir(cat_dir):
        skill_path = f'{cat_dir}/{skill_name}/SKILL.md'
        if not os.path.isfile(skill_path): continue
        with open(skill_path) as f: content = f.read()
        fm = re.match(r'^---\n(.*?)\n---\n(.*)$', content, re.DOTALL)
        if not fm: continue
        desc = re.search(r'description:\s*(.+?)$', fm.group(1), re.MULTILINE | re.DOTALL)
        skills.append({
            'id': skill_name,
            'name': skill_name,
            'display_name': skill_name,  # 你可能想改成中文显示名
            'category': category,
            'description': desc.group(1).strip() if desc else '',
            'lines': len(content.splitlines()),
            'body': fm.group(2),
        })

skills_b64 = base64.b64encode(json.dumps(skills, ensure_ascii=False).encode('utf-8')).decode('ascii')

with open('docs/chat.html', 'r') as f: html = f.read()
html = re.sub(r"const SKILLS_B64 = '[^']+'", f"const SKILLS_B64 = '{skills_b64}'", html)
with open('docs/chat.html', 'w') as f: f.write(html)
print('Done')
```

---

## 自定义品牌

如果你想把这套界面用作自己公司的产品，至少改这些：

- **`docs/index.html`** —— logo 文字、配色、tagline
- **`docs/chat.html`** —— titlebar 里的"AI Native / Builder · 快速咨询"
- **`docs/assets/og-image.svg`** —— 社交媒体分享图
- **CSS 变量** —— 三个 HTML 文件顶部都有 `:root { --amber: ...; }`，全局换色只改这里

---

## 常见问题

**Q：用户看到的页面会被搜索引擎索引吗？**
A：会。GitHub Pages 默认允许爬虫。如果不想被索引，在 `docs/` 下加一个 `robots.txt`：
```
User-agent: *
Disallow: /
```

**Q：可以加个自定义域名吗？**
A：可以。在 `docs/` 下加 `CNAME` 文件写你的域名，然后在 DNS 服务商那里加 CNAME 记录指到 `medocmay.github.io`。GitHub Pages 设置里也要填一遍。

**Q：API key 真的安全吗？**
A：在 LIVE 模式下，key 只存在用户**自己的浏览器 localStorage** 里，不会传给任何服务器（除了 Anthropic 自己）。但**用户的浏览器里能看到 fetch 调用，所以 key 在 DevTools Network tab 是可见的**——这是为什么不要用作公开服务。个人/团队内部用没问题。

**Q：DEMO 模式的对话流是写死的吗？**
A：是。DEMO 模式有 4 步脚本化对话（场景分析 → 伪需求检查 → 项目定义 → Agent 边界），用户不管输入什么，AI 都按这 4 步走。这是为了让没有 API Key 的人也能完整看到咨询流程长什么样，决定要不要付费/接入真实 API。

**Q：怎么把这个嵌入到 Notion / 飞书 / 微信公众号？**
A：iframe 嵌入即可：
```html
<iframe src="https://medocmay.github.io/ai-native-builder-consultant-skills/chat.html"
        width="100%" height="800" frameborder="0"></iframe>
```

---

## 反馈与贡献

部署遇到问题？发现 bug？想贡献新的 skill？

- Issue: `https://github.com/MedocMay/ai-native-builder-consultant-skills/issues`
- 详细贡献规范见 [CONTRIBUTING.md](CONTRIBUTING.md)

_Last updated: 2026-04 · v3.1_
