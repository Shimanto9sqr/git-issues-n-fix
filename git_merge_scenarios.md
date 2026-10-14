# Git Merge, Rebase, Squash, and Reflog Lab Notes

## Repository Initialization & Remote Configuration

```bash
mkdir git-hash-lab && cd git-hash-lab
git init -b main
echo "Base structure" > README.md
git add README.md 
git commit -m "chore: initial baseline commit"
git remote add origin git@github.com:Shimanto9sqr/git-issues-n-fix.git
git push -u origin main 

```

**Key Artifacts & References:**

* Commit: `98a597773bd8c5ab2096829b299df88e3829b239` (`origin/main`)
* `.git/HEAD` points to `refs/heads/main`

---

## Branching & Standard Non-Fast-Forward Merge (`--no-ff`)

### Feature Creation & Commits

```bash
git checkout -b feat/std-merge
echo "feat alpha" > alpha.txt
git add alpha.txt 
git commit -m "feat: implement feat alpha" # Commit: f152cfe

echo "Fix feat alpha" >> alpha.txt
git add alpha.txt 
git commit -m "fix: fix feature alpha"     # Commit: 0b25650

```

### Standard Merge Execution

```bash
git checkout main 
git merge feat/std-merge --no-ff -m "Merge PR#1 from feat/std-merge"

```

* **Merge Strategy:** `ort`
* **Merge Commit Created:** `26e047788596ffa9ff4bf17cb60080d175066fb7`
* **Parents:** `98a5977` (main base) and `0b25650` (feature head)

---

## Squash Merge (`--squash`)

```bash
git checkout -b feat/squash-merge
echo "Feat beta" > beta.txt
git add beta.txt 
git commit -m "feat: implement feat beta"  # Commit: fa03867

git checkout main 
git merge --squash feat/squash-merge 
git commit -m "feat: implement feat beta #2"

```

* **Outcome:** Changes from `feat/squash-merge` staged into working tree without copying history.
* **Resulting Main Commit:** `e51a988d4ce5a72ca7eb7ca4ee87802eebf61b1f` (linear single parent, no merge node created).

---

## Rebase & Fast-Forward Merge

### Divergent History Setup

```bash
# Branch setup
git checkout -b feat/rebase-merge
echo "Feat gamma" > gamma.txt
git add gamma.txt 
git commit -m "feat: implement feat gamma" # Original Commit: 4f93417

# Main branch advances independently
git checkout main 
echo "Team mates fix merged" >> README.md 
git add README.md 
git commit -m "fix: teammate fix"           # Commit: 3d83018

```

### Rebase & Fast-Forward Progression

```bash
git checkout feat/rebase-merge 
git rebase main                             # Rebased Commit generated: 90150a9

git checkout main 
git merge feat/rebase-merge                 # Fast-forward update: 3d83018..90150a9

```

* **Outcome:** `4f93417` rewritten onto `3d83018`, producing new commit hash `90150a9`.
* **Main Update:** Linear fast-forward update to `90150a9`.

---

## Reflog Exploration & Reset Testing

### Fast-Forward vs Explicit Merge Experimentation

```bash
git checkout -b feat/tree
echo "Node 1" > node.txt && git add node.txt && git commit -m "test: node 1" # Commit: 25c7402
echo "Node 2" > node2.txt && git add node2.txt && git commit -m "test: node 2" # Commit: 6a27335

git checkout main
git merge feat/tree                         # Fast-forwarded to 6a27335

```

### Working Tree State Recovery via Reflog and Reset

```bash
git reset --hard 90150a9                    # Rollback main before feat/tree merge
git reset --hard 6a27335                    # Move main back forward to 6a27335
git reset 90150a9                           # Mixed reset (keeps changes untracked)
git reset --hard 90150a9                    # Clean state back to 90150a9

# Final Merge Choice (--no-ff)
git merge feat/tree --no-ff -m "Merge pr#3 from feat/tree"

```

* **Final Merge Commit:** `a0ae1092c8fc98cce02f022ae8dc3ba9ee1d8724`

---

## Final Repository Commit Structure

```text
*   a0ae109 (HEAD -> main) Merge pr#3 from feat/tree
|\  
| * 6a27335 (feat/tree) test: node 2
| * 25c7402 test: node 1
|/  
- 90150a9 (feat/std-merge, feat/rebase-merge) feat: implement feat gamma
- 3d83018 fix: teammate fix
- e51a988 feat: implement feat beta #2
*   26e0477 Merge PR#1 from feat/std-merge
|\  
| * 0b25650 fix: fix feature alpha
| * f152cfe feat: implement feat alpha
|/  
- 98a5977 (origin/main) chore: initial baseline commit

```

---

## Key Reflog Entries for Tracing Context

```text
a0ae109 HEAD@{0}: merge feat/tree: Merge made by the 'ort' strategy.
90150a9 HEAD@{1}: reset: moving to 90150a9
6a27335 HEAD@{2}: reset: moving to 6a27335
90150a9 HEAD@{3}: reset: moving to 90150a9
90150a9 HEAD@{10}: merge feat/rebase-merge: Fast-forward
26e0477 HEAD@{24}: merge feat/std-merge: Merge made by the 'ort' strategy.

```