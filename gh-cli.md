# gh management

***I am currently working on a [CLI tool](https://github.com/peter-bread/gamon) to manage this along with some additonal functionality.***

Currently, this tool does what this script does.

I intend to add more functionality.

---

How to manage multiple GitHub accounts with [GitHub CLI](https://cli.github.com/) (gh v2.40 onwards).

## Prerequisites

### System Requirements

| Category | Requirement |
| --- | --- |
| OS | MacOS, Linux |
| Shell | bash, zsh |

### Software Requirements

| Software | Link |
| --- | --- |
| gh | [Installation](https://github.com/cli/cli?tab=readme-ov-file#installation) |
| yq | [Installation](https://github.com/mikefarah/yq?tab=readme-ov-file#install) |

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
# This script is intended to be sourced from a .bashrc or .zshrc file.
# It uses features that are specific to bash and zsh, and may not work correctly in other shells.

gh_auth_switch_on_pwd() {

  # check if gh is installed
  trap 'GH_INSTALLED=' ERR

  if [ -z "$GH_INSTALLED" ]; then
    if ! command -v gh &> /dev/null; then
      echo "Error: gh is not installed" >&2
      return 1
    fi
    GH_INSTALLED=1
  fi

  # check if yq is installed
  trap 'YQ_INSTALLED=' ERR

  if [ -z "$YQ_INSTALLED" ]; then
    if ! command -v yq &> /dev/null; then
      echo "Error: yq is not installed" >&2
      return 1
    fi
    YQ_INSTALLED=1
  fi

  # check if yq is the Go version
  yq_version=$(yq --version)
  if [[ $yq_version != *"https://github.com/mikefarah/yq"* ]]; then
    echo "Error: Incorrect version of yq installed. Please install the Go version of yq." >&2
    return 1
  fi

  # check if gh config files are in default location
  # ensure config_dir does not end with a slash
  if [[ -n "$GH_CONFIG_DIR" ]]; then
    config_dir="${GH_CONFIG_DIR%/}"
  else
    config_dir="$HOME/.config/gh"
  fi

  # Check if config_dir exists and is a valid directory
  if [[ ! -d "$config_dir" ]]; then
    echo "Error: $config_dir could not be found" >&2
    return 1
  fi

  hosts="hosts.yml"

  # Get the current account and list of accounts in one go
  if ! accounts_info=$(yq -r '.["github.com"]' "$config_dir/$hosts") 2>/dev/null; then
    echo "Error: Could not find accounts in hosts.yml" >&2
    return 1
  fi

  # get current account
  if ! current_account=$(echo "$accounts_info" | yq -r '.user') 2>/dev/null; then
    echo "Error: Could not find current account in hosts.yml" >&2
    return 1
  fi

  # check if current_account is empty
  if [ -z "$current_account" ]; then
    echo "Error: Current account in hosts.yml is empty" >&2
    return 1
  fi

  # get accounts registered with gh
  if ! account_names=$(echo "$accounts_info" | yq eval '.users | keys') 2>/dev/null; then
    echo "Error: Could not find accounts in hosts.yml" >&2
    return 1
  fi

  # check if account_names is empty
  if [ -z "$account_names" ]; then
    echo "Error: Accounts in hosts.yml is empty" >&2
    return 1
  fi

  # format (get rid of `- ` at the start of each line)
  account_names=${account_names//- /}

  # check if account_names still contains `- `
  if [[ $account_names == *"- "* ]]; then
    echo "Error: Failed to format account names" >&2
    return 1
  fi

  # switch account if current directory is in a different account
  while IFS= read -r account_name; do
    if [[ "$PWD" == "$HOME/repos/$account_name"* && "$current_account" != "$account_name" ]]; then
      if ! gh auth switch --user "$account_name"; then
        echo "Error: Could not switch to account $account_name" >&2
        return 1
      fi
      break # break after switching account
    fi

    # handle error if mkdir fails
    if ! mkdir -p "$HOME/repos/$account_name"; then
      echo "Error: Could not create directory $HOME/repos/$account_name" >&2
      return 1
    fi

  done <<< "$account_names"

}

# get current shell
current_shell=$(ps -p $$ -ocomm=)

# calls function to check and/or switch github account on every cd
if [[ $current_shell == *"zsh"* ]]; then
  autoload -U add-zsh-hook
  add-zsh-hook chpwd gh_auth_switch_on_pwd
elif [[ $current_shell == *"bash"* ]]; then
  cd() {
    builtin cd "$@"
    gh_auth_switch_on_pwd
  }
else
  echo "Error: Unsupported shell. Only zsh and bash are supported." >&2
  return 1
fi

```

> **Note:**
> `gh` requires an internet connection.
>
> If you use `cd` and the script tries to run `gh auth switch --user <username>` without an internet connection it will throw an error.
