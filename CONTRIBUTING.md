# Contributing

Contributions are welcome. Please follow these guidelines to keep the repo consistent.

## Commit Messages

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>: <short summary>
```

Common types:

| Type     | When to use                                      |
|----------|--------------------------------------------------|
| `docs`   | Add or update documentation / guides            |
| `fix`    | Correct broken links, typos, or factual errors  |
| `feat`   | Add a new guide, section, or resource           |
| `chore`  | Maintenance (CI config, `.gitattributes`, etc.) |

Examples:
```
docs: add 9front Bluetooth setup guide
fix: correct broken link to cat-v.org resource
feat: add Harvey OS benchmarking notes
chore: add GitHub Actions link checker
```

## Adding Content

- Place new guides in their own subdirectory with a `README.md` (uppercase).
- Update the root `README.md` to link to the new guide.
- Prefer relative links for internal references.

## Images & Binary Files

This repo uses **Git LFS** for images and other binary files. Before adding large assets:

1. Install Git LFS: `git lfs install`
2. Add your file — the `.gitattributes` rules will route it to LFS automatically.

Avoid committing binary files larger than 1 MB without LFS. If you don't have LFS set up, note it in your PR and a maintainer can migrate the asset.

## External Links

A link-checking workflow runs on every push and weekly on Mondays. Before opening a PR, verify that any external links you add are reachable.
