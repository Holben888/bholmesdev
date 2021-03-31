---
title: How to revert your git commits without panicking
description: When your code turns to 💩 but you've already committed, how do you go back?
layout: blog-post
publishedOn: 2019-06-13T02:23:26.903Z
---

## Step 1: Take deep breaths

The best way to overcome your escalating fear that your codebase / world is collapsing around you is deep breathing. Before frantically flying to the keyboard, just go _in through the nose... and out through the mouth._

![Person meditating in front of an orange sunset](https://thepracticaldev.s3.amazonaws.com/i/icmotiqzlpbpgucj0rhi.jpg)

## Step 2: Look over the commit log

Before doing anything you might regret, it's best to look through all recent commits to pinpoint where you're reverting to. The cleanest way to do so is using:

```bash
git log --oneline
```
![git log of commit history](https://thepracticaldev.s3.amazonaws.com/i/sgohk5klbw8z2nxfjuey.png)

This will show all of the commit history for your branch, starting with the most recent. It also shows branch points to see the history of everything that got merged.

Also notice the alphanumeric strings to the left of each commit. These are hashes which uniquely identify a given commit. Once you find the inflection point for your code explosion, copy the hash for that commit and move on to the next step.

_**Note**: You can exit this log by hitting the "q" key. This is very [vim](https://vim.dev) so I don't blame you if you got stuck here_ 😛 

## Step 3: Diff against the most recent commit 

Checking for differences against your revert point is a great way to verify what changes you're about to make. To do so, use the copied hash and run the following command:

```bash
git diff HEAD COPIED_HASH^
```
![git diff sample output](https://thepracticaldev.s3.amazonaws.com/i/12cvckmewfkubzntuhky.png)

Let's break this output down. First, we see a log of all binary files changed, with paths listed as `a/file_path` and `b/file_path`. These files are often editor / build specific files (as in `.vscode` or `.circleci` directories) and image files. In this case, the the resolution of a few images were updated.

Next, we see a line-by-line breakdown of lines added and removed from each modified file. Note this is from the perspective of changing _from_ our most recent commit _to_ the commit we're reverting to. So, everything in green is what will be added _after_ reverting, and everything in red will be removed.

Also note the `^` at the end there. This makes sure we diff down to the ancestor of that commit hash. Otherwise, the diff will stop one commit early.

If you scroll through this and everything looks good, it's time to obliterate those commits!

## Step 4 Option 1: Run a commit revert (recommended)

The first option is the most obvious one: run the `revert` command onto the commit hash we're reverting to. This is very straightforward:

```bash
git revert COPIED_COMMIT_HASH..
```

Note that doing so will create a new commit meant to revert all changes made down to the commit specified. This works well since you should be able to push your changes and have a clean change log as intended. 

Worth warning that this will open a confirmation file in vi (assuming you're using the default git editor) for **each** commit being reverted. If everything looks as expected, type ":wq" to confirm the reverts.

You can also add the `--no-commit` flag to stage each revert without actually creating a revert commit. This is useful when reverting several commits deep since it allows you to submit one large revert commit at the end. Otherwise, you would end up submitting a revert commit for each one individually.

Still, you might want to change your commit history entirely so it appears those nasty commits were never there. For that, there is a second option...

## Option 2: Rebase and drop commits

_**Note**: Read the disclaimer at the end of this section before going with this option._

Rather than making a revert commit, you can modify the commit history directly using a rebase. To start the process, type the following:

```bash
git rebase -i COPIED_COMMIT_HASH^
```
![git interactive rebase sample output](https://thepracticaldev.s3.amazonaws.com/i/p8zgqcct0ncexpwzv2e0.png)

Notice the `-i` flag here. This is a shorthand for `--interactive`, which allows you to edit each commit individually from the head to our specified commit hash.

It's simple from here to drop all of our commits. The instructions are actually visible right there in the comments, but as a TLDR: Replace every "pick" with a "drop" for all commits listed.

...If you're stuck wondering why your delete key isn't working, that's because git uses [vi](https://vim.dev) as the default editor. You can go [here](https://www.kevinkuszyk.com/2016/03/08/git-tips-2-change-editor-for-interactive-git-rebase/) to change over to your favorite editor for this process, but if you're fine sticking with vi, it shouldn't be too difficult.

- To start editing, hit the "i" key and use the arrow keys to move your cursor around
- Move your cursor over to each "pick" statement, delete, and replace it with "drop"
- Once all the picks are replaced, hit "esc"
- To confirm the rebase, type ":wq". This will save the rebase file you just edited

If you want to abort at any time, delete all the lines listed above the commented section (those starting with "#") to force the rebase to error out. If you're using vi, just mash "d" repeatedly until the lines are gone 🙃

If your terminal window closes or something else happens, the rebase will hang uncompleted. To abort the rebase and start over, enter the command `git rebase --abort`.

If all goes well, you will have officially changed history!

![Applicable Doctor Who quote: wibbly wobbly timey wimey stuff](https://media.giphy.com/media/M6VxE9CEHMDtK/giphy.gif)

### Disclaimer

Modifying git history can be a problem if you have already pushed now reverted commits to the remote branch. After reverting, it will appear your branch is behind the remote, so you cannot push directly anymore. In this case, you will have to run `git push --force-with-lease`. This flag will force your history onto the remote by removing those dropped commits. `with-lease` acts as an error check to ensure the rest of your commits are in line with the remote's.

You may have heard using `--force` on a push isn't best practice since it involves rewriting commit history on the remote. This can be a problem for others working off of the same branch, since their history will no longer line up with the remote's. [There are ways to fix this](https://stackoverflow.com/questions/9813816/git-pull-after-forced-update), but make sure others are aware of your force push before they run into trouble!

## Wrapping up

Hopefully by the end of this you've gotten your branch back on track! Mainly, always make super sure of what commands you're about to run when modifying commits, especially when editing git history directly. This can be a very daunting task for beginners and experts alike, so making sure of the diff and viewing the commit log are a must!

I actually wrote this article after having to go through the same process for my job, wanting to give my hastily written notes some clarity. Hope this helps others stuck in the same boat 😊 
