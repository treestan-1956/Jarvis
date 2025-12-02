# CLAUDE.md - Jarvis Project Configuration

## Project Overview

**Jarvis** is a neurodivergent executive function prosthesis - an AI-augmented personal OS designed for individuals with AUDHD, autism, POTS, MCAS, EDS, and CPTSD.

The system provides a 24/7 compensatory cognitive scaffold combining automation, prediction, tracking, and emotional modeling to help users thrive sustainably.

## Project Structure

```
Jarvis/
├── docs/                    # Documentation
│   └── Jarvis High Level PRD.md
├── src/                     # Source code (TBD)
├── tests/                   # Test files (TBD)
├── .gitignore
├── CLAUDE.md               # This file
└── README.md
```

## Development Guidelines

### Core Principles

1. **User-Centric Design**: Every feature must reduce cognitive load, not add to it
2. **Privacy First**: User-owned data, local encryption, no cloud dependency unless opted in
3. **Neurodivergent-Friendly**: Low sensory load, predictable interfaces, nonjudgmental language
4. **Adaptability**: System must work in low-capacity states and adapt to nonlinear energy rhythms

### Code Style

- Follow standard conventions for the target language/framework
- Prioritize readability and maintainability
- Document complex logic with clear comments
- Use meaningful variable and function names

### Testing Requirements

- Minimum 90% unit test coverage
- Integration tests for all external integrations (Apple Health, Calendar, etc.)
- UI tests for critical user workflows

### Commit Message Format

Use conventional commits:
- `feat:` - New features
- `fix:` - Bug fixes
- `docs:` - Documentation changes
- `refactor:` - Code refactoring
- `test:` - Test additions/modifications
- `chore:` - Maintenance tasks

### Pull Request Guidelines

- Keep PRs focused and reasonably sized
- Include description of changes and testing performed
- Link to relevant issues when applicable
- Ensure all tests pass before requesting review

## Key Documentation

- [High Level PRD](docs/Jarvis%20High%20Level%20PRD.md) - Product requirements and vision

## Technology Stack (TBD)

Stack decisions will be made based on:
- iOS/macOS primary platform (Apple Health integration required)
- Privacy-first architecture (local-first data storage)
- AI/ML capabilities for pattern recognition and prediction
- Low-latency, calm UI requirements

## Out of Scope for v1.0

- Medical diagnosis
- Multi-user support
- Complex CBT therapy modules
- Remote-access automation
- Social networking features
- High-stimulation UI
- Real-time emergency response

## Contact

Owner: Tristan Parker
