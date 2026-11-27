# ProcessMono Development Guide

> **📋 Note for AI Assistant:** Read this file at the start of each session to understand project conventions and safety procedures.

## 🛡️ Local Backup Strategy

**Purpose:** Protect against accidental file reverts, bulk deletions, or workflow mistakes.

### Backup Before Major Changes

Before starting any of these activities, **create a manual backup**:
- Restructuring ProcessMono.js (moving sections, refactoring functions)
- Large deletions or file removals
- Reverting commits or resetting branches
- Applying complex multi-file edits
- Any session where you'll be making repeated `git` commands

### How to Create a Local Backup

**PowerShell (Windows):**
```powershell
# Create a timestamped backup of ProcessMono.js before major changes
$timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
Copy-Item "src\scripts\ProcessMono.js" "src\scripts\ProcessMono.js.backup_$timestamp"
Write-Host "Backup created: ProcessMono.js.backup_$timestamp"
```

**Or backup the entire src folder:**
```powershell
$timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
$backupDir = "backups\src_backup_$timestamp"
Copy-Item -Path "src" -Destination $backupDir -Recurse
Write-Host "Full backup created: $backupDir"
```

### Restore from Backup

If a revert or deletion goes wrong:
```powershell
# List available backups
Get-ChildItem src\scripts\ProcessMono.js.backup_*

# Restore from a specific backup
Copy-Item "src\scripts\ProcessMono.js.backup_20261123_161530" "src\scripts\ProcessMono.js" -Force
```

### Backup File Retention

- Keep backup files locally (do NOT commit to git)
- Backups are added to `.gitignore` automatically
- Delete old backups manually when they're no longer needed
- Use filenames with timestamps for easy identification

---

## 🎯 Development Practices

### Code Style
- Use consistent naming conventions: camelCase for variables/functions, PascalCase for classes
- Keep related UI code grouped together
- Add comments only when logic is non-obvious

### Function Documentation
- Use JSDoc-style comments for all functions.
- Include `@function`, `@summary`, `@param`, and `@returns` tags where applicable.
- Provide a brief description of the function's purpose and behavior.
- Document any side effects, such as modifying global state or UI elements.

### MD Documentation
- Limit markdown files to README.md and DEVELOPMENT.md
- README.md should include an overview of the project, installation instructions, and basic usage examples.
- README.md should also include a high level overview of the project's design, structure, and behavior.
- DEVELOPMENT.md should document development practices, coding standards, and backup procedures.

### Testing Changes
- Run PixInsight script validation before committing
- Test UI changes with realistic workflows
- Verify all dropdowns and preset combinations work as expected

### Git Commits
- Include clear commit messages explaining what changed and why
- Keep commits focused on a single feature or fix
- Use `git status` and review changes before committing

### Key Files
- **src/scripts/ProcessMono.js** - Main script (auto-backup before large edits)
- **DEVELOPMENT.md** - This file (read by AI at session start)

---

## ⚠️ Safety Checklist

Before running major commands, verify:
- [ ] Recent changes are committed or backed up
- [ ] No uncommitted work you want to keep
- [ ] You understand what the command will do
- [ ] A backup copy exists if this is a risky operation

---

## 🤖 AI Assistant Interaction Preferences

### Communication Style
- Avoid exclamatory phrases like "Perfect!" or "Excellent!"
- Use direct, matter-of-fact language
- Reflect on requests and push back if you think I'm headed in the wrong direction
- Explain your reasoning when you disagree with an approach

### Decision Making
- Question assumptions that seem risky or suboptimal
- Provide alternatives with pros/cons rather than just one recommendation
- Challenge me if an approach seems to solve the wrong problem or miss the actual goal

### Git Workflow
- Commits should have clear, descriptive messages explaining what changed and why
- Keep commits focused on a single feature or fix when practical
- Include `Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>` trailer in commits (unless explicitly asked not to)
- Use `git status` and review changes before committing to ensure only intended files are included

### GraXpert Integration
- ProcessMono runs GraXpert as an external command-line process (not via PixInsight's process plugin)
- GraXpert automatically appends `.xisf` extension to output filenames
- When calling GraXpert CLI, pass the base filename without extension in the `-output` parameter

---

## 📝 Recent Lessons Learned

**Issue:** Bulk reverts (via `git checkout` or undo) can wipe out hours of work.

**Solution:** Always create timestamped local backups before:
- Running git revert/reset commands
- Making wholesale file replacements
- Reverting to previous commits
- Bulk deletions or refactoring

This protects against accidental data loss while keeping git history clean.

**Issue:** GraXpert CLI `process.start()` returns false even on successful execution.

**Solution:** Don't rely on the return value to determine success; instead, check for the existence of output files and log details to the console.

**Issue:** Channel combination and LinearFit functions received incorrect parameters.

**Solution:** Always pass complete channel arrays (3 channels for RGB or SHO) to channel combination functions; refactor function names to be branch-agnostic (e.g., use "channel 1/2/3" instead of "S/H/O" only).
