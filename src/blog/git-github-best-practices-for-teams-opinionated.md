---
title: Git + GitHub Best Practices for Teams (Opinionated)
image: https://thepracticaldev.s3.amazonaws.com/i/8g5esyy2y11yjo8vuabn.png
description: A collection of thoughts on using Git to its fullest when working on a project team 
layout: blog-post
date: 2019-05-16T19:50:13.820Z
---

_**Disclaimer:** This was adapted from a guide written internally for our Bits of Good organization to help newbies get their footing when joining a project team. You can learn more about the organization [here](https://bitsofgood.org) if you're interested. Yes, we are reworking the website these next few months :)_

There are [P](https://datree.io/github-best-practices/) [L](https://yeti.co/blog/get-the-most-out-of-git-7-best-practices-for-budding-developers/) [E](https://www.git-tower.com/learn/git/ebook/en/command-line/appendix/best-practices) [N](https://raygun.com/blog/git-workflow/) [T](https://medium.freecodecamp.org/how-git-best-practices-saved-me-hours-of-rework-cf227bad9a50) [Y](https://git-scm.com/book/en/v2) of articles out there with a new spin on how to best use Git. Given how versatile and free-form the tool can be, it's easy to find a new system that works for someone. As such, there really isn't a catch-all gold standard that everyone **must** follow. So don't take the contents of this article as the law! This is merely an opinion piece with suggestions our Bits of Good organization has found to work well. 

Note that Bits of Good is a college-level organization, so my opinions are based on experiences with smaller teams rather than working at large companies. So, issues like backlogs, ticketing, and user feedback are not covered here given my limited exposure. 

## Creating and Organizing Issues

Before jumping into modifying the codebase, let's talk about organizing what needs to be done first. A nifty GitHub feature is the option to create "issues" to break down tasks into assignable chunks. This is helpful in a team setting to decide on deadlines and priorities to keep everyone on track.

### What a good issue should say

Let's start with an example of a bad issue:

![Example of bad GitHub issue](https://thepracticaldev.s3.amazonaws.com/i/tpjshixy7vhwkb2c1nji.png)

Not only is it unclear how the final result should look, but it's nowhere near enough direction to keep a newbie's head from exploding. Do everyone a favor and break issues down!

GitHub themselves are nice enough to include example headers whenever you create a new issue. These are super nice for getting started, but may prove to be a little too much for the issues you create. At a minimum, try to include:

1. If it's a feature: A description of the feature being added to the project
2. If it's a bug: A description of what the bug is and how to replicate it
3. A breakdown of **action items**. This should be a list (with checkboxes preferably) of bite-sized tasks to complete in order to finish the feature / resolve the bug
4. General notes for the feature / bug or improvements that may be needed down the road
5. Discussion points if the team still needs to decide on certain details

The action items should be the bulk of the issue. You can get as granular or technical as you want, so long as it makes someone's life easier trying to read the issue. Good ideas are to include hyperlinks to potential solutions and supporting images like mockups as a guide.

**Optional:** Break down the action items like a tutorial, walking the reader through one potential method of coding out the issue. This is great for new team members so they don't feel lost navigating an unfamiliar codebase.

Here's an example of how to beef up that confusing one-liner issue from earlier:

![Example of good GitHub issue](https://thepracticaldev.s3.amazonaws.com/i/whvbzrw1l6cesm5xn76j.png)

### Leveraging milestones

If your team happens to use [Agile development](https://linchpinseo.com/the-agile-method/) or another form of rapid release cycles, you should probably avoid chucking issues around without any reference for completion timeframes. GitHub allows you to get a focused view of what you're working on for each dev cycle / sprint using "milestones." This is little more than a subset of issues in the repo at a given time.

To view these, look for the "milestones" button within the "Issues" tab. From here, you can create a new milestone with a useful title, description, and deadline. You may tack an issue onto a given milestone using the sidebar when creating a new issue. In general, make sure you don't have too many issues in a given milestone. It may take some time to feel out how long certain tasks will take, so stay optimistic in the beginning and pull back depending on the workload.

![Example of a GitHub milestones setup](https://thepracticaldev.s3.amazonaws.com/i/wc3c731g2nsb9o3cbldh.png)

_A look at our team's past milestones working on an Agile development schedule. We found emojis can help focus milestone objectives_ 😛 

## Branch flow using Gitflow

It's easy to jump into a new project and immediately think "screw the formalities, I'm gonna jump into some code!" Well, it's easy to forget an important detail when blinded by excitement: you should always branch off of a development branch first!

This approach is often lost on beginner Git projects, branching off of master alone and merging in whenever code gets the thumbs up. This approach works well on small-scale projects that are only seen by developers most of the time. However, When users get involved that may access the live project at any time, pushing to a "production" branch like master for every change may not always be a good idea. For example, say the you're on a team building a notification dashboard, with two developers splitting up the UI and a third working on API endpoints. Instead of pushing to master or working off of some awkward middle-way branch, wouldn't it be nice to push to a develop branch to merge everyone's changes gracefully?

This process is known as [Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow), using a `develop` branch for building out features and a `master` branch for the user-facing code. Other flows include further branching stages such as a `staging` branch, used for testing and validating code before it is merged into the `master` branch.

Setting up this flow is extremely easy in Git. Just leave the default `master` branch, treating it as the production branch, and checkout a new `develop` branch off of `master` to branch from going forward. Also make sure all pull requests made to the project are targeted at this branch instead of master! The only PR to master should be from the develop branch itself most of the time.

## Branching best practices 

If you've been through a workshop or basic Git tutorial before, you've probably made branch names like `Ben-Holmes`, `bug-fix`, or `homepage-57`. **Please don't do this!**

As a general rule, try to title branches like an overarching todo task for yourself. So, it should be descriptive enough that you know exactly what you're working on, but kept to 5-7 words or less so it's not impossible for teammates to type. In addition, the scope of a branch should be more focused than broad so all your commits are working on a common issue. This has an added bonus of avoiding little "side quests" while you're coding. If you see a little spelling mistake on the splash page that has nothing to do with what you're working on, don't make the fix in your current branch!

### Formatting

You can use whatever works best for the team of course, but a solid practice is to:
1. Start the branch name with either your name or your Git(hub, lab, whatever) username. This makes it easier to come back and find your branches later.
2. Include the issue number your branch resolves. If your building something without an issue attached (don't do this too often!), use a short tag on what your branch is trying to resolve. This could be `feature`, `bug`, `poc`, etc.
3. Add a 4-5 word title on what your branch does. You can use dashes, underscores or camel case to separate the words.
4. Use a consistent character to separate all of these 👆 like a dot or a dash. You can also uses slashes, but this can have some [unintended git-y consequences](https://github.com/desktop/desktop/issues/7397)!

### Let's see an example

Say you’re working on an issue titled "**#154**: Add profile picture editing to the 'My Account' page." As the title implies, you are adding an option to the user's account management page to edit their profile picture, say, when they click on the image. Here's a decent branch name for the issue:

```
git checkout -b pr0H@ck3r.154.my_account_edit_profile_pic
```

Yes, this is a pretty rigid structure for a simple branch name. However, it really improves scan-ability when there are tens or even hundreds of branches open at a given time!

## Committing best practices

It's the end of the day. You've exhausted your third cup of coffee and finally squashed an Internet Explorer bug that took hours to track down. Now you're ready to push up your changes and finally shut your computer. So, what's the best way to communicate all the pain and suffering you worked through with the team?

```
git commit -m "fixes"
```

😓☕️ Well... it's definitely not this. The commit message isn't some throw-away blob of text that no one will ever look at. This is the developer's chance to explain what their code is meant to accomplish, so when a peer combs through the log of changes you made, they will be able to step through the commits and track which changes are worth adding. This is invaluable for the occasional rollback, when changes made beyond a certain commit should get scrapped.

### Writing a good commit message

So, how can you make the most of a commit message? First, it's best to start with some verb to describe what adding the commit to the project will accomplish. This is often a present tense phrase like "add," "fix," or "include." 

Following this, include a brief phrase to describe the commit's purpose, usually in 50 characters or less. If you use VS Code, the built-in commit menu (the third button down in the sidebar) will do this check for you. This limit should be a check for yourself on how focused the content of the commit actually is. For example, if you were committing a bug fix for an out-of-place profile picture in Internet Explorer, a solid message might be:

```
git commit -m "fix profile picture position on homepage IE"
```

### Spacing out your commits

Yes, that was a pretty easy example to contrive. Commits can easily span many more files and many more features. Always try to fit within the character limit as well as you can, but if there's too much to explain, it might be worth breaking up the commits into smaller chunks.

This can sounding daunting at first, but Git offers a handy tool for this when you're staging everything for a commit: `git add -p`. Rather than staging all your changes at once, this allows you to walk through each change to the codebase you've made file by file, giving you options to stage at each step. Here's a sample output staging some ESLint configuration files:

![Example output of git add -p](https://thepracticaldev.s3.amazonaws.com/i/7im4h1ai8ypah5prj31y.png)

Here, we see a script added to the file `package.json`, along with a number of ways to respond for staging. If you're unsure about what all those random letters mean, just type the "?".

This can be a tedious process for larger commits, but is super helpful when just starting out with Git. It's also safest to **commit like a madman** so your changes are easy to summarize, and you won't have to do this walk(through) of shame as often.

_Extended reading: Much of this section comes from experience and various Stack Overflow sources, but [this](https://who-t.blogspot.com/2009/12/on-commit-messages.html) is a great long-form read on quality commit messages._

## Pull request best practices

Everything about Git we've talked about so far has been pretty standard fare: you read through an issue, you make a branch, you commit code to it, you push it up. It's easy at this point to push everything to the development branch and call it a day... but hang on a minute! If you're working on a team, it's probably best to have someone else review the changes you've made before merging them in. 

The first best practice for pull requests are to... just do them 🤷 Whenever an issue is resolved by the branch you've created and your changes are pushed up, whip up a PR.

There admittedly isn't a great way to do this from the terminal, so it's likely easiest from GitHub directly. Note that the purpose of a PR is to allow others to review your changes before merging into another branch (most always the development branch), so be sure **you have already rebased the remote `master` / `develop` branch onto yours** to make reviewing changes a bit easier.

_Note: If you are unfamiliar with rebasing, it is essentially merging but with an adjustment to the commit history. With rebasing, all commits in the remote will be placed before your own commits, making it seem like your branch is building off of the new remote. [This article](https://medium.com/datadriveninvestor/git-rebase-vs-merge-cc5199edd77c) has a nice visual aid for showing how that works._

Once the pull request is made, add people for review so they will get notified. It's usually best to reach for people who did not work on the change with you so they can review your approach with a fresh set of eyes. However, if you [are a fan of pair programming](https://www.agilealliance.org/glossary/pairing/), it's possible to bypass the review process after having a solid back-and-forth with your partner.

![Example of a good pull request](https://thepracticaldev.s3.amazonaws.com/i/srgwp1ttv0ho7fetgxmb.png)

Here's an example of what I'd consider a good pull request. First, note that the team's lead was requested for review (shown by the checkmark in the "reviewers" section). Also note that the comment box offers a detailed breakdown of what changes were made in the branch. This can be structured however you choose, but it's best to at least:

1. Include the issue number so the reader can reference what is being resolved
2. Write a bulleted / numbered breakdown of each feature added, or changes made to resolve a bug

Below this, there is a log of each commit made in the branch requested for merging. This shows how important quality commit messages are! 
