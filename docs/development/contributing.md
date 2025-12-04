# Contributing to FastGPT

Thank you for your interest in contributing to FastGPT! This guide will help you understand how to contribute effectively to the project.

## Table of Contents

1. [Code of Conduct](#code-of-conduct)
2. [Getting Started](#getting-started)
3. [How to Contribute](#how-to-contribute)
4. [Development Workflow](#development-workflow)
5. [Pull Request Process](#pull-request-process)
6. [Types of Contributions](#types-of-contributions)
7. [Community Guidelines](#community-guidelines)

## Code of Conduct

### Our Pledge

We are committed to making participation in our community a harassment-free experience for everyone, regardless of:

- Age, body size, disability, ethnicity, gender identity and expression
- Level of experience, education, socioeconomic status, nationality, personal appearance
- Race, religion, or sexual identity

### Our Standards

**Positive behavior includes:**
- Using welcoming and inclusive language
- Being respectful of differing viewpoints and experiences
- Gracefully accepting constructive criticism
- Focusing on what is best for the community
- Showing empathy towards other community members

**Unacceptable behavior includes:**
- Harassing, bullying, or discriminating against others
- Publishing private information without permission
- Submitting malicious code or intentionally breaking the project
- Using derogatory language or slurs
- Creating a hostile or unwelcoming environment

### Enforcement

Instances of abusive behavior may be reported by contacting:
- Email: conduct@fastgpt.io
- GitHub issues with the "conduct" label

All reports will be reviewed and investigated in a way that protects the privacy of the reporter.

## Getting Started

### 1. Fork the Repository

1. Visit the [FastGPT repository](https://github.com/labring/FastGPT)
2. Click the "Fork" button in the top right
3. Clone your fork locally:

```bash
git clone https://github.com/YOUR_USERNAME/FastGPT.git
cd FastGPT
```

### 2. Set Up Development Environment

Follow the [Getting Started guide](./getting-started.md) to set up your development environment.

### 3. Configure Git

Set up your git configuration:

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Add your fork as a remote
git remote add fork https://github.com/YOUR_USERNAME/FastGPT.git
```

## How to Contribute

### 1. Find an Issue to Work On

- **Good First Issues**: Look for issues labeled `good first issue`
- **Help Wanted**: Issues labeled `help wanted` need community help
- **Bug Reports**: Help fix reported bugs
- **Feature Requests**: Implement new features

### 2. Create an Issue

If you've found a bug or have a feature request:

1. Check existing issues to avoid duplicates
2. Use the issue templates provided
3. Provide clear, detailed information
4. Include reproduction steps for bugs

#### Bug Report Template

```markdown
## Bug Description
A clear description of what the bug is.

## Reproduction Steps
1. Go to '...'
2. Click on '...'
3. Scroll down to '...'
4. See error

## Expected Behavior
What you expected to happen.

## Actual Behavior
What actually happened.

## Environment
- OS: [e.g., macOS 13.0]
- Browser: [e.g., Chrome 108]
- FastGPT Version: [e.g., v4.0.0]

## Additional Context
Any other relevant information.
```

#### Feature Request Template

```markdown
## Feature Description
A clear description of the feature you'd like to add.

## Problem Statement
What problem does this feature solve?

## Proposed Solution
How you envision the feature working.

## Alternatives Considered
Any alternative solutions you've thought of.

## Additional Context
Any other relevant information or examples.
```

## Development Workflow

### 1. Create a Branch

Always create a branch from `main` or the appropriate base branch:

```bash
# Update main branch
git checkout main
git pull upstream main

# Create feature branch
git checkout -b feature/your-feature-name

# Or for bugs
git checkout -b fix/your-bug-fix

# Or for documentation
git checkout -b docs/your-documentation-update
```

### 2. Follow Coding Standards

- Read the [Coding Standards](./coding-standards.md)
- All code must be in English
- Follow TypeScript best practices
- Write tests for new functionality
- Update documentation

### 3. Make Commits

Follow [conventional commits](https://www.conventionalcommits.org/):

```bash
# Format: <type>[optional scope]: <description>

# Examples:
git commit -m "feat(auth): add two-factor authentication"
git commit -m "fix(api): resolve user creation bug"
git commit -m "docs(readme): update installation instructions"
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Code formatting
- `refactor`: Code refactoring
- `test`: Adding tests
- `chore`: Build/dependency updates

### 4. Write Tests

Ensure your changes include appropriate tests:

```bash
# Run tests
pnpm test

# Run specific test file
pnpm test test/unit/my-feature.test.ts

# Run with coverage
pnpm test:coverage
```

### 5. Update Documentation

If your change affects:
- User-facing features: Update user documentation
- API changes: Update API documentation
- Architecture: Update architecture docs
- README: Update if needed

## Pull Request Process

### 1. Before Opening PR

- [ ] Your code follows the project's coding standards
- [ ] You have written tests for new functionality
- [ ] All tests pass locally (`pnpm test`)
- [ ] You have updated relevant documentation
- [ ] Your commit messages follow conventional commit format
- [ ] You have rebased your branch on latest main

### 2. Create Pull Request

1. Push your branch to your fork:
   ```bash
   git push fork feature/your-feature-name
   ```

2. Go to your fork on GitHub
3. Click "New Pull Request"
4. Select your branch
5. Fill out the PR template

### 3. Pull Request Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update

## How Has This Been Tested?
- Unit tests
- Integration tests
- Manual testing
- E2E tests

## Checklist
- [ ] My code follows the style guidelines
- [ ] I have performed self-review
- [ ] I have commented my code
- [ ] I have made corresponding changes to the docs
- [ ] My changes generate no new warnings
- [ ] I have added tests that prove my fix works
- [ ] New and existing tests pass locally
- [ ] Any dependent changes are merged and published

## Screenshots (if applicable)
Add screenshots to help explain your changes.

## Additional Context
Any other context about the pull request.
```

### 4. Code Review Process

1. **Automated Checks**: CI/CD runs automatically
   - Linting
   - Type checking
   - Tests
   - Build

2. **Review Requirements**:
   - At least one maintainer approval
   - All automated checks must pass
   - No merge conflicts

3. **Review Feedback**:
   - Address all review comments
   - Push changes to your branch
   - PR automatically updates

### 5. Merge Process

- Maintainers will merge approved PRs
- PRs are squashed to maintain clean history
- Release notes are automatically generated

## Types of Contributions

### 1. Code Contributions

#### Features
- Implement new functionality
- Follow feature branch workflow
- Include comprehensive tests
- Update documentation

#### Bug Fixes
- Fix reported issues
- Add regression tests
- Document the fix

#### Performance Improvements
- Optimize existing code
- Add performance benchmarks
- Document improvements

#### Refactoring
- Improve code structure
- Maintain functionality
- Update tests

### 2. Documentation Contributions

#### User Documentation
- Update `document/` directory
- Improve clarity and accuracy
- Add examples and tutorials

#### API Documentation
- Update OpenAPI specifications
- Add code examples
- Document endpoints

#### Developer Documentation
- Update `docs/` directory
- Improve setup guides
- Add architecture documentation

### 3. Plugin Development

Create new plugins to extend FastGPT:

1. Fork the plugins repository or create in `plugins/`
2. Follow the [Plugin Architecture Guide](../architecture/plugin-system.md)
3. Include comprehensive documentation
4. Provide example usage

### 4. Translation

Help translate FastGPT to new languages:

1. Locate translation files in `packages/web/i18n/`
2. Add new language folder if needed
3. Update existing translations
4. Test UI with new language

### 5. Design and UX

- Improve user interface
- Create better user experiences
- Submit mockups and prototypes
- Contribute to Chakra UI theme

## Community Guidelines

### 1. Communication Channels

- **GitHub Issues**: For bugs and feature requests
- **GitHub Discussions**: For general questions and ideas
- **Discord**: For real-time discussion
- **Email**: For security issues (security@fastgpt.io)

### 2. Asking for Help

When asking for help:

1. Check existing documentation and issues
2. Provide context about what you're trying to do
3. Include error messages and stack traces
4. Share what you've already tried

### 3. Design Discussions

For significant changes:

1. Open an issue for discussion first
2. Create design documents if needed
3. Get community feedback
4. Implement based on consensus

### 4. Security Issues

For security vulnerabilities:

- Email security@fastgpt.io directly
- Do not open public issues
- Include full details
- We'll respond promptly

## Recognition

### Contributors

All contributors are recognized in:

1. **Contributors.md**: List of all contributors
2. **Release Notes**: Mention of significant contributions
3. **Contributor Badge**: Special badge on GitHub profile
4. **Annual Recognition**: Top contributors highlighted

### Types of Recognition

- **Code Contributors**: PRs merged
- **Documentation**: Documentation improvements
- **Community**: Helping others in discussions
- **Bug Reports**: Quality issue reports
- **Design**: UI/UX improvements

## Development Tips

### 1. Debugging

```bash
# Debug Node.js
node --inspect-brk dist/server.js

# Debug Tests
pnpm test --debug

# Debug Playwright
pnpm test:e2e --debug
```

### 2. Common Issues

**Port Conflicts:**
```bash
# Check port usage
lsof -i :3000

# Kill process
kill -9 <PID>
```

**Dependencies:**
```bash
# Clean install
rm -rf node_modules pnpm-lock.yaml
pnpm install
```

**Cache Issues:**
```bash
# Clear caches
pnpm store prune
rm -rf .next/
rm -rf coverage/
```

### 3. Useful Git Commands

```bash
# Interactive rebase
git rebase -i HEAD~5

# Stash changes
git stash save "work in progress"

# Apply stash
git stash pop

# Find commit that introduced bug
git bisect start
git bisect bad HEAD
git bisect good <previous-commit>

# Squash commits
git rebase -i <base-commit>
```

## License

By contributing to FastGPT, you agree that your contributions will be licensed under the [MIT License](https://github.com/labring/FastGPT/blob/main/LICENSE).

## Getting Help

If you need help with contributing:

1. **Documentation**: Check project docs first
2. **Issues**: Search existing issues
3. **Discussions**: Ask in GitHub Discussions
4. **Discord**: Join our community
5. **Email**: hello@fastgpt.io

Thank you for contributing to FastGPT! Your contributions help make this project better for everyone. 🚀