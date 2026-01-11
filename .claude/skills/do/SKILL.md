---
name: do
description: "Personal to-do list manager with GitHub sync. Add tasks with natural language: '/do buy groceries tomorrow priority:high #shopping'. Auto-commits to GitHub for mobile access."
---

# To-Do List Manager

Manage your tasks with automatic GitHub sync for phone access.

## Commands

| Command | Example | Description |
|---------|---------|-------------|
| `/do [task]` | `/do buy groceries tomorrow` | Add a new task |
| `/do list` | `/do list` or `/do list #work` | Show pending tasks |
| `/do done [id or text]` | `/do done 1` or `/do done groceries` | Complete a task |
| `/do delete [id or text]` | `/do delete 3` | Remove a task |

## Adding Tasks

When adding a task, parse the input to extract:

1. **Task text**: Everything that isn't a modifier
2. **Due date**: Natural language dates
3. **Priority**: `priority:X` or `p:X` where X is high/medium/low (or h/m/l)
4. **Tags**: Words starting with `#`

### Date Patterns
- `tomorrow` → next day
- `today` → current day
- Day names: `monday`, `friday`, etc. → next occurrence
- `in X days` → X days from now
- Month + day: `jan 15`, `february 20`
- ISO format: `2026-01-15`

### Priority Patterns
- `priority:high` or `p:high` or `p:h` → high
- `priority:medium` or `p:medium` or `p:m` → medium
- `priority:low` or `p:low` or `p:l` → low
- Default if not specified: medium

### Examples

```
/do buy groceries tomorrow priority:high #shopping
→ Text: "buy groceries", Due: tomorrow, Priority: high, Tags: [shopping]

/do review PR friday #work #code
→ Text: "review PR", Due: next Friday, Priority: medium, Tags: [work, code]

/do clean garage p:low
→ Text: "clean garage", Due: none, Priority: low, Tags: []
```

## Workflow

For every task operation:

1. **Read** `tasks/tasks.json` to get current state
2. **Modify** the tasks array (add, complete, or delete)
3. **Update metadata** (counts, lastUpdated timestamp)
4. **Regenerate** `tasks/tasks.md` from the JSON data
5. **Git commit and push**:
   ```bash
   git add tasks/tasks.md tasks/tasks.json
   git commit -m "task: [action] - [summary]"
   git push origin main
   ```
6. **Confirm** to user with the updated task list

## File Locations

- **Tasks JSON**: `/Users/shayanmirzazadeh/Documents/Vibes/To Do/tasks/tasks.json`
- **Tasks Markdown**: `/Users/shayanmirzazadeh/Documents/Vibes/To Do/tasks/tasks.md`
- **GitHub Repo**: https://github.com/shayancelso/to-do

## JSON Structure

```json
{
  "version": "1.0",
  "lastUpdated": "2026-01-11T16:30:00Z",
  "tasks": [
    {
      "id": "task_001",
      "text": "Buy groceries",
      "status": "pending",
      "priority": "high",
      "dueDate": "2026-01-12",
      "createdAt": "2026-01-11T14:00:00Z",
      "completedAt": null,
      "tags": ["shopping"]
    }
  ],
  "metadata": {
    "totalTasks": 1,
    "pendingCount": 1,
    "completedCount": 0
  }
}
```

## Markdown Format

Generate `tasks.md` grouped by priority:

```markdown
# To-Do List

*Last updated: Jan 11, 2026 4:30 PM*

---

## High Priority

- [ ] **Buy groceries** `#shopping` - Due: Jan 12

## Medium Priority

- [ ] **Review PR** `#work` `#code` - Due: Jan 17

## Low Priority

- [ ] **Clean garage**

---

## Completed

- [x] ~~Submit report~~ - Completed: Jan 10
```

## Commit Messages

- Add: `task: add - Buy groceries (due: Jan 12, priority: high)`
- Complete: `task: complete - Submit report`
- Delete: `task: delete - Old reminder`

## ID System

Tasks display with numbers 1, 2, 3... ordered by priority (high first), then by due date.

For `/do done` or `/do delete`:
- Use number: `/do done 1` → completes task #1
- Use text match: `/do done groceries` → completes task containing "groceries"

## Error Handling

- **No match**: "No task found matching '[query]'. Current tasks: [list]"
- **Multiple matches**: "Multiple tasks match '[query]': [list]. Use the number to be specific."
- **Git push fails**: "Task saved locally. Git push failed: [error]. Run `git push` to sync."

## Listing Tasks

- `/do list` → Show all pending tasks grouped by priority
- `/do list #tag` → Filter by tag
- `/do list all` → Include completed tasks
