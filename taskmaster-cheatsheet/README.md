# Taskmaster AI - Complete Usage Guide

> An AI-powered task management system for breaking down projects into manageable tasks and building with AI assistants. It's basically a PM for your AI agents!

Why use Taskmaster AI? It helps you:
- Break down complex projects into structured tasks which are easy to manage
- Create detailed Product Requirements Documents (PRDs) to define your project scope
- Automate task generation from PRDs using AI
- Leverage AI to generate and expand tasks
- Integrate seamlessly with Claude Code for AI-assisted development

**Official Site:** [TaskMaster AI Documentation](https://www.task-master.dev/)

---

## 📋 Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [Creating a PRD (Product Requirements Document)](#creating-a-prd)
- [Using AI to Create Your PRD](#using-ai-to-create-your-prd)
- [Parsing PRD into Tasks](#parsing-prd-into-tasks)
- [Working with Tasks](#working-with-tasks)
- [Using Claude Code with Taskmaster](#using-claude-code-with-taskmaster)
- [Task Status Management](#task-status-management)
- [Model Configuration](#model-configuration)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)

---

## 🚀 Installation

### Prerequisites

- Node.js 16+ installed
- npm or npx available

### Global Installation (Recommended)

```bash
# Install Taskmaster globally
npm install -g task-master-ai

# Verify installation
task-master --version
```

### Quick Install with npx (No Installation)

```bash
# Run directly without installing
npx task-master-ai init
```

### MCP Integration (For Claude Code)

```bash
# Add Taskmaster as MCP server for Claude Code
claude mcp add taskmaster-ai -- npx -y task-master-ai

# Verify it was added
claude mcp list
```

---

## ⚡ Quick Start

### 1. Initialize Your Project

```bash
# Navigate to your project directory
cd your-project

# Initialize Taskmaster
task-master init

# Follow the interactive prompts:
# - Choose "Solo (Taskmaster)" for individual work
# - Select Git integration: Yes
# - Set up AI IDE rules: Yes
# - Configure AI models
```

**What gets created:**

```
your-project/
├── .taskmaster/
│   ├── config.json          # Configuration
│   ├── state.json           # Project state
│   ├── tasks/               # Task files
│   ├── docs/                # PRDs and documentation
│   ├── reports/             # Analysis reports
│   └── templates/           # PRD templates
├── .env.example             # Environment variables template
└── .gitignore               # Git ignore rules
```

### 2. Create Your PRD

```bash
# Copy an example PRD template
cp .taskmaster/templates/example_prd.txt .taskmaster/docs/my-project-prd.txt

# Edit it with your project details
nano .taskmaster/docs/my-project-prd.txt
```

### 3. Parse PRD into Tasks

```bash
# Generate tasks from your PRD
task-master parse-prd .taskmaster/docs/my-project-prd.txt
```

### 4. Start Working

```bash
# See what to work on next
task-master next

# View all tasks
task-master list

# Start working on first task
task-master set-status --id=1 --status=in-progress
```

---

## 📝 Creating a PRD

### What is a PRD?

A **Product Requirements Document** describes what you want to build. Taskmaster uses AI to parse this into structured tasks.

### PRD Structure

Create a file in `.taskmaster/docs/` (e.g., `project-prd.txt`) with this structure:

```markdown
# Project Title

## Overview
Brief description of what you're building and why.

**Target Audience:** Who is this for?
**Goal:** What problem does this solve?

## Features/Modules

### Module 1: Feature Name
**What it does:**
- List key functionality
- Describe user experience

**Technical Requirements:**
- Technologies needed
- Dependencies

**Deliverables:**
- What gets created
- Acceptance criteria

**Subtasks:**
- Specific steps to complete
- Research needed
- Documentation required

### Module 2: Another Feature
[Repeat structure]

## Technical Stack
- Frontend: React, Vue, etc.
- Backend: Node.js, Python, etc.
- Database: PostgreSQL, MongoDB, etc.

## Success Metrics
- How do you know it's done?
- What does success look like?
```

### Example PRD

```markdown
# Blog Platform with AI Chat

## Overview
Build a blog where users can write posts and chat with an AI about the content.

**Target Audience:** Developers learning Flask and RAG
**Goal:** Teach full-stack development + AI integration

## Module 1: User Authentication
**What it does:**
- Users can register with email/password
- Login and logout functionality
- Password hashing for security

**Technical Requirements:**
- Flask-Login for session management
- SQLAlchemy for user model
- Werkzeug for password hashing

**Deliverables:**
- User registration page
- Login page
- User model in database
- Authentication tests

**Subtasks:**
- Create User model with SQLAlchemy 2.0
- Implement password hashing
- Build registration form
- Build login form
- Add session management
- Write tests

## Module 2: Blog Post Creation
[Continue with more modules...]
```

---

## 🤖 Using AI to Create Your PRD

### Method 1: Ask Claude (Web Interface)

1. **Describe your project** to Claude in natural language:

```
I want to build a web application where users can:
- Register and log in
- Create blog posts with a rich text editor
- Have an AI chatbot answer questions about their posts
- The AI should use RAG (Retrieval-Augmented Generation)

Tech stack:
- Backend: Flask + PostgreSQL + SQLAlchemy 2.0
- Frontend: Bootstrap 5 + JavaScript
- AI: LangChain + OpenAI

Can you create a detailed PRD for this project with 10-12 modules?
Make it beginner-friendly and include subtasks for each module.
```

1. **Claude will generate a comprehensive PRD**
2. **Copy it** to `.taskmaster/docs/project-prd.txt`
3. **Parse it**: `task-master parse-prd .taskmaster/docs/project-prd.txt`

### Method 2: Use Claude Code in Terminal

```bash
# Start Claude Code
claude

# Then say:
"Create a detailed PRD for a [describe your project]. 
Include modules, technical requirements, and subtasks. 
Save it to .taskmaster/docs/project-prd.txt"
```

### Method 3: Iterative Refinement

```bash
# 1. Create basic PRD manually
nano .taskmaster/docs/prd.txt

# 2. Parse it
task-master parse-prd .taskmaster/docs/prd.txt

# 3. Review tasks
task-master list

# 4. Expand tasks for more detail
task-master expand --id=1

# 5. If tasks aren't detailed enough, update PRD and re-parse
```

### Tips for AI-Generated PRDs

**Do:**

- ✅ Be specific about technologies and versions
- ✅ Include "use Context7 to get latest docs" for each technology
- ✅ Specify beginner-friendly or advanced level
- ✅ Include deliverables and subtasks
- ✅ Mention documentation/transcript requirements
- ✅ Be explicit about what NOT to assume

**Don't:**

- ❌ Be vague ("build a website")
- ❌ Skip technical details
- ❌ Forget to mention testing requirements
- ❌ Leave out documentation needs

**Example Requests:**

```
"Create a PRD for a Flask blog with these requirements:
- Use SQLAlchemy 2.0 (new ORM syntax)
- Include Context7 integration to fetch latest docs
- Each module needs a tutorial transcript
- Make it beginner-friendly - explain every concept
- Include async processing with Celery
- Target audience: intermediate Python developers"
```

---

## 📦 Parsing PRD into Tasks

### Basic Parsing

```bash
# Parse your PRD
task-master parse-prd .taskmaster/docs/project-prd.txt
```

### What Happens

1. **AI reads your PRD**
2. **Breaks it into tasks** (usually 10-15 main tasks)
3. **Sets dependencies** (task order)
4. **Assigns priorities** (high/medium/low)
5. **Saves to** `.taskmaster/tasks/tasks.json`

### Viewing Results

```bash
# View all generated tasks
task-master list

# See recommended next task
task-master next

# View specific task details
task-master show 1
```

---

## 📋 Working with Tasks

### Expanding Tasks into Subtasks

Break down tasks into actionable subtasks:

```bash
# Expand specific task
task-master expand --id=1

# Expand all tasks (this takes time!)
task-master expand --all

# Expand with research mode for more detail
task-master expand --id=1 --research
```

**What expansion does:**

- Creates 3-5 subtasks per task
- Adds implementation details
- Identifies dependencies
- Suggests test strategies

### Viewing Tasks

```bash
# List all tasks
task-master list

# List by status
task-master list --status=pending
task-master list --status=in-progress

# Show detailed view of a task
task-master show 1

# See what to work on next
task-master next
```

### Task Properties

Each task has:

- **ID:** Unique identifier (e.g., `1`, `1.1`, `1.2`)
- **Title:** Short description
- **Status:** `pending`, `in-progress`, `review`, `done`, `blocked`, `cancelled`
- **Priority:** `high`, `medium`, `low`
- **Dependencies:** Which tasks must be completed first
- **Complexity:** Estimated difficulty
- **Description:** Detailed implementation notes

---

## 🔧 Using Claude Code with Taskmaster

### Setup

1. **Install Taskmaster MCP Server:**

```bash
# Add Taskmaster as MCP server
claude mcp add taskmaster-ai -- npx -y task-master-ai

# Verify
claude mcp list
```

1. **Configure in Your Project:**

Your project should have:

- `.mcp.json` or `.claude/mcp.json` with Taskmaster config
- `.taskmaster/` directory with tasks

### Starting Claude Code

```bash
# Navigate to your project
cd your-project

# Start Claude Code
claude

# Claude Code can now see your Taskmaster tasks!
```

### Working with Tasks in Claude Code

**Basic Commands:**

```bash
# In Claude Code chat, type:

"Show me the current Taskmaster tasks"
"What's the next task I should work on?"
"Start working on task 1"
"Help me with subtask 1.1"
"Show me task 3 details"
```

**Example Workflow:**

```bash
# Start Claude Code
claude

# Example conversation:
You: "Show me the next Taskmaster task"
Claude: "Task #1: Create Module 1 - Project Introduction. Would you like me to help with this?"

You: "Yes, start working on it"
Claude: [Creates files, writes code, implements the task]

You: "Mark task 1 as done"
Claude: [Updates task status]
```

### Let Claude Code Work Autonomously

**Method 1: Direct Task Assignment**

```bash
# In Claude Code:
"Work on Taskmaster task 1 and all its subtasks. 
Use Context7 to fetch the latest documentation for any technologies.
Create all necessary files and implement the complete solution."
```

**Method 2: Autopilot Mode (if available)**

```bash
# Some Claude Code versions support autopilot
task-master autopilot start 1

# Claude Code will:
# - Read the task
# - Create files
# - Write code
# - Test implementation
# - Mark subtasks complete
```

**Method 3: Subtask-by-Subtask**

```bash
# More controlled approach
"Help me with subtask 1.1 from Taskmaster"
[Claude implements 1.1]

"Now do subtask 1.2"
[Claude implements 1.2]

"Continue with remaining subtasks for task 1"
[Claude completes the rest]
```

### Claude Code Best Practices

**For Code Tasks:**

- ✅ Let Claude Code create files directly
- ✅ Ask it to use Context7 for latest docs
- ✅ Review code before marking complete
- ✅ Ask for tests to be included

**For Writing Tasks:**

- ✅ Have Claude Code create markdown files
- ✅ Ask for beginner-friendly explanations
- ✅ Request examples and code snippets
- ✅ Get it to create diagrams (Mermaid)

**Example Prompts:**

```bash
"Work on Taskmaster task 3: Flask Basics.
Use Context7 to get the latest Flask documentation.
Create the application factory pattern.
Add inline comments explaining each part.
Make it beginner-friendly."

"Implement subtask 1.2: Create Architecture Diagrams.
Use Mermaid syntax to create diagrams.
Save them to .taskmaster/docs/diagrams/"

"Help me with task 5: User Authentication.
Use Context7 to get latest Flask-Login docs.
Implement with best practices.
Include password hashing with Werkzeug.
Add tests."
```

---

## 📊 Task Status Management

### Status Types

- **pending** - Not started yet
- **in-progress** - Currently working on it
- **review** - Completed, needs review
- **done** - Completed and verified
- **blocked** - Cannot proceed (dependency or issue)
- **cancelled** - No longer needed
- **deferred** - Postponed to later

### Changing Status

```bash
# Start working on a task
task-master set-status --id=1 --status=in-progress

# Mark as complete
task-master set-status --id=1 --status=done

# Mark subtask complete
task-master set-status --id=1.1 --status=done

# Block a task with reason
task-master set-status --id=3 --status=blocked

# Cancel a task
task-master set-status --id=5 --status=cancelled
```

### Checking Status

```bash
# View all tasks with status
task-master list

# Filter by status
task-master list --status=in-progress
task-master list --status=pending
task-master list --status=done

# See project dashboard
task-master status

# Get next recommended task
task-master next
```

### Task Dependencies

Tasks can depend on other tasks:

```bash
# Task 2 depends on Task 1
# You can't start Task 2 until Task 1 is done

# View task dependencies
task-master show 2
# Shows: Dependencies: 1

# Taskmaster will warn if you try to start a blocked task
```

---

## ⚙️ Model Configuration

### Viewing Current Models

```bash
# See current model configuration
task-master models

# Check API key status
task-master models --status
```

### Setting Models

**Interactive Setup (Easiest):**

```bash
task-master models --setup
```

**Command Line:**

```bash
# Set main model
task-master models --set-main claude-sonnet-4-20250514

# Set research model (for complex tasks)
task-master models --set-research claude-opus-4-20250514

# Set fallback model
task-master models --set-fallback claude-haiku-4-5
```

### Using Free Models

If you have Claude subscription:

```bash
# Use Claude Code (FREE)
task-master models --set-main sonnet --claude-code

# Use Gemini CLI (FREE)
task-master models --set-main gemini-2.5-pro --gemini-cli

# Use Codex CLI (FREE)
task-master models --set-main gpt-5.2 --codex-cli
```

### API Keys

Add to `.env` file:

```bash
# Anthropic
ANTHROPIC_API_KEY=sk-ant-your-key-here

# OpenAI
OPENAI_API_KEY=sk-your-key-here

# Google
GOOGLE_API_KEY=your-key-here
```

---

## 💡 Best Practices

### PRD Writing

1. **Be specific** about technologies and versions
2. **Include Context7 instructions** for fetching latest docs
3. **Break into logical modules** (8-15 modules ideal)
4. **Add subtasks** for each module in the PRD
5. **Include deliverables** (what gets created)
6. **Specify audience level** (beginner, intermediate, advanced)
7. **Mention documentation** needs (READMEs, tutorials, etc.)

### Task Management

1. **Start with expand** - Expand tasks before working
2. **Work sequentially** - Follow dependency order
3. **Keep status updated** - Mark in-progress and done
4. **Review before completing** - Check quality
5. **Use subtasks** - Break down complex tasks
6. **Track blockers** - Note why tasks are blocked

### Working with AI

1. **Be specific in requests** - "Use Context7 to get latest Flask docs"
2. **Request explanations** - "Explain this code for beginners"
3. **Ask for tests** - "Include unit tests"
4. **Review AI output** - Don't blindly accept
5. **Iterate** - Refine based on results

### Project Organization

```
your-project/
├── .taskmaster/
│   ├── docs/
│   │   ├── project-prd.txt          # Main PRD
│   │   ├── architecture.md          # Architecture docs
│   │   └── diagrams/                # Visual diagrams
│   ├── tasks/
│   │   └── tasks.json               # All tasks
│   ├── reports/
│   │   └── complexity-report.json   # Complexity analysis
│   └── templates/
│       └── example_prd.txt          # PRD templates
├── src/                             # Your code
├── tests/                           # Your tests
└── .env                             # API keys
```

---

## 🔍 Common Commands Reference

### Project Setup

```bash
task-master init                     # Initialize project
task-master models --setup           # Configure AI models
```

### PRD & Tasks

```bash
task-master parse-prd <file>         # Generate tasks from PRD
task-master list                     # View all tasks
task-master next                     # Get next task
task-master show <id>                # View task details
task-master expand --id=<id>         # Break into subtasks
task-master expand --all             # Expand all tasks
```

### Status Management

```bash
task-master set-status --id=<id> --status=<status>
task-master list --status=<status>
task-master status                   # Project dashboard
```

### Analysis

```bash
task-master analyze-complexity       # Analyze task complexity
task-master complexity-report        # Generate complexity report
task-master sprint-plan              # Plan sprints
```

### Utilities

```bash
task-master models                   # View model config
task-master --help                   # Show help
task-master --version                # Show version
```

---

## 🐛 Troubleshooting

### "Command not found: task-master"

```bash
# Option 1: Install globally
npm install -g task-master-ai

# Option 2: Use npx
npx task-master-ai init

# Option 3: Add to PATH
export PATH=$(npm bin -g):$PATH
```

### "API key not found"

```bash
# Create .env file
nano .env

# Add your key
ANTHROPIC_API_KEY=sk-ant-your-key-here

# Verify
cat .env
```

### "Parse hanging/not working"

1. **Check model configuration**:

   ```bash
   task-master models --status
   ```

2. **Try a simpler PRD first**:

   ```bash
   # Create small test PRD
   cat > .taskmaster/docs/test.txt << 'EOF'
   # Test Project
   ## Module 1: Setup
   Create basic project structure
   EOF
   
   # Parse it
   task-master parse-prd .taskmaster/docs/test.txt
   ```

3. **Use free model**:

   ```bash
   task-master models --set-main sonnet --claude-code
   ```

### "Tasks.json not found"

```bash
# Re-initialize
task-master init

# Re-parse PRD
task-master parse-prd .taskmaster/docs/prd.txt
```

### "Deprecation warnings"

These are harmless Node.js warnings. To hide:

```bash
export NODE_NO_WARNINGS=1
```

### MCP Integration Not Working

```bash
# Remove and re-add
claude mcp remove taskmaster-ai
claude mcp add taskmaster-ai -- npx -y task-master-ai

# Verify
claude mcp list

# Restart Claude Code
claude
```

---

## 📚 Additional Resources

- **Official Docs:** [Task Master Documentation](https://github.com/eyaltoledano/claude-task-master)
- **Hamster (Team Version):** [tryhamster.com](https://tryhamster.com)
- **Claude Code:** [claude.ai](https://claude.ai)
- **Context7:** [context7.com](https://context7.com)

---

## 🎯 Quick Workflow Example

Complete workflow from start to finish:

```bash
# 1. Setup
cd my-project
task-master init
task-master models --setup

# 2. Create PRD (use AI or write manually)
# Ask Claude to create a PRD, then:
nano .taskmaster/docs/project-prd.txt

# 3. Generate tasks
task-master parse-prd .taskmaster/docs/project-prd.txt

# 4. View tasks
task-master list
task-master next

# 5. Expand first task
task-master expand --id=1

# 6. Start working
task-master set-status --id=1 --status=in-progress

# 7. Work with Claude Code
claude
# In Claude: "Help me with Taskmaster task 1"

# 8. Mark complete
task-master set-status --id=1 --status=done

# 9. Continue to next task
task-master next

# Repeat steps 6-9 for each task!
```

---

## 📖 Example: Full Tutorial Project

Here's a real example from a Flask RAG tutorial:

```bash
# 1. Initialize
cd flask-ai-rag
task-master init

# 2. Set up model (free option)
task-master models --set-main sonnet --claude-code

# 3. Create comprehensive PRD
# (Ask Claude AI to create it, then save to file)

# 4. Parse PRD
task-master parse-prd .taskmaster/docs/prd.txt

# Result: 12 tasks created!
# - Module 1: RAG Concepts
# - Module 2: Environment Setup
# - Module 3: Flask Basics
# - Module 4: PostgreSQL & SQLAlchemy 2.0
# - ... (12 modules total)

# 5. Expand first task
task-master expand --id=1
# Result: 5 subtasks created
# - 1.1: Write RAG concepts
# - 1.2: Create diagrams
# - 1.3: Write tech stack explanation
# - 1.4: Write transcript
# - 1.5: Review and finalize

# 6. Start first subtask
task-master set-status --id=1.1 --status=in-progress

# 7. Work with Claude Code
claude
# "Help me with subtask 1.1 from Taskmaster.
#  Write beginner-friendly RAG explanation with analogies.
#  Save to .taskmaster/docs/module1-rag-concepts.md"

# 8. Mark complete
task-master set-status --id=1.1 --status=done

# 9. Continue through all subtasks
task-master next  # Shows 1.2 is ready
# ... repeat for all 12 modules
```

---

## 🎓 Tips for Tutorial/Documentation Projects

If you're creating tutorials (like the Flask RAG example):

1. **In your PRD, specify:**
   - "Include tutorial transcript (2500-5000 words) for each module"
   - "Use Context7 to fetch latest documentation"
   - "Make it beginner-friendly - explain every concept"
   - "Include code examples with inline comments"

2. **Use subtasks for content creation:**
   - Research and outline
   - Write draft
   - Create code examples
   - Write transcript
   - Review and edit

3. **Let Claude Code help:**

   ```bash
   "Write Module 1 tutorial about RAG concepts.
   Use analogies for beginners.
   Include architecture diagrams in Mermaid.
   Save to docs/tutorials/module1.md"
   ```

4. **Track deliverables:**
   - Blog post markdown
   - Code examples
   - Video transcript
   - Diagrams
   - README

---

**Happy building with Taskmaster AI! 🚀**

For questions or issues, visit the [GitHub repository](https://github.com/eyaltoledano/claude-task-master) or reach out to [@eyaltoledano](https://x.com/eyaltoledano).
