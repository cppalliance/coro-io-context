# Technical Writing Guidelines for Tutorials

## Core Principles

- **Comprehensive**: Include every command and explanation needed from start to finish
- **Technically accurate**: Test all instructions on fresh systems
- **Practical**: Reader ends with something usable
- **Formal but friendly**: No jargon, memes, slang, or emoji

## Style Rules

### Voice and Tone

- Use second person ("You will configure...") not first person singular
- Focus on outcomes: "In this tutorial, you will install Apache" not "You will learn how to install Apache"
- Avoid: "simple," "straightforward," "easy," "simply," "obviously," "just"

### Formatting

- Minimize formatting; avoid excessive bold, headers, lists, bullets
- Use prose paragraphs for explanations, not bullet lists
- Bold: GUI text, hostnames, usernames, term lists, emphasis for context changes
- Italics: Only for introducing technical terms
- Inline code: Commands, packages, filenames, paths, URLs, ports, key presses

## Article Structure

```
# Title (H1)
### Introduction (H3)
## Prerequisites (H2)
## Step 1 — Doing First Thing (H2)
## Step 2 — Doing Next Thing (H2)
## Step N — Doing Last Thing (H2)
## Conclusion (H2)
```

### Title

Format: "How To \<Accomplish Task\> with \<Software\> on \<Distro\>"

Include the goal, not just the tool. Keep under 60 characters.

### Introduction

Answer these questions in 1-3 paragraphs:

1. What is the tutorial about?
2. Why should the reader learn this?
3. What will the reader do or create?
4. What will they have accomplished when done?

### Prerequisites

- Formatted as a checklist
- Each item links to existing tutorial or documentation
- Be specific: server count, distribution, software versions, accounts needed

### Steps

Each step contains:

1. H2 header: "Step N — Gerund Phrase" (e.g., "Step 1 — Installing Nginx")
2. Introductory sentence describing what reader will do
3. Commands on their own lines in code blocks
4. Explanations before AND after commands
5. Transition sentence summarizing accomplishment and introducing next step

### Commands

```
Execute the following command to display directory contents:

ls -al /home/sammy

The -a switch shows hidden files, -l shows long listing.
```

Separate command output into labeled blocks:

```
Run the program:

node hello.js

Output:

Hello world!
```

### Files and Code

- Explicitly tell reader to create/open each file
- Label code blocks with filename
- Use highlighting to indicate changes in existing files
- Explain what code does and why

### Conclusion

- Summarize what reader accomplished ("you configured" not "we learned")
- Describe next steps, use cases, related tutorials

## Variables and Placeholders

| Placeholder | Usage |
|-------------|-------|
| `sammy` | Default username |
| `your_server` | Default hostname |
| `your_domain` | Default domain |
| `your_server_ip` | Default IP address |
| `203.0.113.x` | Example public IPs |
| `198.51.100.x` | Example private IPs |

Highlight all variables the reader must change.

## Images

- Use only for GUIs, interactive dialogues, architecture diagrams
- Never for code, config files, or terminal output
- Include descriptive alt text
- Include brief caption
- Use PNG format

## Technical Best Practices

- Test all instructions on fresh systems
- Follow official software capitalization
- Link to software home page on first mention
- Use project terminology for multi-server setups (primary/replica, manager/worker)