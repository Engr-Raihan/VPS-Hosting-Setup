# Contributing to VPS Hosting Ubuntu 24.04 Guide

Thank you for your interest in contributing to this project! 🎉

## How to Contribute

### Reporting Issues

If you find a problem with the guide:

1. Check if the issue already exists in [Issues](https://github.com/Engr-Raihan/VPS-Hosting-Setup/issues)
2. If not, create a new issue with:
   - Clear title describing the problem
   - Detailed description of what went wrong
   - Steps to reproduce
   - Your Ubuntu version and environment details
   - Expected vs. actual behavior

### Suggesting Improvements

Have an idea to make this guide better?

1. Open an issue with the "enhancement" label
2. Describe your suggestion clearly
3. Explain why it would be helpful
4. Provide examples if possible

### Submitting Changes

1. **Fork the Repository**
   ```bash
   # Click the "Fork" button on GitHub
   git clone https://github.com/your-username/VPS-Hosting-Setup.git
   cd VPS-Hosting-Setup
   ```

2. **Create a Branch**
   ```bash
   git checkout -b feature/your-feature-name
   # or
   git checkout -b fix/your-bug-fix
   ```

3. **Make Your Changes**
   - Follow the existing format and style
   - Test your changes on a fresh Ubuntu 24.04 instance
   - Update relevant sections
   - Add tips and notes where helpful

4. **Commit Your Changes**
   ```bash
   git add .
   git commit -m "Description of your changes"
   ```

5. **Push to Your Fork**
   ```bash
   git push origin feature/your-feature-name
   ```

6. **Create a Pull Request**
   - Go to the original repository on GitHub
   - Click "New Pull Request"
   - Select your branch
   - Fill in the PR template with:
     - What changed
     - Why it's needed
     - How you tested it

## Contribution Guidelines

### Content Standards

- **Accuracy**: All commands must be tested on Ubuntu 24.04 LTS
- **Security**: Follow security best practices
- **Clarity**: Write clear, beginner-friendly explanations
- **Completeness**: Include error handling and troubleshooting tips

### Formatting

- Use proper Markdown formatting
- Include code blocks with appropriate language tags
- Add comments in code examples
- Use emoji consistently (🚀 🔒 📦 etc.)
- Keep line length reasonable (under 120 characters)

### Code Examples

```bash
# Good: Include comments explaining what the command does
sudo systemctl restart nginx  # Restart Nginx to apply changes

# Bad: No explanation
sudo systemctl restart nginx
```

### Testing Checklist

Before submitting a PR, ensure:

- [ ] Commands work on a fresh Ubuntu 24.04 LTS installation
- [ ] No sensitive information (passwords, IPs) in examples
- [ ] All links work correctly
- [ ] Code blocks have proper syntax highlighting
- [ ] No typos or grammatical errors
- [ ] Changes don't break existing functionality

### What We're Looking For

**High Priority:**
- Bug fixes and corrections
- Security improvements
- Better explanations for complex topics
- Ubuntu 24.04 specific optimizations
- Additional troubleshooting scenarios

**Medium Priority:**
- Performance optimizations
- Additional examples
- Alternative approaches
- Tool updates

**Lower Priority:**
- Formatting improvements
- Typo fixes
- Link updates

## Style Guide

### Headings

```markdown
## 🔒 Security Hardening  # Use emoji + Title Case
### Step 1: Configure Firewall  # Use Step numbers
#### Optional: Advanced Configuration  # Sub-steps
```

### Command Examples

Always show:
- What the command does (comment)
- Expected output (when relevant)
- What to do if it fails

```bash
# Check service status
sudo systemctl status nginx

# Expected output: "Active: active (running)"
# If it fails, check logs: sudo journalctl -u nginx
```

### Tips and Warnings

```markdown
**💡 TIP:** Helpful advice for users

**⚠️ WARNING:** Critical information to prevent issues

**📝 Note:** Additional context or information
```

## Community Guidelines

- Be respectful and constructive
- Help others in discussions
- Share your experiences
- Provide detailed feedback

## Questions?

- Open an [Issue](https://github.com/Engr-Raihan/VPS-Hosting-Setup/issues) with the "question" label
- Start a [Discussion](https://github.com/Engr-Raihan/VPS-Hosting-Setup/discussions)
- Contact via email: engr.raihan.buet@gmail.com
- Connect on [LinkedIn](https://www.linkedin.com/in/engr-raihan/)

## Recognition

All contributors will be acknowledged in the README and release notes!

---

**Thank you for helping make this guide better for everyone!** 🙏

