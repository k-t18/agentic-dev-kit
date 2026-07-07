# Contributing to Agentic Dev Kit Plugin

Guidelines for contributing to this project.

## Suggesting New Skills

When suggesting a new skill:

1. Explain the use case and target audience
2. Describe what the skill should do
3. List relevant technologies/frameworks
4. Provide examples of when it would be triggered

## Submitting Changes

1. Fork and Clone
2. Create a Branch
3. Make Your Changes
4. Test Your Changes
5. Validate Your Skill
6. Commit Your Changes
7. Push and Create Pull Request

## Skill Writing Guidelines

### Frontmatter Schema

```yaml
---
name: my-skill-name
description: Use when [triggering conditions]. Invoke for [specific keywords].
license: MIT
metadata:
  author: https://github.com/YourGitHub
  version: "1.0.0"
  triggers: keyword1, keyword2, phrase1
  role: specialist
  scope: implementation
  output-format: code
  domain: frontend
  related-skills: react-renderer, typescript-teacher, nextjs-nerd
---
```

**Description Formula:**

```
Use when [triggering conditions]. Invoke for [specific keywords].
```

**Example:**

```yaml
description: Use when building React 18+ applications requiring component architecture, hooks patterns, or state management. Invoke for Server Components, performance optimization, Suspense boundaries, React 19 features.
```

### Required Sections (In Order)

````markdown
# [Skill Name]

[One-sentence role definition]

## Role Definition

[2-3 sentences defining expert persona with years of experience and specializations]

## When to Use This Skill

- [Bullet list of specific scenarios]
- [When this skill should be triggered]

## Core Workflow

1. **Step** - Brief description
2. **Step** - Brief description
3. **Step** - Brief description

## Technical Guidelines

[Framework-specific patterns, code examples, tables]

### Subsection Title

| Column | Column |
| ------ | ------ |
| Data   | Data   |

```language
// Code examples with comments
```
````

## Constraints

### MUST DO

- [Required practices - strong directive language]
- [Use imperative form]

### MUST NOT DO

- [Things to avoid - strong directive language]
- [Use imperative form]

## Output Templates

When implementing [X], provide:

1. [Expected output format]
2. [Additional deliverables]

## Knowledge Reference

[Comma-separated keywords only - no sentences]
