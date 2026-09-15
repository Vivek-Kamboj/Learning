# 📦 Git Stash Commands Guide

This document provides a quick reference for commonly used `git stash` commands.

---

## 🔹 What is Git Stash?

`git stash` temporarily saves your uncommitted changes so you can switch branches or work on something else without committing.

---

## 🧰 Common Git Stash Commands

### 1. Save Changes to Stash

```bash
git stash
```

Stashes tracked modified files.

#### Include untracked files:

```bash
git stash -u
```

#### Include ignored files:

```bash
git stash -a
```

#### Add a message:

```bash
git stash push -m "your message"
```

---

### 2. List Stashes

```bash
git stash list
```

---

### 3. Apply Stash (Keep It)

```bash
git stash apply
```

#### Apply specific stash:

```bash
git stash apply stash@{0}
```

---

### 4. Apply and Remove Stash

```bash
git stash pop
```

#### Specific stash:

```bash
git stash pop stash@{0}
```

---

### 5. Show Stash Details

```bash
git stash show
```

#### Full diff:

```bash
git stash show -p
```

---

### 6. Delete a Stash

```bash
git stash drop stash@{0}
```

---

### 7. Delete All Stashes

```bash
git stash clear
```

---

### 8. Create Branch from Stash

```bash
git stash branch branch-name
```

---

## ⚖️ Apply vs Pop

| Command           | Applies Changes | Removes Stash |
| ----------------- | --------------- | ------------- |
| `git stash apply` | ✅               | ❌             |
| `git stash pop`   | ✅               | ✅             |

---

## ⚡ Example Workflow

```bash
git stash
git checkout main
git checkout feature-branch
git stash pop
```

---

## 🧠 Tips

* Use `apply` if you want to keep the stash.
* Use `pop` when you are done with the stash.
* Use messages to identify stashes easily.

---
