# FastGPT Documentation

This is the official FastGPT documentation built with the fumadoc framework.

## Running the Project

To run the documentation, you first need to configure environment variables. Create a `.env.local` file in the document root directory and add the following environment variable:

```bash
FASTGPT_HOME_DOMAIN =    # Domain of the FastGPT project to redirect to (defaults to overseas version)
```

You can run the documentation from the FastGPT project root directory using the following commands:

```bash
npm install # Only use npm install, not pnpm
npm run dev
```

The project will run on `http://localhost:3000` by default.

## Writing Documentation

Documentation is written in `MDX` format, which is mostly similar to Markdown. Currently, the documentation metadata only supports three fields: `title`, `description`, and `icon`. Refer to the following example:

```mdx
---
title: FastGPT Documentation
description: Official FastGPT Documentation
icon: menu # Icons use the `lucide-react` third-party library
---

import { Alert } from '@/components/docs/Alert'; # Alert component for highlighted blocks

<Alert icon="🤖" context="success">
Quick Start Experience
- Overseas version: [https://fastgpt.io](https://fastgpt.io)
- Domestic version: [https://fastgpt.cn](https://fastgpt.cn)
</Alert>

import { Redirect } from '@/components/docs/Redirect' # Redirect component for automatic page redirection

<Redirect to="/docs/introduction/development/docker/#faq" />

<Tabs items={['JavaScript', 'Rust']}> # Tabs component usage
  <Tab value="JavaScript">JavaScript is weird</Tab>
  <Tab value="Rust">Rust is fast</Tab>
</Tabs>

import FastGPTLink from '@/components/docs/linkFastGPT'; # FastGPT redirect link component that automatically redirects to overseas or domestic versions based on domain environment variable

This document explains how to set up the development environment to build and test <FastGPTLink>FastGPT</FastGPTLink>.
```

### Adding New Documentation

After writing documentation, you need to add your filename to the `pages` field in the corresponding directory's `meta.json` file. For example, if you write a `hello.mdx` file in the `content/docs/introduction` directory (the default root for all documentation), you need to add the following to the `introduction` directory's `meta.json`:

```json
{
  "title": "FastGPT Docs",
  "root": true,
  "pages": [
    "[Handshake][Contact Us](https://fael3z0zfze.feishu.cn/share/base/form/shrcnjJWtKqjOI9NbQTzhNyzljc)",
    "index",
    "guide",
    "development",
    "FAQ",
    "shopping_cart",
    "community",
    "hello"
  ],
  "order": 1
}
```

**Note**: "hello" was added to the existing pages array. The order in the `pages` array determines the display order in the documentation. In this example, the "hello" document will appear at the end of the introduction section.

## Internationalization (i18n)

All `.mdx` files in `content/docs` are the default language files (currently defaults to Chinese). Files with `.en.mdx` extension are supported by i18n for English documentation.

For example, you can translate `hello.mdx` and create a `hello.en.mdx` file. At the same time, you need to add the corresponding filename to the `"pages"` field in the corresponding directory's `meta.en.json` to support English documentation.

### File Naming Conventions

- `filename.mdx` - Default language (Chinese)
- `filename.en.mdx` - English version
- `meta.json` - Default language metadata
- `meta.en.json` - English metadata

### Meta Configuration Examples

**Chinese (meta.json):**
```json
{
  "title": "使用文档",
  "root": true,
  "pages": [
    "---入门---",
    "index",
    "cloud",
    "commercial",
    "development",
    "---功能介绍---",
    "...guide"
  ],
  "order": 1
}
```

**English (meta.en.json):**
```json
{
  "title": "FastGPT Docs",
  "root": true,
  "pages": ["guide", "development", "FAQ", "shopping_cart", "community"]
}
```

## Special Configuration

### Adding Top-Level Navigation

To add new top-level navigation items:

1. Edit the `FastGPT/document/app/[lang]/docs/layout.tsx` file
2. Add the new navigation item to the navigation configuration

## Custom Components

### Alert Component

Used for creating highlighted notice blocks with different contexts:

```mdx
<Alert icon="🤖" context="success">
Your success message here
</Alert>

<Alert icon="⚠️" context="warning">
Your warning message here
</Alert>
```

### Redirect Component

Automatically redirects users to another page:

```mdx
<Redirect to="/docs/introduction/development/docker/#faq" />
```

### Tabs Component

Creates tabbed content sections:

```mdx
<Tabs items={['JavaScript', 'Rust']}>
  <Tab value="JavaScript">JavaScript content here</Tab>
  <Tab value="Rust">Rust content here</Tab>
</Tabs>
```

### FastGPTLink Component

Creates links that automatically redirect to the appropriate FastGPT instance based on domain configuration:

```mdx
<FastGPTLink>FastGPT</FastGPTLink>
```

## Available Icons

Icons use the `lucide-react` library. You can find available icons at: https://lucide.dev/icons/

Common icons include:
- `menu` - Menu icon
- `book` - Book icon
- `settings` - Settings icon
- `code` - Code icon
- `database` - Database icon
- And many more from lucide-react

## Development Tips

- Always test your documentation changes locally before committing
- Ensure all custom components are properly imported
- Check that meta.json files are valid JSON
- Verify that all links work correctly
- Test both Chinese and English versions if applicable
- Use meaningful and descriptive titles for better navigation

## Additional Resources

- [Fumadoc Framework Documentation](https://fumadocs.vercel.app/)
- [MDX Documentation](https://mdxjs.com/)
- [Lucide React Icons](https://lucide.dev/)