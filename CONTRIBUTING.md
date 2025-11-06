# Contributing to Awesome Claude Contexts

Thank you for considering contributing to this project! This guide will help you understand how to contribute effectively.

## How Can I Contribute?

### 1. Adding a New Context File

If you have a context file for a framework or technology that isn't already included:

1. **Fork the repository**
2. **Create a new branch**: `git checkout -b add-nextjs-context`
3. **Add your context file** in the appropriate directory:
   ```
   framework-name/
   └── framework-version.md
   ```
4. **Update README.md** to include your context file in the relevant section
5. **Submit a pull request**

### 2. Improving Existing Context Files

Found a way to improve an existing context file?

1. **Fork the repository**
2. **Create a new branch**: `git checkout -b improve-laravel-context`
3. **Make your improvements** (add missing patterns, fix typos, update commands)
4. **Submit a pull request** with a clear description of what you improved

### 3. Reporting Issues

Found a problem? [Open an issue](https://github.com/apo-bozdag/awesome-claude-contexts/issues) with:
- Clear title describing the issue
- Which context file has the problem
- What's wrong and what should be correct
- Example if applicable

## Context File Guidelines

### Quality Standards

A good context file should:

✅ **Be Framework-Specific**
- Focus on one framework/technology and its specific version
- Include version-specific features and best practices

✅ **Be Comprehensive**
- Cover project structure and architecture patterns
- Include coding standards and conventions
- Provide common commands and workflows
- Address security and performance concerns

✅ **Be Actionable**
- Use clear, specific instructions
- Include code examples where helpful
- Provide commands that can be copy-pasted

✅ **Be Well-Organized**
- Use clear markdown headings
- Group related information together
- Use bullet points for lists
- Include a table of contents for long files

### Template Structure

Use this structure as a guide:

```markdown
# [Framework] [Version] Development Guide

> Brief description

## Project Structure
### Architecture Patterns
### Directory Organization

## Coding Standards
### Language Features
### Framework Conventions
### Database
### Testing

## Common Commands
### Development
### Build/Deploy
### Code Quality

## [Framework-Specific Sections]
(e.g., API Development, Component Patterns, etc.)

## Performance
### Caching
### Optimization Techniques

## Security
### Common Vulnerabilities
### Best Practices

## Troubleshooting
### Common Issues
### Debugging

## Resources
- Official documentation
- Community resources
```

## Pull Request Process

1. **Ensure your context file follows the guidelines above**
2. **Update the README.md** with a link to your new context file
3. **Write a clear PR description**:
   - What framework/technology does this cover?
   - What version(s) are supported?
   - What key features does it include?
4. **Respond to review feedback** if requested

## Commit Message Guidelines

Use clear, descriptive commit messages:

```
Add Next.js 14 context file
Update Laravel 12 context with queue best practices
Fix typo in React 18 context
```

## Code of Conduct

### Our Pledge

We pledge to make participation in our project a harassment-free experience for everyone, regardless of age, body size, disability, ethnicity, gender identity and expression, level of experience, nationality, personal appearance, race, religion, or sexual identity and orientation.

### Our Standards

**Positive behavior includes:**
- Using welcoming and inclusive language
- Being respectful of differing viewpoints
- Gracefully accepting constructive criticism
- Focusing on what is best for the community

**Unacceptable behavior includes:**
- Trolling, insulting/derogatory comments, and personal attacks
- Public or private harassment
- Publishing others' private information without permission
- Other conduct which could reasonably be considered inappropriate

## Questions?

Feel free to [open an issue](https://github.com/apo-bozdag/awesome-claude-contexts/issues) with the `question` label, or reach out to the maintainers.

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

---

Thank you for helping make Claude Code development better for everyone! 🚀
