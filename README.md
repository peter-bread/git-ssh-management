# git ssh management <!-- omit in toc -->

***I am currently working on a [CLI tool](https://github.com/peter-bread/gamon) to manage this along with some additonal functionality.***

As of right now, this tool depends on the file structure detailed here being implented manually.

In the future, I hope to make generating this file structure automatic.

In the even further future, I hope to make this file structure more flexible, but that is not my priority at the moment.

---

How to manage multiple GitHub accounts with ssh on one machine.

> **Note:** This is an updated solution.
>
> [Click here](./README.old.md) to see old solution.
>
> [Click here](./change.md) to see explanation of differences and reasons between the two versions.

## Table of Contents <!-- omit in toc -->

- [Prerequisites](#prerequisites)
- [Our Accounts](#our-accounts)
- [Set up File Structure](#set-up-file-structure)
- [Generate ssh keys](#generate-ssh-keys)
- [Add Stuff to `.gitconfig` Files](#add-stuff-to-gitconfig-files)
- [gh](#gh)

## Prerequisites

Before you begin, ensure you have met the following requirements:

- You have installed the latest version of [Git](https://git-scm.com/downloads).
- You have a Linux or MacOS machine. Windows users can use WSL.
- You have read the [guide on how to set up multiple accounts with GitHub CLI](./gh-cli.md).

## Our Accounts

In this let's assume we have two GitHub accounts, *personal* and *work*.

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

To be clear:

> You will only be able to clone repositories with **ssh** when in `~/repos/personal` or `~/repos/work`. These should be repositories **owned by you**.
>
> ---
>
> If you want to build an app from source, for example, you can clone a public repository **owned by anyone** anywhere you like as long as you use **https**.
>
> If you have any other ssh key loaded in your ssh agent or with a default name (like `id_ed25519`), ssh may work for this, but I haven't tested it.

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

### `~/.gitconfig` <!-- omit in toc -->

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

### `~/repos/personal/.gitconfig` <!-- omit in toc -->

```bash
# ~/repos/personal/.gitconfig

[user]
    email = personal@email.com

[core]
    sshCommand = ssh -i ~/.ssh/github-personal -F /dev/null
```

### `~/repos/work/.gitconfig` <!-- omit in toc -->

```bash
# ~/repos/work/.gitconfig

[user]
    email = work@org.com

[core]
    sshCommand = ssh -i ~/.ssh/github-work -F /dev/null
```

## `core.sshCommand` Explanation <!-- omit in toc -->

**`[core]`:** This is a section in the Git configuration that is used for core settings that apply to the entire repository.

**`sshCommand`:** This is an option within the [core] section that allows you to specify a custom SSH command.

**`ssh -i /path/to/personal/key -F /dev/null`:** This is the custom SSH command itself. Let's break it down:

> **`ssh`:** The base SSH command.
>
> **`-i /path/to/personal/key`:** Specifies the path to the private key that Git should use when connecting via SSH. In this case, it points to a specific private key file associated with your personal account.
>
> **`-F /dev/null`:** Specifies the configuration file to use for SSH. `/dev/null` is a special file that essentially means "no configuration." This is useful when you want to use a specific key (`-i`) without any additional configuration. It ensures that no SSH configuration file is read, including `~/.ssh/config`, and the key specified with `-i` is used directly.

## gh

[Click here](./gh-cli.md) to see how to set up multiple accounts with GitHub CLI.
