![dotfile-management-for-humans](https://user-images.githubusercontent.com/20260845/59078947-724fd680-88af-11e9-8457-465e2510460b.png)

---

<p align="center">
🎉 A simplified explanation for a popular dotfile management workflow! 🎉
</p>

---

## Starting From Scratch

### 🚦 A Quick Note About Workflows

> To start from scratch, the remote repository you intend to use should be
empty. If your remote repository is non-empty (even if the only contents are a
single README file), then I recommend skipping to the
[Migrating to a New System](#migrating-to-a-new-system) section. The workflows
are similar (and I promise you won't miss out on any important details), but in
the migration section I cover how to deal with file conflicts; a topic not
covered in this section since the remote repository is assumed to be empty.

The first step to managing your dotfiles is to create a bare repository and
establish an alias for working with it.

```bash
git init --bare $HOME/.dotfiles
alias dotfiles='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'
```

Next, hide untracked files and prevent adding the repository to itself.

```bash
dotfiles config --local status.showUntrackedFiles no
echo ".dotfiles" >> $HOME/.gitignore
```

Lastly, add a remote so that your local branch is tracked remotely.

```bash
dotfiles remote add origin <git-repo-url>
```

### 🍾 You're Done

Now your dotfiles repository is ready for use. I suggest adding the alias to
your `.bashrc` file so that it is present for future session.

## Using the Dotfiles Alias

We can demo the dotfiles alias by adding the `.gitignore` file we either created
or modified during the setup:

```bash
dotfiles add .gitignore
dotfiles status
dotfiles commit -m "Added gitignore."
dotfiles push
```

Easy peasy 🍋 squeezy!

## Migrating to a New System

To migrate your existing dotfiles repository to a new system, clone a bare
repository for your remote repository and establish an alias for working with
it.

```bash
git clone --bare <git-repo-url> $HOME/.dotfiles
alias dotfiles='/usr/bin/git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME'
```

Next, hide untracked files and prevent adding the repository to itself.

```bash
dotfiles config --local status.showUntrackedFiles no
echo ".dotfiles" >> $HOME/.gitignore
```

Then, use the alias to checkout your dotfiles in your `$HOME` directory.

```bash
dotfiles checkout
```

### ⚠️ File Conflicts

> If current files on your system conflict with those being checked out, then
you will get a warning about untracked working tree files being overwritten. To
resolve this, rename each of those files to some non-conflicting name or move
them to a backup directory. When I encounter this warning message I move all
conflicting files to a `.dotfiles.bak` directory so that the files no longer
pose a conflict and are contained within a single directory that is easy to
manage. Below is the simple command that I use to automate this process.
>```bash
>mkdir -p .dotfiles.bak \
>   && dotfiles checkout 2>&1 | egrep "^\s+" | awk {'print $1'} | xargs -I {} mv
>   && dotfiles checkout
>```

### 🍾 You're Done

Now your dotfiles repository is ready for use. Don't forget to specify your
upstream branch on your first push (e.g., `dotfiles push -u origin master`)! If
you do not have the dotfiles alias in you `.bashrc` already, then I suggest
adding it so that the alias is present for future session.

## Why Bare Repositories

The concept of using a bare repository is nothing new and definitely not a
concept that I originated, but a thorough explanation of why a bare repository
is used over a normal repository seemed to be missing. I use this section to
disect the differences of the repositories types to show by example why bare
repositories have been chosen for the task.

According to Git (`man gitrepository-layout`):

> A Git repository comes in two different flavours:
> * a `.git` directory at the root of the working tree;
> * a `<project>.git` directory that is a bare repository (i.e., without its own
working tree), that is typically used for exchanging histores with others by
pushing into it and fetching from it.

We can turn to the Git glossary (`man gitglossary`) for further explanation:

> bare repository\
\
A bare repository is normally an appropriately named directory with a .git
suffix that does not have a locally checked-out copy of any of the files
under revision control. That is, all of the Git administrative and control
files that would normally be present in the hidden `.git` sub-directory are
directly present in the repository `.git` directory instead, and no other
files are present and checked out.

For example, if we were to initiate a normal repository `<repo>` in our `$HOME`
directory with the command `git init $HOME/<repo>`. The result file structure
will look like:

```bash
$HOME/
├── ...
└── <repo>/
    └── .git/
        ├── branches/
        ...
        └── refs/
            ├── heads/
            └── tags/
```

Now let's say we opt to initialize `<repo>` as a bare repository instead using
the command `git init --bare $HOME/<repo>`. The resulting file structure should
take theform:

```bash
$HOME/
├── ...
└── <repo>/
    ├── branches/
    ...
    └── refs/
        ├── heads/
        └── tags/
```

We can use the normal repository to mimic the behavior of our bare repository
with
`alias dotfiles=/usr/bin/git --git-dir=$HOME/.dotfiles/.git --work-tree=$HOME`.
Although the behavior seems identical, there is a fundamental difference between
how Git interpets these two cases:
* For the normal repository, the default working tree is `$HOME\<repo>` which we
override with the `--work-tree=$HOME` option. When we override the working tree
to `$HOME`, the `$HOME/<repo>` directory becomes useless except for the fact
that it is the parent directory of the `<git dir>`.
* For the bare repository, we have to specify `--work-tree=$HOME` since there is
no default working tree for bare repositories. As a result, `$HOME/<repo>` is
the `<git dir>` and there is never a scenario where `$HOME/<repo>` becomes
useless.

Is this difference signficant? No, the difference is semantics and an extra
folder.
