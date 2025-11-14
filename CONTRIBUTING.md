# Contributing to Django Multi-Tenant Prompt

Thank you for your interest in contributing to this project! This guide will help you get started.

## 📋 Table of Contents

- [How Can I Contribute?](#how-can-i-contribute)
- [Development Process](#development-process)
- [Style Guidelines](#style-guidelines)
- [Submitting Changes](#submitting-changes)
- [Reporting Issues](#reporting-issues)

## How Can I Contribute?

### 🐛 Reporting Bugs

If you find an issue with the implementation plan or code examples:

1. Check if the issue already exists
2. Create a new issue with:
   - Clear title and description
   - Steps to reproduce (if applicable)
   - Expected vs actual behavior
   - Django/Python version information

### 💡 Suggesting Enhancements

We welcome suggestions for improvements:

1. Check if the suggestion already exists
2. Create an issue describing:
   - The use case or problem
   - Your proposed solution
   - Any alternative approaches considered

### 📝 Improving Documentation

Documentation improvements are always welcome:

- Fix typos or clarify confusing sections
- Add missing examples or explanations
- Improve code comments
- Update outdated information

### 🔧 Contributing Code Examples

If you'd like to contribute code improvements:

1. Ensure examples are complete and working
2. Follow Django best practices
3. Include comments explaining complex logic
4. Test examples in a real Django project if possible

## Development Process

### Setting Up

1. Fork the repository
2. Clone your fork locally
3. Create a new branch for your changes

```bash
git checkout -b feature/your-feature-name
```

### Making Changes

1. Make your changes in the appropriate files
2. Update relevant documentation
3. Test any code examples you modify
4. Commit with clear, descriptive messages

```bash
git commit -m "Add: Clear description of what you added/changed"
```

## Style Guidelines

### Documentation Style

- Use clear, concise language
- Include practical examples
- Format code blocks with proper syntax highlighting
- Use consistent heading levels
- Keep line length reasonable (80-100 characters when possible)

### Code Example Style

Follow Django and Python best practices:

```python
# Good: Clear, documented, following conventions
class TenantAwareManager(models.Manager):
    """Manager that provides tenant-scoped query methods."""
    
    def for_tenant(self, tenant):
        """Filter queryset to specific tenant."""
        return self.get_queryset().filter(tenant=tenant)
```

### Commit Message Format

Use conventional commit format:

- `Add:` New features or documentation
- `Fix:` Bug fixes or corrections
- `Update:` Changes to existing content
- `Remove:` Deletion of content
- `Refactor:` Code restructuring without behavior change

Examples:
```
Add: Example for handling tenant switching
Fix: Incorrect middleware configuration in phase 2
Update: Clarify role hierarchy in architecture doc
```

## Submitting Changes

### Pull Request Process

1. Update documentation to reflect any changes
2. Ensure all code examples are valid Python/Django
3. Update the README if you're adding new sections
4. Create a pull request with:
   - Clear title describing the change
   - Detailed description of what and why
   - Reference to any related issues
   - Screenshots (if applicable)

### Review Process

- Maintainers will review your PR
- Address any feedback or requested changes
- Once approved, your PR will be merged

### What to Expect

- Initial response within a few days
- Constructive feedback on improvements
- Recognition for your contribution

## Reporting Issues

### Security Issues

If you discover a security vulnerability in the implementation guidance:

- **Do not** open a public issue
- Email the maintainers directly
- Provide detailed information about the vulnerability

### Bug Reports

Use this template for bug reports:

```markdown
**Description**
Clear description of the issue

**Steps to Reproduce**
1. Go to '...'
2. Look at '...'
3. See error

**Expected Behavior**
What should happen

**Actual Behavior**
What actually happens

**Environment**
- Django version:
- Python version:
- Database:

**Additional Context**
Any other relevant information
```

### Feature Requests

Use this template for feature requests:

```markdown
**Problem Statement**
Describe the problem or use case

**Proposed Solution**
How you think it should be solved

**Alternatives Considered**
Other approaches you've thought about

**Additional Context**
Any other relevant information
```

## Code of Conduct

### Our Standards

- Be respectful and inclusive
- Accept constructive criticism gracefully
- Focus on what's best for the community
- Show empathy towards others

### Unacceptable Behavior

- Harassment or discriminatory language
- Trolling or insulting comments
- Personal or political attacks
- Publishing others' private information

## Questions?

Don't hesitate to ask questions:

- Open an issue with the `question` label
- Be specific about what you need help with
- Provide context about your use case

## Recognition

Contributors will be recognized in:

- The project README
- Release notes for significant contributions
- Project documentation where appropriate

## License

By contributing, you agree that your contributions will be licensed under the same MIT License that covers the project.

---

Thank you for contributing to making Django multi-tenant implementation more accessible to everyone! 🎉
