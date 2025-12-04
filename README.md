<div align="center">

<a href="https://fastgpt.io/"><img src="/.github/imgs/logo.svg" width="120" height="120" alt="fastgpt logo"></a>

# FastGPT

![Qoute](./.github/imgs/image.png)

<p align="center">
  <a href="./README.md">简体中文</a> |
  <a href="./README_en.md">English</a> |
  <a href="./README_ja.md">日本語</a>
</p>

FastGPT is a knowledge-based question-answering system built on Large Language Models (LLMs) that offers out-of-the-box data processing, model invocation capabilities, and visual workflow orchestration through Flow. It enables you to easily develop and deploy complex question-answering systems without extensive setup or configuration.

[![GitHub Repo stars](https://img.shields.io/github/stars/labring/FastGPT?style=flat-square&labelColor=d4eaf7&color=7d09f1)](https://github.com/labring/FastGPT/stargazers)
[![GitHub pull request](https://img.shields.io/badge/PRs-welcome-fffff?style=flat-square&labelColor=d4eaf7&color=7d09f1)](https://github.com/labring/FastGPT/pulls)
[![GitHub last commit](https://img.shields.io/github/last-commit/labring/FastGPT?style=flat-square&labelColor=d4eaf7&color=7d09f1)](https://github.com/labring/FastGPT/pulls)
[![License](https://img.shields.io/badge/License-Apache--2.0-ffffff?style=flat-square&labelColor=d4eaf7&color=7d09f1)](https://github.com/labring/FastGPT/blob/main/LICENSE)
[![Documentation](https://img.shields.io/badge/Documentation-7d09f1?style=flat-square)](https://doc.fastgpt.io/docs/introduction)
[![Local Development](https://img.shields.io/badge/Local_Development-%23d4eaf7?style=flat-square&logo=xcode&logoColor=7d09f1)](https://doc.fastgpt.io/docs/introduction/development/intro)
[![Explore our platform](https://img.shields.io/badge/Explore_our_platform-d4eaf7?style=flat-square&logo=spoj&logoColor=7d09f1)](https://fastgpt.io/)

[![discord](https://theme.zdassets.com/theme_assets/678183/cc59daa07820943e943c2fc283b9079d7003ff76.svg)](https://discord.gg/mp68xkZn2Q)&nbsp;&nbsp;&nbsp;&nbsp;
[![Wechat](https://upload.wikimedia.org/wikipedia/en/thumb/a/af/WeChat_logo.svg/100px-WeChat_logo.svg.png?20231125073656)](https://oss.laf.run/otnvvf-imgs/feishu3.png)

</div>

## 🎥 Comprehensive Feature Demonstration

https://github.com/labring/FastGPT/assets/15308462/7d3a38df-eb0e-4388-9250-2409bd33f6d4

## 🛸 Online Use

Website: [fastgpt.io](https://fastgpt.io/)

| | |
| ---------------------------------- | ---------------------------------- |
|       Conversational AI Setup      |        Workflow Automation         |
| ![Demo](./.github/imgs/intro1.png) | ![Demo](./.github/imgs/intro2.png) |
|       Knowledge Base Setup         |        Integration Process         |
| ![Demo](./.github/imgs/intro3.png) | ![Demo](./.github/imgs/intro4.png) |

<a href="#FastGPT">
    <img src="https://img.shields.io/badge/-Back_to_Top-7d09f1.svg" alt="#" align="right">
</a>

## 💡 Features

| **Features**                               | **Details**                                       |
|--------------------------------------------|---------------------------------------------------|
| **Application Orchestration Features**   | ✅ **Simple Mode**: Streamlined interface eliminating complex orchestration needs <br> ✅ **Guided Workflows**: Clear step-by-step instructions in dialogues <br> ✅ **Visual Workflow Builder**: Drag-and-drop workflow orchestration <br> ✅ **Source Tracking**: Complete reference tracking in source files <br> ✅ **Module Reusability**: Multi-level encapsulation for enhanced reuse <br> ✅ **Search & Ranking**: Combined search and reordering functionality <br> 🔜 **Tool Module**: Comprehensive tool integration (coming soon) <br> 🔜 **Laf Integration**: Online HTTP module creation via [Laf](https://github.com/labring/laf) (coming soon) <br> 🔜 **Plugin System**: Advanced plugin encapsulation capabilities (coming soon) |
| **Knowledge Base Features**              | ✅ **Multi-Database Support**: Mixed use of multiple database types <br> ✅ **Change Tracking**: Complete tracking of modifications and deletions in data chunks <br> ✅ **Custom Vector Models**: Specific vector model selection per knowledge base <br> ✅ **Source File Storage**: Original file preservation and management <br> ✅ **Flexible Import**: Direct input and segment-based QA import <br> ✅ **Multi-Format Support**: PDF, DOCX, TXT, HTML, MD, CSV compatibility <br> ✅ **Bulk Operations**: URL reading and batch CSV importing <br> 🔜 **Extended Format Support**: PPT and Excel file import (coming soon) <br> 🔜 **File Reader**: Advanced file processing capabilities (coming soon) <br> 🔜 **Data Preprocessing**: Diverse data preprocessing options (coming soon) |
| **Application Debugging Features**        | ✅ **Targeted Testing**: Knowledge base search testing <br> ✅ **Interactive Editing**: Real-time feedback, editing, and deletion during conversations <br> ✅ **Full Context Display**: Complete interaction context visualization <br> ✅ **Module Transparency**: All intermediate values within modules <br> 🔜 **Advanced Debug Mode**: Enhanced debugging for orchestration (coming soon) |
| **OpenAPI Interface**                    | ✅ **Chat Completions**: GPT-compatible chat completions interface <br> ✅ **Knowledge Base CRUD**: Complete CRUD operations for knowledge bases <br> 🔜 **Conversation Management**: Full CRUD operations for conversations (coming soon) |
| **Operational Features**                   | ✅ **Public Sharing**: Share applications without login requirements <br> ✅ **Easy Embedding**: Simple iframe embedding capabilities <br> ✅ **Customizable Interface**: Chat window embedding with customizable features (default open, drag-and-drop) <br> ✅ **Conversation Analytics**: Centralized conversation records for review and annotation |

<a href="#FastGPT">
    <img src="https://img.shields.io/badge/-Back_to_Top-7d09f1.svg" alt="#" align="right">
</a>

## 🛠️ Technology Stack

**Core Technologies:**
- **Frontend**: Next.js 14 + TypeScript + Chakra UI
- **Backend**: Node.js + Express + MongoDB/Mongoose
- **Vector Databases**: PostgreSQL (with PG Vector) / Milvus
- **Package Manager**: pnpm workspaces (monorepo structure)

**Key Libraries & Frameworks:**
- **UI Components**: Chakra UI + Framer Motion
- **State Management**: React Query + Zustand
- **Database**: MongoDB + PostgreSQL/Milvus for vector storage
- **Workflow Engine**: Custom DAG execution engine
- **AI Integration**: Multiple LLM providers (OpenAI, Anthropic, local models)
- **Internationalization**: i18next (supports English, Chinese, Japanese)

**Monorepo Structure:**
- `packages/global/` - Shared types, constants, utilities
- `packages/service/` - Backend services, database models, API controllers
- `packages/web/` - Frontend components, hooks, styles, i18n
- `packages/templates/` - Application templates
- `projects/app/` - Main NextJS application
- `projects/sandbox/` - Code execution sandbox
- `projects/mcp_server/` - Model Context Protocol server

<a href="#FastGPT">
    <img src="https://img.shields.io/badge/-Back_to_Top-7d09f1.svg" alt="#" align="right">
</a>

## 👨‍💻 Development

### ⚡ One-Click Deployment

Deploy FastGPT instantly using [Sealos](https://sealos.io) cloud platform - no server or domain purchase required. Features high concurrency, dynamic scaling, and Kubeblocks database with IO performance that significantly exceeds basic Docker container deployments.

<div align="center">
[![](https://cdn.jsdelivr.net/gh/labring-actions/templates@main/Deploy-on-Sealos.svg)](https://cloud.sealos.io/?openapp=system-fastdeploy%3FtemplateName%3Dfastgpt&uid=fnWRt09fZP)
</div>

**Note**: Allow 2-4 minutes after deployment for database initialization. Initial performance may be slower due to basic settings configuration.

### 📚 Development Guides

- **[Getting Started with Local Development](https://doc.fastgpt.io/docs/introduction/development/intro)**
  - Prerequisites and environment setup
  - Local development server configuration
  - Database setup and migration

- **[Deploying FastGPT](https://doc.fastgpt.io/docs/introduction/development/docker)**
  - Docker deployment instructions
  - Environment configuration
  - Production deployment best practices

- **[System Configuration Guide](https://doc.fastgpt.io/docs/introduction/development/configuration)**
  - Configuration file structure
  - Environment variables
  - Feature flags and settings

- **[Multi-Model Configuration](https://doc.fastgpt.io/docs/introduction/development/modelConfig/intro)**
  - Supported AI providers
  - Model configuration examples
  - API key management

- **[Version Updates & Upgrades](https://doc.fastgpt.io/docs/introduction/development/upgrading/index)**
  - Migration guides
  - Breaking changes
  - Update procedures

### 🔧 Development Commands

```bash
# Install dependencies
pnpm install

# Start development environment (all projects)
pnpm dev

# Build all projects
pnpm build

# Run tests
pnpm test

# Run workflow-specific tests
pnpm test:workflow

# Code linting and formatting
pnpm lint
pnpm format-code
```

### 📁 Project Structure

```
FastGPT/
├── packages/           # Shared libraries
│   ├── global/        # Types, constants, utilities
│   ├── service/       # Backend services, database models
│   ├── web/          # Frontend components, hooks, styles
│   └── templates/    # Application templates
├── projects/          # Applications
│   ├── app/          # Main NextJS application
│   ├── sandbox/      # Code execution sandbox
│   ├── mcp_server/   # Model Context Protocol server
│   └── marketplace/  # Template marketplace
├── plugins/           # External plugins
├── deploy/           # Deployment configurations
├── document/         # Documentation
└── test/            # Test files and utilities
```

<a href="#FastGPT">
    <img src="https://img.shields.io/badge/-Back_to_Top-7d09f1.svg" alt="#" align="right">
</a>

## 💪 Related Projects

- **[Laf](https://github.com/labring/laf)**: 3-minute quick access to third-party applications
- **[Sealos](https://github.com/labring/sealos)**: Rapid deployment of cluster applications
- **[One API](https://github.com/songquanpeng/one-api)**: Multi-model management, supports Azure, Wenxin Yiyuan, etc.
- **[TuShan](https://github.com/msgbyte/tushan)**: Build a backend management system in 5 minutes
- **[FastGPT-plugin](https://github.com/labring/fastgpt-plugin)**: Plugin ecosystem for FastGPT

<a href="#FastGPT">
    <img src="https://img.shields.io/badge/-Back_to_Top-7d09f1.svg" alt="#" align="right">
</a>

## 🤝 Third-party Ecosystem

- **[luolinAI](https://github.com/luolin-ai/FastGPT-Enterprise-WeChatbot)**: Enterprise WeChat bot, ready to use
- **[PPIO 派欧云](https://ppinfra.com/user/register?invited_by=VITYVU&utm_source=github_fastgpt)**: One-click access to cost-effective open-source model APIs and GPU containers
- **[AI Proxy](https://sealos.run/aiproxy/?k=fastgpt-github/)**: Domestic model aggregation service
- **[SiliconCloud](https://cloud.siliconflow.cn/i/TR9Ym0c4)**: Open-source model online experience platform

<a href="#FastGPT">
    <img src="https://img.shields.io/badge/-Back_to_Top-7d09f1.svg" alt="#" align="right">
</a>

## 🏘️ Community & Support

+ 🌐 Visit the [FastGPT website](https://fastgpt.io/) for full documentation and useful links.
+ 💬 Join our [Discord server](https://discord.gg/mp68xkZn2Q) to chat with FastGPT developers and other FastGPT users. This is a good place to learn about FastGPT, ask questions, and share your experiences.
+ 🐞 Create [GitHub Issues](https://github.com/labring/FastGPT/issues/new/choose) for bug reports and feature requests.
+ 📧 Scan the QR code to join our Feishu community group:

  ![Feishu Group](https://oss.laf.run/otnvvf-imgs/fastgpt-feishu2.png)

<a href="#FastGPT">
    <img src="https://img.shields.io/badge/-Back_to_Top-7d09f1.svg" alt="#" align="right">
</a>

## 👀 Additional Resources

### 📖 Documentation & Tutorials
- **[FastGPT FAQ](https://kjqvjse66l.feishu.cn/docx/HtrgdT0pkonP4kxGx8qcu6XDnGh)** - Frequently asked questions
- **[Docker Deployment Tutorial Video](https://www.bilibili.com/video/BV1jo4y147fT/)** - Step-by-step deployment guide
- **[Official Account Integration Tutorial](https://www.bilibili.com/video/BV1xh4y1t7fy/)** - Integration with official accounts
- **[FastGPT Knowledge Base Demo](https://www.bilibili.com/video/BV1Wo4y1p7i1/)** - Knowledge base features demonstration

### 🔗 API Documentation
- **[OpenAPI Documentation](https://doc.fastgpt.io/docs/introduction/development/openapi/)** - Complete API reference
- **[Knowledge Base Structure](https://doc.fastgpt.io/docs/introduction/guide/knowledge_base/RAG/)** - Understanding RAG implementation

<a href="#FastGPT">
    <img src="https://img.shields.io/badge/-Back_to_Top-7d09f1.svg" alt="#" align="right">
</a>

## 🌱 Contributors

We welcome all forms of contributions! Whether you're fixing bugs, adding features, improving documentation, or sharing ideas, your participation helps make FastGPT better.

**How to contribute:**
- 🐛 **Report Issues**: Found a bug? [Create an issue](https://github.com/labring/FastGPT/issues/new/choose)
- 💡 **Feature Requests**: Have an idea? [Open a discussion](https://github.com/labring/FastGPT/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc)
- 🔧 **Code Contributions**: Ready to code? Check our open issues and submit pull requests
- 📚 **Documentation**: Help improve our docs and guides

Every contribution matters, and we appreciate your interest in making FastGPT even better!

### Development Guidelines
1. **Design First**: Create detailed design documents before implementation
2. **Test-Driven Development**: Write tests before implementing features
3. **Code Standards**: Follow ESLint and Prettier configurations
4. **Documentation**: Update documentation for new features
5. **Code Review**: All contributions require review before merging

<a href="https://github.com/labring/FastGPT/graphs/contributors" target="_blank">
  <table>
    <tr>
      <th colspan="2">
        <br><img src="https://contrib.rocks/image?repo=labring/FastGPT"><br><br>
      </th>
    </tr>
    <tr>
      <td>
        <picture>
          <source media="(prefers-color-scheme: dark)" srcset="https://next.ossinsight.io/widgets/official/compose-org-active-contributors/thumbnail.png?activity=active&period=past_28_days&owner_id=102226726&repo_ids=605673387&image_size=2x3&color_scheme=dark">
          <img alt="Active participants of labring - past 28 days" src="https://next.ossinsight.io/widgets/official/compose-org-active-contributors/thumbnail.png?activity=active&period=past_28_days&owner_id=102226726&repo_ids=605673387&image_size=2x3&color_scheme=light">
        </picture>
      </td>
      <td rowspan="2">
        <picture>
          <source media="(prefers-color-scheme: dark)" srcset="https://next.ossinsight.io/widgets/official/compose-org-participants-growth/thumbnail.png?activity=new&period=past_28_days&owner_id=102226726&repo_ids=605673387&image_size=4x7&color_scheme=dark">
          <img alt="New trends of labring" src="https://next.ossinsight.io/widgets/official/compose-org-participants-growth/thumbnail.png?activity=new&period=past_28_days&owner_id=102226726&repo_ids=605673387&image_size=4x7&color_scheme=light">
        </picture>
      </td>
    </tr>
    <tr>
      <td>
        <picture>
          <source media="(prefers-color-scheme: dark)" srcset="https://next.ossinsight.io/widgets/official/compose-org-active-contributors/thumbnail.png?activity=new&period=past_28_days&owner_id=102226726&repo_ids=605673387&image_size=2x3&color_scheme=dark">
          <img alt="New participants of labring - past 28 days" src="https://next.ossinsight.io/widgets/official/compose-org-active-contributors/thumbnail.png?activity=new&period=past_28_days&owner_id=102226726&repo_ids=605673387&image_size=2x3&color_scheme=light">
        </picture>
      </td>
    </tr>
  </table>
</a>

<a href="#FastGPT">
    <img src="https://img.shields.io/badge/-Back_to_Top-7d09f1.svg" alt="#" align="right">
</a>

## 🌟 Star History

<a href="https://github.com/labring/FastGPT/stargazers" target="_blank" style="display: block" align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=labring/FastGPT&type=Date&theme=dark" />
    <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=labring/FastGPT&type=Date" />
    <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=labring/FastGPT&type=Date" />
  </picture>
</a>

<a href="#FastGPT">
    <img src="https://img.shields.io/badge/-Back_to_Top-7d09f1.svg" alt="#" align="right">
</a>

## 📄 License

This repository is licensed under the [Apache 2.0 License](./LICENSE).

### Permitted Use:
- ✅ Direct commercial use as a backend service
- ✅ Academic and research purposes
- ✅ Personal and internal company use

### Restrictions:
- ❌ Provision of SaaS services is not permitted without commercial license
- ❌ Commercial services must retain copyright information
- ❌ Removing attribution or license notices

### Commercial Licensing:
For SaaS deployment or enterprise use cases, please contact us for commercial licensing options.

📧 **Contact:** Dennis@sealos.io
💼 [View Commercial Pricing](https://doc.fastgpt.io/docs/introduction/commercial/)
📖 [Read Full License](./LICENSE)

---

<div align="center">
  <p>Made with ❤️ by the FastGPT community</p>
  <p>If you find FastGPT helpful, please consider giving us a ⭐️</p>
</div>