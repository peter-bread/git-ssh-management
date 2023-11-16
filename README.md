# git ssh management

How to manage multiple GitHub accounts with ssh on one machine.

This guide is for Linux and MacOS.

For Windows, filepaths will need to be modified and there will be extra steps in getting bash scripts to run. Would recommend using WSL, then this tutorial will work.

In this let's assume we have two GitHub accounts, personal and work.

Suppose our emails are:

```text
personal: personal@email.com
work    : work@org.com
```

## Set up File Structure

We will have a directory for each account:

```text
~/
    |–– .gitconfig
    |–– repos/
    |   |–– personal/
    |   |   |–– .gitconfig
    |   |–– work/
    |   |   |–– .gitconfig
```

Repos on our personal GitHub will be stored in `~/repos/personal/`.

Repos on our work GitHub will be stored in `~/repos/work/`.

> On Linux: `~/` is the same as `/home/username/`.
>
> On MacOS: `~/` is the same as `/Users/username/`.

## Add Stuff to `.gitconfig` Files

```bash
# ~/.gitconfig

[user]
    name = John Smith

[init]
    defaultBranch = main

[includeIf "gitdir:~/repos/personal/"]
    path = ~/repos/personal/.gitconfig

[includeIf "gitdir:~/repos/work/"]
    path = ~/repos/work/.gitconfig
```

```bash
# ~/repos/personal/.gitconfig

[user]
    email = "personal@email.com"
```

```bash
# ~/repos/work/.gitconfig

[user]
    email = "work@org.com"
```

## Generate ssh keys

```bash
ssh-keygen -t ed25519 -C "personal@email.com" -f ~/.ssh/github-personal

ssh-keygen -t ed25519 -C "work@org.com" -f ~/.ssh/github-work
```

> **Note:** keygen options:
>
> - the `-t` lets us specify the key type to be generated. (ed25519 is newer and more secure than the more common rsa key.)
>
> - the `-C` option lets us add a comment to the key so it is easier to keep track of.
>
> - the `-f` option lets us specify the name of the key and where we want to it be saved.

Our ssh directory will look like:

```text
~/
    |–– .ssh/
        |–– github-personal
        |–– github-personal.pub
        |–– github-work
        |–– github-work.pub
```

> **Note:** With this management technique we will not be using `~/.ssh/config` (for github at least, we can still use `~/.ssh/config` for other things, e.g. ssh remote connections).

## ssh-agent

We will load the correct key to the ssh agent whenever we enter the specified directory.

For example, when we change into `~/repos/work/` we want `~/.ssh/github-work` to be loaded.

When we leave the directory, the key is removed from the agent.

### Make Sure ssh-agent is Running

> On MacOS, the ssh-agent should start by default.
>
> We do not need to run the following command and do not need to put in our shell RC file.
>
> Skip to the [next step](#define-function-for-managing-keys-in-ssh-agent).

Run `eval $(ssh-agent -s)`. This starts the ssh-agent.

If we want the agent to be run in every session, we can add this command to wer shell RC file (e.g. `~/.bashrc`, `~/.zshrc`, etc). Make sure it is put it before the next code snippet. If we do this then we need to run `source ~/.bashrc` or `source ~/.zshrc` to reload `~/.bashrc` (or equivalent) and execute new changes (in this case starting the ssh-agent).

### Define Function for Managing Keys in ssh-agent

This function ensures there is only ever one key loaded in the agent at any time. When we enter a directory it checks if it is a sub directory of `~/repos/personal/` or `~/repos/work/`, and then adds the corresponding key to the agent. When we leave a valid directory, the key(s) are removed from the agent.

This function is not optimal but it is functional.

```bash
ssh_key_manager() {

        # Store the previous key (if any) to be removed
        previous_key=""

        if [[ "$PWD" == "$HOME/repos/work"* ]]; then
                previous_key="github-personal"
                current_key="github-work"
        elif [[ "$PWD" == "$HOME/repos/personal"* ]]; then
                previous_key="github-work"
                current_key="github-personal"
        fi

        # Remove the previous key (if any)
        if [ -n "$previous_key" ]; then
                ssh-add -d ~/.ssh/"$previous_key" 2>/dev/null
        fi

        if [[ "$PWD" == "$HOME/repos/work"* || "$PWD" == "$HOME/repos/personal"* ]]; then
                # Load the correct key
                ssh-add -l | grep -q "$current_key" || ssh-add ~/.ssh/"$current_key" 2>/dev/null
        else
                # If we go to non-repo directory remove any remaining keys
                ssh-add -D 2>/dev/null
        fi
}

# redefine cd to run `ssh_key_manager` every time it is used
cd() {
        builtin cd "$@"
        ssh_key_manager
}

# Load correct key if terminal starts in ~/Developer/*/
# e.g. when you laucnh terminal from within vscode
ssh_key_manager
```

> **Some  definitions:**
>
> `$PWD` stands for 'Print Working Directory', and it returns the current working directory.
>
> `$HOME` is the same as `~`.
>
> `ssh-add` outputs to `STDERR` (standard error output, which is associated with the number `2`). We don't want to be told about ssh keys being added and removed from the agent every time we use the `cd` command, so we redirect the output of `ssh-add` from `STDERR` to `/dev/null` (which discards any data written to it). We do this but putting `2>/dev/null` at the end of every `ssh-add` command.
