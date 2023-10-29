+++
title = "GNU Stow"
date = 2023-10-29
draft = false
+++

I use [GNU Stow](https://www.gnu.org/software/stow/) in order to have important `.dot` files, usually located in separate directories in the filesystem, version controlled in `git` from a designated repository.
Easier to use a tool like this than to do it manually.

<!--more-->


## Steps required {#steps-required}

-   Create a designated `'stow'` directory, e.g. `~/.dotfiles`.
-   Recreate the directory structure for the files you want to manage as subdir(s) of that directory.

    e.g. to have my [Doom emacs](https://github.com/doomemacs/doomemacs) config files managed by `stow` and they have been installed to `~/.config/doom/`, i would do the following:

<!--listend-->

```bash
# N.B. directory structure
mkdir -p ~/.dotfiles/doom/.config/doom
mv ~/.config/doom ~/.dotfiles/doom/config/
cd ~/.dotfiles
# finally, invoke stow
stow doom
```

Now, listing the initial config directory should show a symlink pointing to new location:

```bash
ls -al ~/.config
doom -> ../.dotfiles/doom/.config/doom
```

Initialize a git repository in `~/.dotfiles`, commit and push to remote and your done.
