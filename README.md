# linux-for-millenials
My notes on finding you way around the Linux Kernel workflow
<> means you have to enter something you want there.

This might contain errors or things that aren't best practice,
but this is were I write down the workflow you can't just easily google online and seems to be just "assumend knowledge"
for kernel developers.
If anything, correct me if I'm wrong or you have a better way, so I learn as well! :D

### Taking commits from another Linux tree and putting them in your own kernel tree
No, you don't have to type them over :)
Say you found a commit in with branch or tag `fancy-new-feature` in `git://git.kernel.org/pub/scm/linux/kernel/git/broonie/regmap`.
You can tell your local Linux git repo about the existance of that repo like this:
```bash
git remote add <local name of repo with fancy feature> git://git.kernel.org/pub/scm/linux/kernel/git/broonie/regmap
git fetch <local name of repo with fancy feature>
```
This is very handy if you add the upstream kernel repo when backporting!
All commits/branches whatever that matter will be there already :)

Now you can see the git log of the branch/tag you are interested in with
```bash
git log <local name of repo with fancy feature>/<fancy-new-feature>
```
Look through this to find your commit sha XXXXX you are interested in, or multiple subsequent commits starting at yyyyy ending at xxxxxx (both inclusive).
Then you can apply your single commit to you tree with
```
git cherry-pick -x xxxxxx
```
The -x will reference the upstream commit in your commit message, good practice when backporting.
For the range, you can apply all of them with
```
git cherry-pick -x xxxxxx~..yyyyy
```

You might get merge conflicts, which you will have to solve yourself.
Check which files have problems with `git status`, change them (look for the <<<<, ====, >>>>), and add them `git add <path to conflict file>`.
When you have changed and added all conflicted files, you can finish the cherry pick with
```
git cherry-pick --continue
```
or if the conflicts scare you and you want to abort the cherry picking
```
git cherry-pick --abort
```

### Finding patch series
Linux never imports a single commit at a time, instead, people send the maintainers their patches.
Those maintainers add those patches to some seperate branch they have in their own repo.
When the time comes for a new linux release, you will see merge commits by Linus Torvalds for Linux subsystems (e.g. overlayfs / ovl).

If you are lucky and working with a recent kernel, these commits will have a line like this:
```
* tag 'regmap-v6.3' of git://git.kernel.org/pub/scm/linux/kernel/git/broonie/regmap:
```
You are in luck!
In this case, just clone the `git://git.kernel.org/pub/scm/linux/kernel/git/broonie/regmap` repo and
`git checkout regmap-v6.3`. In the git log you should see a commit by Linus for the previous linux version (6.2) here.
Everything above that are the commits included in this patch series!

On older commits, this tag workflow wasn't always used it seems.
They would use branches instead which might not exist or have been overwritten anymore.
An example I ran into is this:
```
commit 48023102b7078a6674516b1fe0d639669336049d
Merge: ba2b137d10ba 16149013f839
Author: Linus Torvalds <torvalds@linux-foundation.org>
Date:   Fri Apr 13 16:55:41 2018 -0700

    Merge branch 'overlayfs-linus' of git://git.kernel.org/pub/scm/linux/kernel/git/mszeredi/vfs
```
When finding one of those merge commits, the top line will say merge xxxxxx yyyyyy,
where xxxxxx is a commit sha that will point the the top of the previous linux version git log,
and yyyyyy is a commit sha that will point to the top of the new commits in this patch series that are new since xxxxxx.
So to view them for example you can do `git log yyyyyyy`!

Since you have all these new commits one after another, you can cherry pick all of them with `git cherry-pick xxxxxx..yyyyyy` now!

### finding documentation
The bootlin website has a great click and link system for finding definitions of kernel functions/macro's/...
But that is not where the info is a lot of the time.

Instead, never underestimate the power of:
```
cd Documentation
git grep -i help_me_with_this
```
The kernel Documentation has docs for kernel developers, not (just) users!
A great trick when trying to figure something out is read the docs of the parent subsystem or the tools it uses (e.g. vfs for overlayfs).

### I want to write something for which I don't have hardware

For some things Linux provides fake/dummy devices, such as this fake i2c bus I just found.
https://docs.kernel.org/i2c/i2c-stub.html


### Where can I find the Linux git repo's ###
Don't use the one you find on Github when you google it.
Use the on over at git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git

That one is the canonical Linux repo. Everything that appears there is the official state of Linux.
But it is not the "this is coming up in Linux" repo, this one only has the patches that have been approved and grouped for every release or request for comments (rc).
The "bleeding edge" / staging one is over at git://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git
You should create your patches against the master branch over there.
This repo contains tags for every day, so `next-YYYYMMDD`.

=== I want pretty Documentation ===
The (recent) kernel allows creating html from the documentation by using `make htmldocs`.
You will then find the webpage under `Documentation/output/html`.
You can find how to generate and write documentation here: https://www.kernel.org/doc/html/v4.10/doc-guide/sphinx.html
