# gh management

How to manage multiple GitHub accounts with [GitHub CLI](https://cli.github.com/) (gh v2.40 onwards).

This guide is for Linux and MacOS.

This assumes that we have the same accounts and file structure as in [the first part](./README.md) of this guide.

## Installation

See the offical docs [here](https://github.com/cli/cli#installation).

## Signing in

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
# Switch on pwd:
gh_auth_switch_on_pwd() {

  current_account=$(gh api /user | jq -r .login)

  if [[ "$PWD" == "$HOME/Developer/work"* ]]; then
    if [[ $current_account != "work" ]]; then
      gh auth switch --user work
    fi
  elif [[ "$PWD" == "$HOME/Developer/personal"* ]]; then
    if [[ $current_account != "personal" ]]; then
      gh auth switch --user personal
    fi
  fi

}

# Switch on cd:
cd() {
  builtin cd "$@"
  gh_auth_switch_on_pwd
}
```
