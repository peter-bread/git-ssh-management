# git ssh management

How to manage multiple GitHub accounts with ssh on one machine.

This guide is for Linux and MacOS.

For Windows, filepaths will need to be modified. Would recommend using WSL, then this tutorial will work.

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
    email = personal@email.com

[core]
    sshCommand = ssh -i ~/.ssh/github-personal -F /dev/null
```

```bash
# ~/repos/work/.gitconfig

[user]
    email = work@org.com

[core]
    sshCommand = ssh -i ~/.ssh/github-work -F /dev/null
```

## `core.sshCommand` Explanation

**`[core]`:** This is a section in the Git configuration that is used for core settings that apply to the entire repository.

**`sshCommand`:** This is an option within the [core] section that allows you to specify a custom SSH command.

**`ssh -i /path/to/personal/key -F /dev/null`:** This is the custom SSH command itself. Let's break it down:

> **`ssh`:** The base SSH command.
>
> **`-i /path/to/personal/key`:** Specifies the path to the private key that Git should use when connecting via SSH. In this case, it points to a specific private key file associated with your personal account.
>
> **`-F /dev/null`:** Specifies the configuration file to use for SSH. `/dev/null` is a special file that essentially means "no configuration." This is useful when you want to use a specific key (`-i`) without any additional configuration. It ensures that no SSH configuration file is read, including `~/.ssh/config`, and the key specified with `-i` is used directly.
