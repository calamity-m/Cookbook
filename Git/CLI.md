[[Dreambook/Void/Snippets|Snippets]]

# Display local config

```bash
git config --list --local
```

## Display global config

```bash
git config --list --global
```

# Set Global User/Email

```bash
git config --global user.name xyz
git config --global user.email xyz
```

# Delete all branches except main

```bash
git branch | grep -v "main" | xargs git branch -D
```

# Mirror repository

```bash
git clone --bare https://github.com/EXAMPLE-USER/OLD-REPOSITORY.git
cd OLD-REPOSITORY
git push --mirror https://github.com/EXAMPLE-USER/NEW-REPOSITORY.git
```