# Contributing

Thanks for your interest in contributing to Browser Automation Skill!

## How to Contribute

### 1. Report Issues

Found a bug or have a feature request? [Open an issue](https://github.com/hienlh/claude-skill-browser-automation/issues/new).

### 2. Submit Pull Requests

1. Fork the repository
2. Create a branch: `git checkout -b feature/your-feature`
3. Make changes
4. Test your changes
5. Commit: `git commit -m "feat: your feature description"`
6. Push: `git push origin feature/your-feature`
7. Open a Pull Request

### Commit Convention

We use [Conventional Commits](https://www.conventionalcommits.org/):

- `feat:` - New feature
- `fix:` - Bug fix
- `docs:` - Documentation changes
- `refactor:` - Code refactoring
- `test:` - Adding tests

## Project Structure

```
browser-automation/
├── SKILL.md                      # Main skill file (keep concise <80 lines)
├── README.md                     # Documentation
├── LICENSE                       # MIT License
├── CONTRIBUTING.md               # This file
└── references/
    ├── interactive-flow.md       # Auth/login patterns
    ├── examples.md               # Code examples
    └── troubleshooting.md        # Error handling
```

## Guidelines

### SKILL.md

- Keep it **concise** (<80 lines)
- Quick reference only
- Move detailed content to `references/`

### References

- **Progressive disclosure** - detailed docs go here
- Use practical examples
- Include code snippets

### Code Examples

- Use `browser_run_code` for efficiency
- Include error handling
- Add comments for clarity

## Questions?

Open an issue or reach out!
