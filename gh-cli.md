# gh management

***I am currently working on a CLI tool to manage this along with some additonal functionality.***

It is private right now but will be going public soon.

---

How to manage multiple GitHub accounts with [GitHub CLI](https://cli.github.com/) (gh v2.40 onwards).

## Prerequisites

| Category | Requirement | Link |
| --- | --- | --- |
| OS | MacOS, Linux | - |
| Shell | bash, zsh | - |
| Other Software | gh | [Installation](https://github.com/cli/cli?tab=readme-ov-file#installation) |
| Other Software | yq | [Installation](https://github.com/mikefarah/yq?tab=readme-ov-file#install) |

This assumes that we have the same accounts and file structure as in [the first part](./README.md) of this guide.

## Signing in

> **Note:**
> `gh` requires an internet connection.

After installing gh, we need to sign in to both of our accounts.

In a terminal, run:

```bash
gh auth login
```

Follow the on screen prompts.

Do this for both accounts.

Run `gh auth status` to see the status of your accounts. The output will look something like this:

```bash
❯ gh auth status
github.com
  ✓ Logged in to github.com account personal (keyring)
  - Active account: true
  - Git operations protocol: ssh
  - Token: gho_************************************
  - Token scopes: 'gist', 'read:org', 'repo'

  ✓ Logged in to github.com account work (keyring)
  - Active account: false
  - Git operations protocol: ssh
  - Token: gho_************************************
  - Token scopes: 'gist', 'read:org', 'repo'
```

Here, the account tokens are stored in Apple Keychain.

If they are stored in plain text, it may look something like this:

```bash
❯ gh auth status
github.com
  ✓ Logged in to github.com account personal (/home/users/username/.config/gh/hosts.yml)
  - Active account: true
  - Git operations protocol: ssh
  - Token: gho_************************************
  - Token scopes: 'gist', 'read:org', 'repo'

  ✓ Logged in to github.com account work (/home/users/username/.config/gh/hosts.yml)
  - Active account: false
  - Git operations protocol: ssh
  - Token: gho_************************************
  - Token scopes: 'gist', 'read:org', 'repo'
```

The `Active account` is the one that is currently active (obviously).

## Automatic Switching

We want to change the active account based on a context. In this case, we will do it based on the current directory we are in, and if it is either `~/repos/personal/` or `~/repos/work/` it will switch to the correct account.

To do this, add the following functions to your shell configuration file (`~/.bashrc`, `~/.zshrc`, etc):

```bash
# Switch on pwd (bash | zsh):
gh_auth_switch_on_pwd() {

  # These are not really necessary, but they make the script more robust ------

  # check if gh is installed
  if ! command -v gh &> /dev/null; then
    echo "Error: gh is not installed" >&2
    return 1
  fi

  # check if yq is installed
  if ! command -v yq &> /dev/null; then
    echo "Error: yq is not installed" >&2
    return 1
  fi

  # ---------------------------------------------------------------------------

  # check if gh config files are in default location
  if [[ -n "$GH_CONFIG_DIR" ]]; then
    # ensure GH_CONFIG_DIR ends with a slash
    config_dir="${GH_CONFIG_DIR%/}/"
  else
    config_dir="$HOME/.config/gh/"
  fi

  # Check if config_dir exists and is a valid directory
  if [[ ! -d "$config_dir" ]]; then
    echo "Error: $config_dir could not be found" >&2
    return 1
  fi

  hosts="hosts.yml"

  # get current account from hosts.yml
  if ! current_account=$(yq -r '.["github.com"].user' "$config_dir$hosts") 2>/dev/null; then
    echo "Error: Could not find current account in hosts.yml" >&2
    return 1
  fi

  # get accounts registered with gh from hosts.yml
  if ! account_names=$(yq eval '.["github.com"].users | keys' "$config_dir$hosts") 2>/dev/null; then
    echo "Error: Could not find accounts in hosts.yml" >&2
    return 1
  fi

  # format (get rid of `- ` at the start of each line)
  account_names=${account_names//- /}

  # switch account if current directory is in a different account
  while IFS= read -r account_name; do
    if [[ "$PWD" == "$HOME/repos/$account_name"* && "$current_account" != "$account_name" ]]; then
      if ! gh auth switch --user "$account_name"; then
        echo "Error: Could not switch to account $account_name" >&2
        return 1
      fi
      mkdir -p "$HOME/repos/$account_name"
    fi
  done <<< "$account_names"

}

# Switch on cd (bash | zsh):
cd() {
  builtin cd "$@"
  gh_auth_switch_on_pwd
}

# Switch on cd (zsh):
autoload -U add-zsh-hook
add-zsh-hook chpwd gh_auth_switch_on_pwd
```

> **Note:**
> `gh` requires an internet connection.
>
> If you use `cd` without an internet connection, you will get a warning form `gh` saying it couldn't connect.
> If this is annoying, you can alter the `cd` function to only call `gh_auth_switch_on_pwd` if there is a connection, and/or send stdout/stderr to `/dev/null`.
