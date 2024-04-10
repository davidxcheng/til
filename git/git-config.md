# Git config

## How to edit the config files

Almost all examples of adding config values look like this:

```
git config --global user.email hey@example.com
```

This feels a bit like flying blind to me and I prefer to make edits using a text editor, and git provides an `--edit` option that does exactly that:

```
git config --global --edit
```

### How to find the source of each git config setting

There are (at least) 3 levels of git config: `local`, `global` and `system` and git looks for config values in that order. This means that configuration of git can vary depending on the working directory and you can get gits view of the config with the `--list` option. Adding `--show-origin` displays the name of the file where the config line is stored and `--show-scope` shows which level it's on (`local`, `global` or `system`).

```
git config --list --show-origin --show-scope
```

## How to set user.email for all repos inside a directory


There is also a feature called [Conditional includes](https://git-scm.com/docs/git-config?ref=blog.gitbutler.com#_includes) which can be used to point to any random config file. Example:

```
[includeIf "gitdir:~/work/client-uno/"]
  path = ~/.gitconfig-client-uno
```

The data that follows `gitdir` is a glob pattern and the slash at the end is expanded to `/**` so all subdirectories will also be a match. This can be useful if you're using the same laptop for both private coding and for work (or if you have several clients) and want to use the same `email` for all repos in a directory without setting `user.email` locally in each repository.

### How to use the `system` config for fallback/default values

Set your default `user.email` in the `--system` config:

```
# /private/etc/gitconfig (system)
[user]
  email = me@example.com
```

Remove `user.email` in the `--global` config and add `includeIf` for directories where you want to override the default `user.email`:

```
# ~/.gitconfig (global)
[includeIf "gitdir:~/work/client-uno/"] # Notice the last slash
  path = ~/.gitconfig-client-uno

# Don't have user.email in this file
```

Create the custom config file(s) that you refer to in your `includeIf` clauses in the `--global` config:

```
# ~/.gitconfig-client-uno (custom config for repos in the client-uno directory)
[user]
  email = me@client-uno.com
```

#### Gotchas

- The `gitdir` glob patterns must have a `/` at the end or else it will not match subdirectories
- The `global` config must not contain `user.email` or else git will use that value instead of the `includeIf` config
