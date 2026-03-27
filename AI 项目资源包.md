# AI 项目资源包

> 5 个适合写进简历的 AI 项目，从易到难，每个都能帮你拿到面试。

---

## 项目 1：AI 日报生成器 ⭐⭐

**难度：** ⭐⭐  
**技术栈：** Python + OpenAI API + 飞书/钉钉机器人  
**开发时间：** 1-2 天

### 项目描述
自动聚合 GitHub Trending、Hacker News、技术博客，用 AI 生成每日技术简报，推送到飞书/钉钉群。

### 核心代码

```python
# main.py
import requests
import openai
from datetime import datetime

# 1. 获取 GitHub Trending
def get_github_trending():
    url = "https://api.github.com/search/repositories"
    params = {"q": "stars:>1000", "sort": "stars", "order": "desc"}
    response = requests.get(url, params=params)
    return response.json()["items"][:5]

# 2. AI 摘要生成
def generate_summary(items):
    prompt = """
    你是技术编辑，请将以下内容整理成技术简报：
    - 提取关键技术点
    - 按「趋势」「工具」「文章」分类
    - 每项不超过 50 字
    
    内容：{content}
    """
    content = "\n".join([f"- {item['name']}: {item['description']}" for item in items])
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[{"role": "user", "content": prompt.format(content=content)}]
    )
    return response.choices[0].message.content

# 3. 发送到飞书
def send_to_feishu(webhook_url, content):
    payload = {
        "msg_type": "text",
        "content": {"text": f"📰 技术日报 {datetime.now().strftime('%Y-%m-%d')}\n\n{content}"}
    }
    requests.post(webhook_url, json=payload)

# 主函数
if __name__ == "__main__":
    items = get_github_trending()
    summary = generate_summary(items)
    send_to_feishu("YOUR_WEBHOOK_URL", summary)
```

### Prompt 模板

```
你是技术编辑，请将以下内容整理成技术简报：

【要求】
1. 提取关键技术点
2. 按「趋势」「工具」「文章」三类组织
3. 每项不超过 50 字
4. 用 emoji 增加可读性

【内容】
{content}

【输出格式】
📈 技术趋势
- ...

🛠️ 工具更新
- ...

📖 深度文章
- ...
```

### 部署方式

```bash
# 1. 安装依赖
pip install requests openai

# 2. 配置环境变量
export OPENAI_API_KEY="sk-xxx"
export FEISHU_WEBHOOK="https://open.feishu.cn/open-apis/bot/v2/hook/xxx"

# 3. 设置定时任务（每天早 8 点）
crontab -e
0 8 * * * cd /path/to/project && python main.py
```

---

## 项目 2：前端组件文档助手 ⭐⭐⭐

**难度：** ⭐⭐⭐  
**技术栈：** Node.js + Vue/React + OpenAI API  
**开发时间：** 3-5 天

### 项目描述
输入组件代码，自动生成文档（Props 说明、使用示例、注意事项），支持一键导出 Markdown。

### 核心代码

```javascript
// generate-docs.js
const fs = require('fs');
const OpenAI = require('openai');

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

// 解析 Vue 组件 Props
function parseVueProps(code) {
    const propsMatch = code.match(/props:\s*{([^}]+)}/s);
    if (!propsMatch) return [];
    
    const props = [];
    const propLines = propsMatch[1].split('\n');
    propLines.forEach(line => {
        const match = line.match(/(\w+):\s*{?\s*type:\s*(\w+)?/);
        if (match) {
            props.push({ name: match[1], type: match[2] || 'Any' });
        }
    });
    return props;
}

// AI 生成文档
async function generateDoc(componentName, props) {
    const prompt = `
    你是技术文档工程师，请为以下 Vue 组件生成文档：
    
    组件名：${componentName}
    Props:
    ${props.map(p => `- ${p.name} (${p.type})`).join('\n')}
    
    请生成：
    1. 组件功能描述（50 字内）
    2. Props 表格（名称 | 类型 | 说明 | 默认值）
    3. 使用示例代码
    4. 注意事项
    `;
    
    const response = await openai.chat.completions.create({
        model: "gpt-3.5-turbo",
        messages: [{ role: "user", content: prompt }]
    });
    
    return response.choices[0].message.content;
}

// 主函数
async function main() {
    const code = fs.readFileSync('./MyComponent.vue', 'utf-8');
    const props = parseVueProps(code);
    const doc = await generateDoc('MyComponent', props);
    fs.writeFileSync('./MyComponent.md', doc);
    console.log('文档生成完成！');
}

main();
```

### Prompt 模板

```
你是技术文档工程师，请为以下组件生成文档：

【组件信息】
- 组件名：{componentName}
- 技术栈：Vue 3 / React
- Props: {propsList}

【输出要求】
1. 功能描述（50 字内，说明组件用途）
2. Props 表格（Markdown 格式）
3. 使用示例（完整代码，可复制运行）
4. 注意事项（边界情况、性能提示）

【输出格式】
## 功能描述
...

## Props

| 名称 | 类型 | 说明 | 默认值 |
|------|------|------|--------|
| ... | ... | ... | ... |

## 使用示例
\`\`\`vue
<template>
  <MyComponent prop1="value" />
</template>
\`\`\`

## 注意事项
- ...
```

### 使用方式

```bash
# 1. 安装依赖
npm install openai

# 2. 运行
node generate-docs.js

# 3. 输出
# 文档生成完成！ -> MyComponent.md
```

---

## 项目 3：AI 代码 Review 机器人 ⭐⭐⭐⭐

**难度：** ⭐⭐⭐⭐  
**技术栈：** Node.js + GitHub Actions + OpenAI API  
**开发时间：** 5-7 天

### 项目描述
集成到 GitHub PR 流程，自动 Review 代码变更，给出改进建议（性能、安全、代码规范）。

### 核心代码

```yaml
# .github/workflows/ai-review.yml
name: AI Code Review
on: [pull_request]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Get PR Diff
        id: diff
        uses: actions/github-script@v6
        with:
          script: |
            const { data: pr } = await github.rest.pulls.get({
              owner: context.repo.owner,
              repo: context.repo.repo,
              pull_number: context.issue.number
            });
            const diff = pr.diff_url;
            console.log(diff);
      
      - name: AI Review
        uses: actions/github-script@v6
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
        with:
          script: |
            const OpenAI = require('openai');
            const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });
            
            const prompt = `
            你是一位资深前端工程师，请 Review 以下代码变更：
            1. 指出潜在的性能问题
            2. 检查安全隐患（XSS、SQL 注入等）
            3. 代码规范建议（命名、结构、注释）
            4. 给出具体修改建议（带代码示例）
            
            变更内容：${{ steps.diff.outputs.result }}
            `;
            
            const response = await openai.chat.completions.create({
              model: "gpt-4",
              messages: [{ role: "user", content: prompt }]
            });
            
            await github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: `## 🤖 AI Code Review\n\n${response.choices[0].message.content}`
            });
```

### Prompt 模板

```
你是一位资深前端工程师，请 Review 以下代码变更：

【审查维度】
1. 性能问题（循环、DOM 操作、异步处理）
2. 安全隐患（XSS、SQL 注入、敏感信息泄露）
3. 代码规范（命名、结构、注释、类型安全）
4. 可维护性（函数复杂度、重复代码）

【输出格式】
## ⚠️ 发现问题

### 性能
- [问题描述]
  ```diff
  - 原代码
  + 建议修改
  ```

### 安全
- [问题描述]

### 规范
- [问题描述]

## ✅ 亮点
- [值得肯定的地方]
```

### 部署方式

```bash
# 1. 创建 GitHub Actions 工作流
# 将上述 YAML 保存到 .github/workflows/ai-review.yml

# 2. 配置 Secret
# Settings → Secrets and variables → Actions
# 添加 OPENAI_API_KEY

# 3. 测试
# 创建一个 PR，观察是否自动评论
```

---

## 项目 4：智能简历优化器 ⭐⭐⭐⭐

**难度：** ⭐⭐⭐⭐  
**技术栈：** Next.js + pdf-parse + OpenAI API  
**开发时间：** 7-10 天

### 项目描述
上传简历 PDF，AI 分析内容并给出优化建议（措辞、结构、关键词），支持一键生成优化版本。

### 核心代码

```javascript
// pages/api/optimize.js
import pdfParse from 'pdf-parse';
import OpenAI from 'openai';

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

export default async function handler(req, res) {
    if (req.method !== 'POST') return res.status(405).end();
    
    // 1. 解析 PDF
    const pdfBuffer = req.body.pdf;
    const pdfData = await pdfParse(pdfBuffer);
    const resumeText = pdfData.text;
    
    // 2. AI 分析
    const prompt = `
    你是资深 HR 和简历顾问，请分析这份简历：
    
    1. 措辞优化（将「负责」改为「主导」「实现」等主动词）
    2. 量化建议（添加数据支撑，如「性能提升 40%」）
    3. 结构建议（哪些内容应该前置/删除）
    4. 关键词匹配（针对前端工程师岗位）
    
    简历内容：
    ${resumeText}
    
    输出格式：
    ## 整体评价
    ## 措辞优化建议
    ## 量化建议
    ## 结构优化
    ## 修改后版本
    `;
    
    const response = await openai.chat.completions.create({
        model: "gpt-4",
        messages: [{ role: "user", content: prompt }]
    });
    
    res.json({ suggestions: response.choices[0].message.content });
}
```

### Prompt 模板

```
你是资深 HR 和简历顾问，请分析这份简历：

【分析维度】
1. 措辞优化
   - 将被动描述改为主动成就（「负责」→「主导」「实现」）
   - 删除空洞词汇（「熟悉」「了解」→ 具体项目证明）

2. 量化建议
   - 找出可以添加数据的地方
   - 建议具体指标（性能提升%、用户增长数、节省时间）

3. 结构优化
   - 哪些内容应该前置（最亮眼的成就）
   - 哪些内容可以删除（无关经历）

4. 关键词匹配
   - 针对目标岗位 JD 提取关键词
   - 检查简历是否覆盖

【输出格式】
## 整体评价
（50 字内总结）

## 措辞优化
| 原文 | 建议修改 |
|------|----------|
| ... | ... |

## 量化建议
- 「优化了系统性能」→ 「将 API 响应时间从 500ms 降至 150ms，提升 70%」
- ...

## 修改后版本
（完整重写版本）
```

### 前端示例

```jsx
// pages/index.js
import { useState } from 'react';

export default function Home() {
    const [file, setFile] = useState(null);
    const [result, setResult] = useState(null);
    
    const handleUpload = async () => {
        const reader = new FileReader();
        reader.onload = async (e) => {
            const response = await fetch('/api/optimize', {
                method: 'POST',
                body: JSON.stringify({ pdf: e.target.result })
            });
            const data = await response.json();
            setResult(data.suggestions);
        };
        reader.readAsArrayBuffer(file);
    };
    
    return (
        <div>
            <input type="file" onChange={e => setFile(e.target.files[0])} />
            <button onClick={handleUpload}>开始优化</button>
            {result && <div dangerouslySetInnerHTML={{ __html: result }} />}
        </div>
    );
}
```

---

## 项目 5：AI 学习路径生成器 ⭐⭐⭐⭐⭐

**难度：** ⭐⭐⭐⭐⭐  
**技术栈：** Next.js + Pinecone/Chroma + OpenAI API + RAG  
**开发时间：** 10-15 天

### 项目描述
输入学习目标（如「3 个月转型全栈」），AI 生成个性化学习路径（每日任务、资源推荐、项目实战）。

### 核心代码

```javascript
// pages/api/generate-path.js
import OpenAI from 'openai';
import { ChromaClient } from 'chromadb';

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });
const chroma = new ChromaClient({ path: 'http://localhost:8000' });

export default async function handler(req, res) {
    const { goal, currentLevel, timeAvailable } = req.body;
    
    // 1. 查询向量数据库（技术栈知识）
    const collection = await chroma.getCollection({ name: 'tech-stack' });
    const queryResults = await collection.query({
        queryTexts: [goal],
        nResults: 10
    });
    
    // 2. AI 生成学习路径
    const prompt = `
    用户目标：${goal}
    当前水平：${currentLevel}
    可用时间：${timeAvailable}
    
    参考资源：
    ${queryResults.documents.map(d => d.join('\n')).join('\n')}
    
    请生成：
    1. 按周拆解的学习任务
    2. 推荐学习资源（文档、视频、文章）
    3. 实战项目（与目标匹配）
    4. 每周自测题目
    
    输出格式：
    ## 第 1 周：基础
    - 学习任务
    - 资源推荐
    - 实战项目
    
    ## 第 2 周：进阶
    ...
    `;
    
    const response = await openai.chat.completions.create({
        model: "gpt-4",
        messages: [{ role: "user", content: prompt }]
    });
    
    res.json({ path: response.choices[0].message.content });
}
```

### Prompt 模板

```
你是资深技术导师，请为用户生成个性化学习路径：

【用户信息】
- 目标：{goal}
- 当前水平：{currentLevel}（如：前端 2 年，会 Vue/React）
- 可用时间：{timeAvailable}（如：3 个月，每天 2 小时）

【生成要求】
1. 按周拆解任务（共{weeks}周）
2. 每周包含：
   - 学习任务（具体知识点）
   - 资源推荐（官方文档 + 视频 + 文章，带链接）
   - 实战项目（可写进简历）
   - 自测题目（5 道，检验掌握程度）
3. 难度递进（基础→进阶→实战）
4. 可执行（每天 2 小时内完成）

【输出格式】
## 第 1 周：{主题}
### 学习任务
- ...

### 资源推荐
- [官方文档] 链接
- [视频教程] 链接
- [深度文章] 链接

### 实战项目
- 项目名称 + 描述

### 自测
1. ...
2. ...
```

### 向量数据库准备

```javascript
// scripts/seed-knowledge.js
const { ChromaClient } = require('chromadb');

const techKnowledge = [
    {
        id: 'nodejs-basics',
        text: 'Node.js 基础：事件循环、模块系统、文件系统、HTTP 服务器',
        metadata: { category: 'backend', level: 'beginner' }
    },
    {
        id: 'database-sql',
        text: 'SQL 基础：SELECT/INSERT/UPDATE/DELETE、JOIN、索引、事务',
        metadata: { category: 'database', level: 'beginner' }
    },
    // ... 更多知识点
];

async function seed() {
    const chroma = new ChromaClient({ path: 'http://localhost:8000' });
    const collection = await chroma.createCollection({ name: 'tech-stack' });
    
    await collection.add({
        ids: techKnowledge.map(k => k.id),
        documents: techKnowledge.map(k => k.text),
        metadatas: techKnowledge.map(k => k.metadata)
    });
    
    console.log('知识库存入完成！');
}

seed();
```

---

## 📥 使用指南

### 1. 环境准备

```bash
# Node.js 项目
npm install openai

# Python 项目
pip install openai requests

# 向量数据库（项目 5）
docker run -p 8000:8000 chromadb/chroma
```

### 2. API Key 配置

```bash
# 阿里云百炼（推荐）
export OPENAI_API_KEY="sk-sp-xxx"
export OPENAI_BASE_URL="https://dashscope.aliyuncs.com/compatible-mode/v1"

# 或 OpenAI 官方
export OPENAI_API_KEY="sk-xxx"
```

### 3. 成本估算

| 项目 | 调用次数 | 预估成本（元） |
|------|----------|----------------|
| AI 日报生成器 | 30 次/月 | ~5 元 |
| 组件文档助手 | 10 次 | ~2 元 |
| 代码 Review 机器人 | 50 次/月 | ~10 元 |
| 简历优化器 | 10 次 | ~5 元 |
| 学习路径生成器 | 5 次 | ~3 元 |

**总计：** 约 25 元/月（个人使用）

---

## 📚 扩展阅读

- [OpenAI API 文档](https://platform.openai.com/docs)
- [阿里云百炼文档](https://help.aliyun.com/zh/dashscope/)
- [GitHub Actions 官方文档](https://docs.github.com/en/actions)
- [Chroma 向量数据库](https://docs.trychroma.com/)

---

## 💬 问题反馈

如果遇到问题，欢迎在公众号后台留言「AI 项目 + 问题描述」。

**持续更新中...** 后续会补充：
- 完整代码仓库（GitHub 链接）
- 视频教程
- 读者项目展示

---

*最后更新：2026-03-27*  
*资源包版本：v1.0（快速版）*
