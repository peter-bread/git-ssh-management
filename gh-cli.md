# gh management

How to manage multiple GitHub accounts with [GitHub CLI](https://cli.github.com/) (gh v2.40 onwards).

## Prerequisites

This guide is for Linux and MacOS.

You use bash or zsh.

This assumes that we have the same accounts and file structure as in [the first part](./README.md) of this guide.

You have `yq` installed.

## Installation

See the offical docs [here](https://github.com/cli/cli#installation).

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

The `Active account` is the one that is currently active (obviously).

## Automatic Switching

We want to change the active account based on a context. In this case, we will do it based on the current directory we are in, and if it is either `~/repos/personal/` or `~/repos/work/` it will switch to the correct account.

To do this, add the following functions to your shell configuration file (`~/.bashrc`, `~/.zshrc`, etc):

```bash
# Switch on pwd (bash | zsh):
gh_auth_switch_on_pwd() {
  current_account=$(yq -r '.["github.com"].user' "$HOME/.config/gh/hosts.yml")

  account_names=("work" "personal")

  for account_name in "${account_names[@]}"; do
    if [[ "$PWD" == "$HOME/repos/$account_name"* && $current_account != "$account_name" ]]; then
      gh auth switch --user "$account_name"
    fi
  done
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
