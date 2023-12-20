# Explanation of Change

## Original Solution

### Problem

Adding personal and work keys to ssh agent did not work correctly. The agent would always use the first key loaded at the top of the list after running `ssh-add -l`.

### Solution

Defined a bash function that would be called every time `cd` is used. This function would ensure that only one ssh key is loaded in the agent, and that it would correspond to a specific directory.

## New Solution

### Problem

The reason the ssh agent would always use the first key is because since the keys are both for github, the ssh Hostname for both is github.com. This meant that as far as the agent was concerned, the first key matched the Hostname so was the right key to use.

### Solution

Use `core.sshCommand` in `.gitconfig` files to specify the exact ssh command to be used with git instead of using `~/.ssh/config` or an ssh agent.
