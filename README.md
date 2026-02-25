# 💼 职业问答社区 MVP

一个轻量级的职业规划问答社区，支持提问、回答和模拟付费功能。

## 功能

- ✅ 问答列表浏览
- ✅ 发起提问
- ✅ 查看问题详情
- ✅ 回答问题
- ✅ 模拟付费回答
- ✅ 标签分类
- ✅ 搜索功能
- ✅ localStorage 数据持久化

## 技术栈

- Vue 3 + Vite
- Tailwind CSS v4
- localStorage（模拟后端）

## 本地开发

```bash
npm install
npm run dev
```

## 构建部署

```bash
npm run build
```

构建产物在 `dist/` 目录。

## 部署到 GitHub Pages

1. 在 GitHub 创建新仓库（如 `career-qa`）
2. 推送代码：
   ```bash
   git remote add origin https://github.com/你的用户名/career-qa.git
   git branch -M main
   git push -u origin main
   ```
3. 在仓库 Settings → Pages → Source 选择 `main` 分支
4. 访问 `https://你的用户名.github.io/career-qa/`

## 后续迭代

- [ ] 接入真实后端 API
- [ ] 用户登录/注册
- [ ] 真实支付功能
- [ ] 通知系统
- [ ] 问题收藏
