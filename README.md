# git-user

A CLI tool to seamlessly switch between multiple Git and GitHub identities. Manage personal, work, and other profiles with a single command that updates your git config, GitHub CLI auth, and SSH routing all at once.

## The Problem

If you use multiple GitHub accounts (personal, work, open-source org), switching between them is painful. You have to manually:

- Change `git config --global user.name` and `user.email`
- Run `gh auth switch`
- Swap SSH keys

**git-user** handles the global identity switch with one command.

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
# → prompts for: name, email, GitHub username, org name / repo owner, SSH key (optional), remote host

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
3. **SSH alias** — If the profile has an SSH key configured, creates or updates a managed SSH host alias and activates global Git URL rewrites so existing `git@github.com:owner/repo.git` remotes use that profile's key. If not, it clears any previous managed SSH routing.
4. **Remote owner** — If you're inside a git repo and it has an `origin`, updates the owner part to the profile's configured org name or repo owner. This defaults to the profile's GitHub username.

```
$ git-user switch work

  Switching to profile: work

  ✔ git config → John Smith <john@company.com>
  ✔ gh auth → switched to @john-company
  ✔ SSH alias → git-user-work-github-com (~/.ssh/id_ed25519_work)
  ✔ remote origin → git@github.com:megacorp/company-api.git

  ✔ Switched to 'work'
```

## Configuration File

All profiles are stored in `~/.git_user.conf` as a simple key-value format:

```ini
# ~/.git_user.conf
personal.name=John Doe
personal.email=john@gmail.com
personal.gh_user=johndoe
personal.org_name=johndoe
personal.remote_host=github.com

work.name=John Smith
work.email=john@company.com
work.gh_user=john-company
work.org_name=megacorp
work.ssh_key=~/.ssh/id_ed25519_work
work.remote_host=github.com

_active=personal
```

### Profile Fields

| Field | Required | Description |
|---|---|---|
| `name` | ✔ | Git author name (`user.name`) |
| `email` | ✔ | Git author email (`user.email`) |
| `gh_user` | ✔ | GitHub username for `gh` CLI auth |
| `org_name` | | Repo owner to use for `origin` updates. Defaults to `gh_user`. |
| `ssh_key` | | Path to SSH private key used for the managed SSH alias |
| `remote_host` | | GitHub host for `gh auth` and the SSH alias (default: `github.com`) |

### Custom Config Location

Override the config path with the `GIT_USER_CONF` environment variable:

```bash
export GIT_USER_CONF="$HOME/.config/git-user/profiles.conf"
git-user list
```

Override the SSH config path with `GIT_USER_SSH_CONFIG`:

```bash
export GIT_USER_SSH_CONFIG="$HOME/.config/ssh/git-user-config"
git-user switch personal
```

## Examples

### Setting Up Personal + Work Profiles

```bash
$ git-user add personal
  Full name: John Doe
  Email: john@gmail.com
  GitHub username: johndoe
  Org name / repo owner [johndoe]:
  SSH key path (optional): ~/.ssh/id_ed25519
  Remote host (optional) [github.com]:

✔ Profile 'personal' added successfully!

$ git-user add work
  Full name: John Smith
  Email: john.smith@megacorp.com
  GitHub username: john-megacorp
  Org name / repo owner [john-megacorp]: megacorp
  SSH key path (optional): ~/.ssh/id_ed25519_work
  Remote host (optional) [github.com]:

✔ Profile 'work' added successfully!
```

### Switching Before Starting Work

```bash
$ cd ~/projects/company-api
$ git-user switch work
# → git config, gh auth, SSH alias routing, and origin owner all updated

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
  Org name / repo owner [jsmith]: engineering
  SSH key path (optional):
  Remote host (optional) [github.com]: github.corp.internal

✔ Profile 'enterprise' added successfully!
```

## Tips

- **Shell alias** — Add `alias gu="git-user"` to your `.bashrc`/`.zshrc` for faster switching.
- **Multiple SSH keys** — Each profile can point to a different SSH key. `switch` manages per-profile SSH host aliases in `~/.ssh/config` and updates Git's global `url.*.insteadOf` rules so standard GitHub SSH remotes follow the active profile.
- **Edit the config directly** — The `.git_user.conf` file is human-readable; feel free to edit it by hand.

## License

MIT
