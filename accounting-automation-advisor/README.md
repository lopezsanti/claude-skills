# Accounting Automation Advisor Skill — Deployment & GitHub Guide

## What this skill does

The `accounting-automation-advisor` skill turns Claude into a senior AI agentic architect
for accounting automation. When you or a team member asks Claude to automate any accounting
process, this skill guides the full workflow:

- Captures business requirements and designs the agent architecture
- Writes a PRD in the right format for business stakeholder sign-off
- Selects the best platform (n8n, recon-lite, CrewAI, or hybrid)
- Produces Agile stories in JSON format ready for sprint planning
- Provides deployment instructions at three levels of effort

---

## Installing the Skill (Solo Developer)

### Option 1 — Install directly in Claude (Cowork / Claude Code)

1. Open your Claude Cowork or Claude Code session
2. Run: `install skill accounting-automation-advisor.skill`  
   (or drag the `.skill` file into the Cowork window)
3. The skill is now active in your project — Claude will use it automatically when you ask
   about accounting automation

### Option 2 — Install into a Claude Code project's skill folder

```bash
# Copy the skill to your project's .claude/skills folder
cp accounting-automation-advisor.skill /path/to/your/project/.claude/skills/

# Unpack it (Claude Code reads the folder, not the .skill zip)
cd /path/to/your/project/.claude/skills/
unzip accounting-automation-advisor.skill -d accounting-automation-advisor
```

The next time Claude Code runs in that project, the skill will be available.

---

### Developer workflow — installing a skill from the team repo

Any developer on the team installs the latest skill with:

```bash
# Clone the skills repo (one-time setup)
git clone https://github.com/your-org/accounting-skills.git ~/accounting-skills

# Install into a project
cd ~/accounting-skills
cp releases/accounting-automation-advisor.skill /path/to/your-accounting-project/.claude/skills/
cd /path/to/your-accounting-project/.claude/skills/
unzip accounting-automation-advisor.skill -d accounting-automation-advisor
```

Or install from the packaged .skill file by downloading it from GitHub Releases:

```bash
# Download from GitHub Releases page
gh release download v1.0.0 --repo your-org/accounting-skills --pattern "*.skill"

### Developer workflow — installing a skill from the team repo

Any developer on the team installs the latest skill with:

```bash
# Clone the skills repo (one-time setup)
git clone https://github.com/your-org/accounting-skills.git ~/accounting-skills

# Install into a project
cd ~/accounting-skills
cp releases/accounting-automation-advisor.skill /path/to/your-accounting-project/.claude/skills/
cd /path/to/your-accounting-project/.claude/skills/
unzip accounting-automation-advisor.skill -d accounting-automation-advisor
```

Or install from the packaged .skill file by downloading it from GitHub Releases:

```bash
# Download from GitHub Releases page
gh release download v1.0.0 --repo your-org/accounting-skills --pattern "*.skill"

## Quick-Start Cheatsheet for a New Developer

```bash
# Clone the skills repo
git clone https://github.com/your-org/accounting-skills.git

# Install into your active project
cp accounting-skills/releases/accounting-automation-advisor.skill \
   my-accounting-project/.claude/skills/
cd my-accounting-project/.claude/skills/
unzip accounting-automation-advisor.skill -d accounting-automation-advisor

# Start Claude Code in your project — skill is now active
cd my-accounting-project
claude

# Ask Claude to start a new accounting automation
# "I want to automate our AP aging report for multiple clients"
```

That's it. Claude will pick up the skill and walk you through the full design and
implementation process.