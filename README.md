# git-user

A CLI tool to seamlessly switch between multiple Git and GitHub identities. Manage personal, work, and other profiles with a single command that updates your git config, GitHub CLI auth, remote URLs, and SSH keys all at once.

## The Problem

If you use multiple GitHub accounts (personal, work, open-source org), switching between them is painful. You have to manually:

- Change `git config --global user.name` and `user.email`
- Run `gh auth switch`
- Update your repo's remote URL to point to the right owner
- Swap SSH keys

**git-user** does all of this with one command.

## Installation

```bash
# Download and install
curl -o /usr/local/bin/git-user https://raw.githubusercontent.com/<your-user>/git-user/main/git-user
chmod +x /usr/local/bin/git-user

# Or just copy manually
sudo cp git-user /usr/local/bin/git-user
chmod +x /usr/local/bin/git-user
```

### Requirements

- **bash** 4.0+
- **git**
- **gh** (GitHub CLI) — optional but recommended for auth switching

## Quick Start

```bash
# 1. Add your profiles
git-user add personal
# → prompts for: name, email, GitHub username, SSH key (optional), remote host

git-user add work

# 2. Switch profiles
git-user switch personal

# 3. See what's active
git-user list
```

## Commands

| Command | Aliases | Description |
|---|---|---|
| `git-user add <profile>` | | Create a new profile interactively |
| `git-user switch <profile>` | `use`, `set` | Activate a profile (see below) |
| `git-user list` | `ls` | List all profiles (● = active) |
| `git-user current` | `who`, `status` | Show the active profile and git config |
| `git-user edit <profile>` | | Modify an existing profile |
| `git-user remove <profile>` | `rm`, `delete` | Delete a profile (with confirmation) |
| `git-user help` | `--help`, `-h` | Show help message |
| `git-user version` | `--version`, `-v` | Print version |

## What `switch` Does

When you run `git-user switch <profile>`, it performs four actions:

1. **Git config** — Sets `user.name` and `user.email` globally
2. **GitHub CLI auth** — Runs `gh auth switch` to the profile's account (prompts `gh auth login` if not yet authenticated)
3. **Remote URL** — If you're inside a git repo, updates the `origin` remote to use the new profile's GitHub username
4. **SSH key** — If the profile has an SSH key configured, sets `core.sshCommand` to use it

```
$ git-user switch work

  Switching to profile: work

  ✔ git config → John Smith <john@company.com>
  ✔ gh auth → switched to @john-company
  ✔ remote origin → https://github.com/john-company/myproject.git
  ✔ SSH key → ~/.ssh/id_ed25519_work

  ✔ Switched to 'work'
```

## Configuration File

All profiles are stored in `~/.git_user.conf` as a simple key-value format:

```ini
# ~/.git_user.conf
personal.name=John Doe
personal.email=john@gmail.com
personal.gh_user=johndoe
personal.remote_host=github.com

work.name=John Smith
work.email=john@company.com
work.gh_user=john-company
work.ssh_key=~/.ssh/id_ed25519_work
work.remote_host=github.com

_active=personal
```

### Profile Fields

| Field | Required | Description |
|---|---|---|
| `name` | ✔ | Git author name (`user.name`) |
| `email` | ✔ | Git author email (`user.email`) |
| `gh_user` | ✔ | GitHub username for `gh` CLI and remote URLs |
| `ssh_key` | | Path to SSH private key (enables `core.sshCommand`) |
| `remote_host` | | Git remote hostname (default: `github.com`) |

### Custom Config Location

Override the config path with the `GIT_USER_CONF` environment variable:

```bash
export GIT_USER_CONF="$HOME/.config/git-user/profiles.conf"
git-user list
```

## Examples

### Setting Up Personal + Work Profiles

```bash
$ git-user add personal
  Full name: John Doe
  Email: john@gmail.com
  GitHub username: johndoe
  SSH key path (optional): ~/.ssh/id_ed25519
  Remote host (optional) [github.com]:

✔ Profile 'personal' added successfully!

$ git-user add work
  Full name: John Smith
  Email: john.smith@megacorp.com
  GitHub username: john-megacorp
  SSH key path (optional): ~/.ssh/id_ed25519_work
  Remote host (optional) [github.com]:

✔ Profile 'work' added successfully!
```

### Switching Before Starting Work

```bash
$ cd ~/projects/company-api
$ git-user switch work
# → git config, gh auth, remote URL, and SSH key all updated

$ git commit -m "feat: add endpoint"
# commit is authored as John Smith <john.smith@megacorp.com>
```

### Listing Profiles

```bash
$ git-user list

  Profiles  (~/.git_user.conf)

  ● personal  John Doe <john@gmail.com> @johndoe
    work      John Smith <john.smith@megacorp.com> @john-megacorp
```

### GitHub Enterprise / Self-hosted

Set `remote_host` to your enterprise instance:

```bash
$ git-user add enterprise
  Full name: John Smith
  Email: john@corp.internal
  GitHub username: jsmith
  SSH key path (optional):
  Remote host (optional) [github.com]: github.corp.internal

✔ Profile 'enterprise' added successfully!
```

## Tips

- **Shell alias** — Add `alias gu="git-user"` to your `.bashrc`/`.zshrc` for faster switching.
- **Per-repo profiles** — Run `switch` inside a repo to update its remote. Outside a repo, it only updates global git config and gh auth.
- **Multiple SSH keys** — Each profile can point to a different SSH key. `switch` automatically configures `core.sshCommand` so git uses the right one.
- **Edit the config directly** — The `.git_user.conf` file is human-readable; feel free to edit it by hand.

## License

MIT
