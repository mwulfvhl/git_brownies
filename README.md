Git: cleaning up a bad commit history

Goal: demonstrate various Git techniques for righting that which is
wrong, without worrying about actual code

We'll be looking at a repository for my grandmother's brownie recipe.
You can clone it if you want to try this at home:
```
git clone https://github.com/mwulfvhl/git_brownies.git
```

Check out a new branch:
```
git co -b mw_rebase_practice
```
Now do an interactive rebase. The first commit is the README -- we
don't need to change that, so we are going to rebase onto that commit:
```
git rebase -i [the first commit]
```
Git will open an editor with your commits listed in ascending
chronological order:

```
pick A brownies: Add the title.
pick B brownies: Add the ingreedients heading.
pick C brownies: Add the directions heading.
pick D brownies: Add the wet ingredients.
pick E brownies: Add the dry ingredients.
pick F brownies: Add the eggs (forgot them when I did the wet
ingredients).
pick G brownies: Add the special ingredient (family secret).
pick H blondies: Add file for recipe.
pick I brownies: Add directions; add walnuts (forgot to include
them in dry ingredients).
pick J brownies: Fix the number of eggs, remove the mustard, and
change the temperature.

# Rebase root..J onto root
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

This is a script. Git is going to start at the commit you specified in
the rebase command and play these commits onto it in order. The command
at the start of each line describes how the commit will be applied.

## `pick`

**`pick`** (the default command) simply applies the commit as-is. If we were
to run this rebase without making any other changes, our branch wouldn't
change. (The commit SHAs would change, though.)

A looks OK for now: we'll leave it as `pick`.

## `reword`

**`reword`** is just like `pick`, except that it will let us edit the commit
message. There's an embarrassing typo in B, so we'll change this
one to `reword`.

## `squash`

**`squash`** applies the commit but makes it part of the commit right above
it. It also will open the editor, show us the combination of all the
commit messages that are being squashed together, and let us edit it.

C logically belongs with B, so we'll change this one to `squash`.

## `fixup`

*fixup* is just like *squash*, except that it drops the commit message:
when we run the rebase, the fixup will just happen without any
intervention from us.

F logically belongs with D, so we'll make it a *fixup*.

But wait: the commit immediately before it is _not_ the one we want to
silently squash it into. What to do?

## Reordering

Git will play these commits in the specified order -- which means that
we can make the commit happen in whatever order we want. We can switch
F with E so that it fixes up the right commit.

## Deleting

What if we made a commit that we don't want to include at all? We're
editing a script here, so if we don't want a commit, we can just delete
it, and Git will forget all about it. (Actually it won't: we'll get into
that a little later.)

G introduces mustard into these brownies, and we really really do
not want to add mustard. We'll delete the commit.

H also introduces something we don't need right now, but we may want
it later. We'll get rid of this one too, but don't worry: we can get it
back later if we need it.

## `edit`

If related commits belong together, we can reorder and squash them. What
if a lot of unrelated changes are in one commit? No problem. We can use
**`edit`** to split that commit into smaller commits and reorder them (in
another rebase). It's a powerful command but more complicated than the
others.

I and J both contain many small, unrelated changes. We'll make both of
these `edit`.

## Rebasing

To start the script, we save it and leave the editor.

Git applies A as-is.

Git needs our input for B: change the message, save it and exit.

Git also needs us to edit the squashed commit message for D and F: change
it, save and exit.

Git applies E as-is.

Now we get to the fun part. When Git gets to I it sets us down in a shell
and says to let us know when we're done:

```
You can amend the commit now, with

git commit --amend

Once you are satisfied with your changes, run

git rebase --continue
```

In our current state, the commit we're editing has already happened. We
want to split it into smaller pieces, so we need to **`reset`** our state
one commit back:
```
git reset HEAD^
```

HEAD now points to the previous commit and _unstages_ our changes. Note
that the changes are still in our working directory, because this is a
_soft_ reset. If, instead, we had done
```
git reset --hard HEAD^
```
HEAD would still point to the previous commit, but our working directory
_would match that commit_: our changes would be gone. (Well, not gone
exactly -- we'll get to that shortly.)

Now we can stage our changes _interactively_, giving us fine-grained
control of the commits we want to make:
```
git add -p
```

Git shows us the _hunk_ of code that is different and asks:
```
Stage this hunk [y,n,q,a,d,/,s,e,?]?
```

We want only a part of this change -- the chopped walnuts -- so we
choose to _split_ the hunk by choosing **`s`**. Git splits the hunk in two
pieces, shows us the first piece and asks if we want to stage it. We
want this one, so we choose **`y`** -- and for the second piece, we choose
*n*.

Now we have staged the change we want, so we commit it:
```
git commit -m 'brownies: Add walnuts to dry ingredients.'
```

We have only one more change, so we can add it directly and create a
second commit for it.
