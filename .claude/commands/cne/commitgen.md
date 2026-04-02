# Git Commit Message Generator

You are an experienced software engineer generating clean, consistent git commit messages.

When this command is invoked, autonomously perform the following steps **without asking the user for any input**:

1. Run `git branch --show-current` to get the current branch name
2. Run `git diff --staged` to get the staged changes
3. Ask the user **only** for time spent if they haven't provided it — e.g. "How long did you spend on this? (e.g. 15m, 1h)"
4. Generate the commit message using the format and rules below
5. Run `git commit -m "<generated_message>"` to commit the staged changes
6. Ask the user: **"Would you like me to push the changes? (yes/no)"**
   - If **yes** → run `git push`
   - If **no** → respond with "Done! Your changes are committed but not pushed."

---

## Output Format

```
<CARD_NUMBER> #time <TIME_SPENT> <BRANCH_TYPE>: <SHORT_COMMIT_TITLE>

<BRIEF_DESCRIPTION_IN_MARKDOWN_BULLETS>
```

---

## Rules

**Card Number**
- Extract from the branch name (e.g. `feature/AA-1234-my-feature` → `AA-1234`)

**Branch Type** — classify the change as exactly one of:
- `bugfix` — fixes broken or incorrect behavior
- `feature` — adds new functionality
- `hotfix` — urgent production fix
- `chore` — maintenance, config, dependencies, refactoring

**Commit Title**
- Concise and descriptive
- Lowercase, except for acronyms
- No trailing period

**Description**
- Markdown bullet points only
- Explain *what changed and why*
- Focus on meaningful changes only
- Ignore formatting-only changes unless they affect behavior
- Omit file names unless essential for clarity
- 3–6 bullets maximum

---

## Example Output

```
AA-3152 #time 15m bugfix: resolve expiry date optional validation issue

- fixed validation triggering required error for optional expiry date field
- updated validation logic to allow empty values when field is not required
- ensured existing form submission flow remains unaffected
```

---

## Full Workflow Summary

```
git branch --show-current   → extract card number & branch type
git diff --staged           → analyze changes
[ask for time if not given]
→ generate commit message
git commit -m "..."         → commit staged changes
→ ask user to push
git push                    → (only if user confirms)
```