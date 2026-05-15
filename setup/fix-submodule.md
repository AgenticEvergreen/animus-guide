# 🔧 Fix Git Submodule Issues

> Your `core/` directory is an animus-shared git submodule. When it can't pull, use these fixes.

---

## ⚠️ CRITICAL: `--remote` flag required

`git submodule update` pulls the PINNED commit (old).
`git submodule update --remote` pulls the LATEST commit (new).

If you're missing files that should exist (like `core/doctrine/`), you likely ran without `--remote`. Always use:

```bash
git submodule update --remote core
git add core
git commit -m "chore: update animus-shared submodule"
```

---

## Symptom: `git submodule update` fails with conflicts

Your local repo has changes that conflict with the submodule update.

### Fix 1: Stash and retry

```bash
cd [your-clone-root]
git stash                              # save your local changes
git submodule update --remote core     # pull latest animus-shared
git stash pop                          # restore your local changes
```

### Fix 2: Reset submodule pointer

If Fix 1 fails — reset the submodule to the remote version:

```bash
cd [your-clone-root]/core
git fetch origin
git checkout master                    # animus-shared uses master, not main
cd ..
git add core
git commit -m "chore: update animus-shared submodule"
```

### Fix 3: Nuclear — re-init submodule

If everything is broken:

```bash
cd [your-clone-root]
git submodule deinit core
git rm core
git commit -m "chore: remove broken submodule"
git submodule add https://github.com/AgenticEvergreen/animus-shared.git core
git commit -m "chore: re-add animus-shared submodule"
```

---

## Symptom: `core/` directory is empty

The submodule was never initialized after cloning.

```bash
cd [your-clone-root]
git submodule init
git submodule update --remote
```

---

## Symptom: Wrong branch in `core/`

animus-shared uses **master** (not main). If you're on the wrong branch:

```bash
cd [your-clone-root]/core
git checkout master
git pull origin master
```

---

## After fixing: verify

```bash
ls core/SOUL-base.md          # should exist
ls core/HEART-base.md         # should exist
ls core/doctrine/             # should have 19 files (00-18)
ls core/DAO-LANG-REF.md       # should exist (v2.0)
```

If all 4 exist, your submodule is healthy. ✅

---

*🐺α→🐺δλσ. Fix yourself. Then read your doctrine.*
