# Tutorial 03: Advanced Usage - Parallel Development

**Read time: 3-5 minutes**

Scale up development by running multiple AI agents in parallel, each working on different issues simultaneously.

## Choose Your Workflow

**Use multiple clones when:**
- Working across different machines
- Prefer complete isolation between workers
- Each worker needs independent fetch/push operations

**Use single repo with worktrees when:**
- Disk space is limited (worktrees share `.git`)
- Working on same machine with multiple terminals
- Want faster setup without re-cloning

Both approaches use the `wt` CLI—pick based on your isolation needs.

## When to Use Parallel Development

**Good for:**
- Multiple independent features
- Large refactoring split into separate issues
- Documentation updates + feature work
- Bug fixes that don't touch the same files

**Avoid when:**
- Issues modify the same files (high conflict risk)
- Issues have dependencies on each other
- You're new to the framework (start with Tutorial 02 first)

## Approach 1: Repository Clones

### Setup

Use `wt clone` to create bare repositories with worktree environments:

```bash
# Source wt.sh for shell integration
source src/cli/wt.sh

# Create parallel workers (each is a bare repo with trees/main)
cd ~/projects
wt clone https://github.com/your-org/my-project.git my-project-worker-1.git
wt clone https://github.com/your-org/my-project.git my-project-worker-2.git
wt clone https://github.com/your-org/my-project.git my-project-worker-3.git
```

Each `wt clone` creates a bare repository and initializes `trees/main` automatically.

### Workflow

Spawn issue worktrees in each clone. Use `--yolo` to skip permission prompts:

**Terminal 1 (Worker 1 - Issue #45):**
```bash
cd ~/projects/my-project-worker-1.git
wt spawn 45 --yolo
# Claude starts automatically in trees/issue-45-*/
```

**Terminal 2 (Worker 2 - Issue #46):**
```bash
cd ~/projects/my-project-worker-2.git
wt spawn 46 --yolo
# Claude starts automatically in trees/issue-46-*/
```

**Terminal 3 (Worker 3 - Issue #47):**
```bash
cd ~/projects/my-project-worker-3.git
wt spawn 47 --yolo
# Claude starts automatically in trees/issue-47-*/
```

Each AI works independently. Resume milestones with `wt goto <issue>` then start claude-code.

### Cleanup

After merging PRs:

```bash
# Remove specific issue worktrees
wt remove 45 --delete-branch
wt remove 46 --delete-branch

# Or delete entire worker repos
rm -rf ~/projects/my-project-worker-*.git
```

## Approach 2: Git Worktrees

Worktrees share the `.git` directory while providing isolated working directories—saves disk space.

### Setup

Initialize once, then spawn worktrees for each issue:

```bash
# Source wt.sh for shell integration
source src/cli/wt.sh

# First-time setup: Initialize worktree environment
cd ~/projects/my-project.git
wt init

# Create worktree (fetches title from GitHub, starts Claude)
wt spawn 42
# Creates: trees/issue-42-<title>/
# Branch: issue-42-<title>
```

The `spawn` command automatically:
- Creates `trees/issue-<N>-<title>/` (gitignored)
- Creates branch following naming convention
- Invokes Claude in the worktree

### Workflow

Use `--yolo` to skip permission prompts, `--headless` for non-interactive mode:

**Terminal 1 (Issue #45):**
```bash
cd ~/projects/my-project.git
wt spawn 45 --yolo
# Claude starts automatically in trees/issue-45-*/
```

**Terminal 2 (Issue #46):**
```bash
cd ~/projects/my-project.git
wt spawn 46 --yolo
# Claude starts automatically in trees/issue-46-*/
```

Each worktree operates independently on its own branch.

### Important: Path Rules

Each worktree is its own "project root" for path resolution. All paths are relative to the active worktree:
- ✅ `docs/tutorial/03-advanced-usage.md` (relative to worktree root)
- ❌ `../main-repo/docs/...` (crossing worktree boundaries)

The `CLAUDE.md` rule "DO NOT use `cd`" applies within each worktree individually.

### Cleanup

```bash
# Remove specific worktree (keeps branch)
wt remove 42

# Remove worktree and delete branch
wt remove 42 --delete-branch

# List all worktrees
wt list

# Clean up stale metadata
wt prune

# Remove all worktrees for closed issues
wt purge
```

## Managing Progress

### Track Assignments

Keep simple notes on which worker/worktree handles which issue:

```
Worker 1 / trees/issue-45-*: Issue #45 - Rust SDK
Worker 2 / trees/issue-46-*: Issue #46 - Docs update
Worker 3 / trees/issue-47-*: Issue #47 - Performance fix
```

### Resume After Milestones

If a worker creates a milestone, resume in the same worktree:

```bash
# Navigate to worktree
wt goto 45

# Start Claude and resume
claude-code
# User: Continue from the latest milestone
```

## Avoiding Conflicts

### Plan for Independence

Design issues to avoid file overlap:
- ✅ Issue #45 modifies `templates/rust/`
- ✅ Issue #46 modifies `docs/`
- ✅ Issue #47 modifies `src/performance.c`

### Stagger Merges

Don't merge all PRs at once:

1. Complete first worker/worktree
   - `/code-review`
   - `/sync-master`
   - Create and merge PR

2. Update others to latest main
   ```bash
   git checkout main
   git pull origin main
   git checkout issue-46-*
   git rebase main
   ```

3. Repeat review and merge for each remaining issue

### Resolve Conflicts

If conflicts occur during rebase:

```bash
git rebase main
# CONFLICT (content): Merge conflict in src/main.c

# Fix in editor, then:
git add src/main.c
git rebase --continue
```

## Best Practices

1. **Limit workers**: 3-4 parallel is manageable, more gets chaotic
2. **Name clearly**: Use descriptive directory/worktree names
3. **Track assignments**: Keep notes on which worker has which issue
4. **Sync before PR**: Always `/sync-master` before creating PRs
5. **Review first**: Always `/code-review` before merge
6. **Start small**: Try 2 parallel issues before scaling up

## When to Use Sequential vs Parallel

**Use sequential (Tutorial 02) when:**
- Learning the framework
- Issues touch the same code
- Issues depend on each other

**Use parallel (this tutorial) when:**
- Issues are independent
- Comfortable with the workflow
- Want to maximize throughput

## Next Steps

You've completed all tutorials! You now know how to:
- ✅ Initialize Agentize (Tutorial 00)
- ✅ Plan issues (Tutorial 01)
- ✅ Implement features (Tutorial 02)
- ✅ Scale with parallel development (Tutorial 03)

Explore the full documentation:
- `.claude/commands/*.md` - All available commands
- `.claude/skills/*/SKILL.md` - How skills work
- `docs/milestone-workflow.md` - Deep dive on milestones
- `README.md` - Architecture and philosophy
