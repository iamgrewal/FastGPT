<div align="center">

<a href="https://fastgpt.io/"><img src="/.github/imgs/logo.svg" width="120" height="120" alt="fastgpt logo"></a>

# FastGPT

![Qoute](./.github/imgs/image.png)

<p align="center">
  <a href="./README_en.md">English</a> |
  <a href="./README.md">简体中文</a> |
  <a href="./README_ja.md">日语</a>
</p>

FastGPT is a knowledge-based platform built on Large Language Models (LLMs) that offers a comprehensive suite of out-of-the-box capabilities including data processing, RAG retrieval, and visual AI workflow orchestration. It enables you to easily develop and deploy complex question-answering systems without extensive setup or configuration. 

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

## 👨‍💻 Development

Project tech stack: NextJs + TS + ChakraUI + MongoDB + PostgreSQL (PG Vector plug-in)/Milvus

- **⚡ One-Click Deployment**

  > Deploy FastGPT instantly using [Sealos](https://sealos.io) cloud platform - no server or domain purchase required. Features high concurrency, dynamic scaling, and Kubeblocks database with IO performance that significantly exceeds basic Docker container deployments.
  <div align="center">
  [![](https://cdn.jsdelivr.net/gh/labring-actions/templates@main/Deploy-on-Sealos.svg)](https://cloud.sealos.io/?openapp=system-fastdeploy%3FtemplateName%3Dfastgpt&uid=fnWRt09fZP)
  </div>

  **Note**: Allow 2-4 minutes after deployment for database initialization. Initial performance may be slower due to basic settings configuration.

  [sealos one click deployment tutorial](https://doc.fastgpt.io/docs/introduction/development/sealos/)

- [Getting Started with Local Development](https://doc.fastgpt.io/docs/introduction/development/intro)
- [Deploying FastGPT](https://doc.fastgpt.io/docs/introduction/development/docker)
- [Guide on System Configs](https://doc.fastgpt.io/docs/introduction/development/configuration)
- [Configuring Multiple Models](https://doc.fastgpt.io/docs//introduction/development/modelConfig/intro)
- [Version Updates & Upgrades](https://doc.fastgpt.io/docs/introduction/development/upgrading/index)

<a href="#FastGPT">
    <img src="https://img.shields.io/badge/-Back_to_Top-7d09f1.svg" alt="#" align="right">
</a>

## 💪 Related Projects

- [Laf: 3-minute quick access to third-party applications](https://github.com/labring/laf)
- [Sealos: Rapid deployment of cluster applications](https://github.com/labring/sealos)
- [One API: Multi-model management, supports Azure, Wenxin Yiyuan, etc.](https://github.com/songquanpeng/one-api)
- [TuShan: Build a backend management system in 5 minutes](https://github.com/msgbyte/tushan)

<a href="#FastGPT">
    <img src="https://img.shields.io/badge/-Back_to_Top-7d09f1.svg" alt="#" align="right">
</a>

## 🤝 Third-party Ecosystem

- [luolinAI: Enterprise WeChat bot, ready to use](https://github.com/luolin-ai/FastGPT-Enterprise-WeChatbot)

<a href="#FastGPT">
    <img src="https://img.shields.io/badge/-Back_to_Top-7d09f1.svg" alt="#" align="right">
</a>


## 🏘️ Community & Support

+ 🌐 Visit the [FastGPT website](https://fastgpt.io/) for full documentation and useful links.
+ 💬 Join our [Discord server](https://discord.gg/mp68xkZn2Q) is to chat with FastGPT developers and other FastGPT users. This is a good place to learn about FastGPT, ask questions, and share your experiences.
+ 🐞 Create [GitHub Issues](https://github.com/labring/FastGPT/issues/new/choose) for bug reports and feature requests.

<a href="#FastGPT">
    <img src="https://img.shields.io/badge/-Back_to_Top-7d09f1.svg" alt="#" align="right">
</a>

## 👀 Others

- [FastGPT FAQ](https://kjqvjse66l.feishu.cn/docx/HtrgdT0pkonP4kxGx8qcu6XDnGh)
- [Docker Deployment Tutorial Video](https://www.bilibili.com/video/BV1jo4y147fT/)
- [Official Account Integration Video Tutorial](https://www.bilibili.com/video/BV1xh4y1t7fy/)
- [FastGPT Knowledge Base Demo](https://www.bilibili.com/video/BV1Wo4y1p7i1/)

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

## 📄 Usage Agreement

This repository is licensed under the [FastGPT Open Source License](./LICENSE).

## 📄 License Terms

**Permitted Use:**
- ✅ Direct commercial use as a backend service
- ✅ Academic and research purposes
- ✅ Personal and internal company use

**Restrictions:**
- ❌ Provision of SaaS services is not permitted without commercial license
- ❌ Commercial services must retain copyright information
- ❌ Removing attribution or license notices

**Commercial Licensing:**
For SaaS deployment or enterprise use cases, please contact us for commercial licensing options.

📧 **Contact:** Dennis@sealos.io
💼 [View Commercial Pricing](https://doc.fastgpt.io/docs/introduction/commercial/)

📖 [Read Full License](./LICENSE)

<a href="#FastGPT">
    <img src="https://img.shields.io/badge/-Back_to_Top-7d09f1.svg" alt="#" align="right">
</a>