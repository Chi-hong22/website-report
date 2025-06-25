# 池鸿的网页报告合集 🚀

一个专注于AI前沿技术研究的个人报告展示网站，集中展示具身智能、大模型、AUV等领域的深度调研成果。

![项目预览](https://img.shields.io/badge/Status-Active-brightgreen) ![技术栈](https://img.shields.io/badge/Tech-HTML%2FCSS%2FJS-blue) ![主题](https://img.shields.io/badge/Theme-AI%20Research-purple)

## 🌐 在线预览

**✨ [立即访问](https://www.yzarp.top/)** | 体验完整功能

## ✨ 功能特性

- 🎨 **现代化设计**：采用渐变背景和卡片式布局，支持浅色/深色主题切换
- 🔍 **智能搜索**：实时搜索项目名称、描述、类型和分类
- 📱 **响应式设计**：完美适配桌面、平板和移动设备
- 🌙 **主题切换**：支持浅色/深色模式，用户偏好本地存储
- 📊 **项目统计**：动态显示总项目数和活跃项目数
- ⚡ **性能优化**：轻量级实现，加载动画提升用户体验

## 📁 项目结构

```
website-report/
├── index.html                     # 主页面
├── README.md                      # 项目文档
├── EAI_in_AUV/                   # 具身智能在AUV中的应用
│   ├── index.html
│   └── EAI_in_AUV.md
├── Embodied_AI/                   # 具身智能研究
│   ├── index.html
│   └── attachments/               # 相关图片资源
├── LLM/                          # 大模型研究
│   ├── index.html
│   └── attachments/               # 相关图片资源
├── LNN/                          # 液态神经网络研究
│   ├── index.html
│   └── LNN.md
└── PDF/                          # 原始PDF报告
    ├── 具身智能 Embodied AI.pdf
    ├── 大模型 LLM.pdf
    └── 液态神经网络 LNN.pdf
```

## 🚀 快速开始

### 本地运行

1. **克隆项目**
   ```bash
   git clone https://github.com/Chi-hong22/250510_website-report.git
   cd 250510_website-report
   ```

2. **启动本地服务器**
   ```bash
   # 使用Python (推荐)
   python -m http.server 8080
   
   # 或使用Node.js
   npx serve .
   
   # 或使用其他HTTP服务器
   ```

3. **访问网站**
   打开浏览器访问 `http://localhost:8080`

### 在线访问

🌍 **已部署版本**：[https://yzarp.top](https://yzarp.top)

### 部署到GitHub Pages

1. Fork本项目到你的GitHub账户
2. 在项目设置中启用GitHub Pages
3. 选择`main`分支作为发布源
4. 访问 `https://你的用户名.github.io/250510_website-report`

## 💻 技术栈

- **前端框架**：原生HTML5 + CSS3 + JavaScript (ES6+)
- **样式特性**：
  - CSS自定义属性(CSS Variables)
  - Flexbox和Grid布局
  - CSS动画和过渡效果
  - 响应式媒体查询
- **功能实现**：
  - 模块化JavaScript类设计
  - 本地存储API
  - 事件驱动编程
  - 动态DOM操作

## 📝 添加新项目

如果你想添加新的研究报告，可以按以下步骤操作：

1. **创建项目目录**
   ```bash
   mkdir 新项目名称
   cd 新项目名称
   ```

2. **添加项目文件**
   - 创建`index.html`作为项目主页
   - 添加相关的资源文件（图片、PDF等）

3. **更新主页配置**
   在`index.html`中的`projectsConfig`数组中添加新项目：
   ```javascript
   {
       name: '项目名称',
       description: '项目描述',
       path: '项目目录/index.html',
       type: '项目类型',
       status: 'Active',
       icon: '📊',
       category: '分类'
   }
   ```

## 🎯 项目亮点

### 1. 研究领域覆盖
- **具身智能 (Embodied AI)**：机器人感知、决策和行动集成
- **大模型 (LLM)**：语言模型架构、训练技术和应用
- **液态神经网络 (LNN)**：新兴AI架构的深度探索
- **AUV应用**：水下环境中的AI导航和SLAM技术

### 2. 技术实现特色
- 🏗️ **模块化架构**：ProjectNavigator类封装所有功能
- 🎨 **主题系统**：完整的明暗主题切换机制
- 🔄 **状态管理**：本地存储用户偏好设置
- 📈 **渐进增强**：从基础功能到高级特性逐步构建

## 🤝 贡献指南

欢迎对项目进行改进和扩展！

### 贡献方式
1. Fork本项目
2. 创建特性分支 (`git checkout -b feature/新功能`)
3. 提交改动 (`git commit -am '添加新功能'`)
4. 推送分支 (`git push origin feature/新功能`)
5. 创建Pull Request

### 代码规范
- 使用小驼峰命名法命名函数
- 使用蛇形命名法命名变量
- 常量使用全大写蛇形命名法
- 保持代码简洁性和可读性

## 📄 许可证

本项目采用MIT许可证 - 查看[LICENSE](LICENSE)文件了解详情

## 👨‍💻 作者信息

**池鸿 (Chi-hong)**
- GitHub: [@Chi-hong22](https://github.com/Chi-hong22)
- 专注领域：具身智能, AUV, SLAM, 大模型

## 🙏 致谢

感谢所有为AI技术发展做出贡献的研究者和开发者，本项目的研究内容基于众多优秀的学术论文和开源项目。

---

如果这个项目对你有帮助，请给个⭐️支持一下！ 