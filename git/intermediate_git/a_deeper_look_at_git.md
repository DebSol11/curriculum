### Introduction

Git is a crucial skill to have, whether you're just a hobbyist or you aim to become a professional web developer.  It's the "save" button on steroids and allows for seamless collaboration.  There really aren't all that many commands for you to learn, but sometimes the real difficulty of Git comes from visualizing what's happening.

In this lesson, we'll help with the visualization by diving deeper than just the `git add` and `git commit` and `git push` commands you've mostly been using. We'll cover topics such as Remotes and Pointers. This will expand your understanding of what's actually going on under the hood with Git.

It is **very important** to take a look at all of this before progressing any further with the curriculum. The project work is becoming more and more complex, so using a disciplined Git workflow is no longer optional. Hopefully, after going through this lesson, you'll have a better understanding of Git as a whole.

### Lesson overview

This section contains a general overview of topics that you will learn in this lesson.

- Pointers.

#### Getting set up

Before we get started with the lesson, let's create a Git playground in which we can safely follow along with the code. 

```bash
touch test{1..4}.md
git add test1.md && git commit -m 'Create first file'
git add test2.md && git commit -m 'Create send file'
git add test3.md && git commit -m 'Create third file and create fourth file'
```

#### Setting up the code editor

To perform certain Git commands it's important to configure your code editor correctly. By default, Git opens the text editor in the command-line interface (CLI), which may prevent you from saving and closing the editor after making changes.

To set up your code editor properly, you can follow the instructions provided in the Git Basics lesson. Here's the specific section that covers the process: [Changing the Git Commit Message Editor](https://www.theodinproject.com/lessons/foundations-git-basics#changing-the-git-commit-message-editor).

#### Changing the last commit

So if we look at the last commit we made *Uh-Oh!*, if you type in `git status` and `git log` you can see we forgot to add a file! Let's add our missing file and run `git commit --amend`

```bash
git add test4.md
git commit --amend
```

What happened here is we first updated the staging area to include the missing file, and then we replaced the last commit with our new one to include the missing file. If we wanted to, we could have changed the message of the commit and it would have overwritten the message of the past commit.

Remember to **only amend commits that have not been pushed anywhere!** The reason for this is that `git commit --amend` does not edit the last commit, it *replaces that commit with an entirely new one*. This means that you could potentially destroy a commit other developers are basing their work on. When rewriting history always make sure that you're doing so in a safe manner, and that your coworkers are aware of what you're doing.

#### Squashing commits

Using `squash` for our commits is a very handy way of keeping our Git history tidier by combining multiple commits into one. It's important to know how to `squash`, because this process may be the standard on some development teams. Squashing makes it easier for others to understand the history of your project. What often happens when a feature is merged, is we end up with some visually complex logs of all the changes a feature branch had on a main branch. These commits are important while the feature is in development, but aren't really necessary when looking through the entire history of your main branch.

Let's say we want to `squash` the second commit into the first commit on the list, which is `Create first file`. First let's rebase all the way back to our root commit by typing `git rebase -i --root`. Now what we'll do is `pick` that first commit, as the one which the second commit is being `squash`ed into:

```bash
pick e30ff48 Create first file
squash 92aa6f3 Create second file
pick 05e5413 Create third file and create fourth file
```

Rename the commit to `Create first and second file`, then finish the rebase. That's it! Run `git log` and see how the first two commits got squashed together.

#### Splitting up a commit

Before diving into Remotes, we're going to have a look at a handy Git command called `git reset`. Let's have a look at the commit `Create third file and create fourth file`. At the moment we're using blank files for convenience, but let's say these files contained functionality and the commit was describing too much at once. In that case what we could do is split it up into two smaller commits by, once again, using the interactive `rebase` tool.

We can open up the tool just like last time, change `pick` to `edit` for the commit we're going to split. But instead, what we're going to do is run `git reset HEAD~`, which resets the commit to the one right before HEAD. This allows us to add the files individually and commit them individually. All together it would look something like this:

```bash
git reset HEAD~
git add test3.md && git commit -m 'Create third file'
git add test4.md && git commit -m 'Create fourth file'
```

Let's start by looking a bit closer at what happened here. When you ran `git reset`, you reset the current branch by pointing HEAD at the commit right before it. At the same time, `git reset` also updated the index (the staging area) with the contents of wherever HEAD is now pointed. So our staging area was also reset to what it was at the prior commit - which is great - because this allowed us to add and commit both files separately.

Now let's say we want to move where HEAD points to but *don't* want to touch the staging area. If we want to leave the index alone, you can use `git reset --soft`. This would only perform the first part of `git reset` where the HEAD is moved to point somewhere else.

You can think of `git reset --soft` as a more powerful amend. Instead of changing the last commit, you can go back multiple commits and combine all the changes included in them into one commit.

The last part of reset we want to touch upon is `git reset --hard`. What this does is it performs all the steps of `git reset`, moving the HEAD and updating the index, but it *also* updates the working directory. This is important to note because it can be dangerous as it can potentially destroy data. A hard reset overwrites the files in the working directory to make it look exactly like the staging area of wherever HEAD ends up pointing to. Similarly to `git commit --amend`, a hard reset is a destructive command which overwrites history. This doesn't mean you should completely avoid it if working with shared repositories on a team with other developers. You should, however, **make sure you know exactly why you're using it, and that your coworkers are also aware of how and why you're using it.**

### Branches are pointers

While the focus of this lesson was more advanced tools for changing Git history, we're going into another advanced topic that might be hard for some to understand - Pointers. You've already learned about branches in the [Rock Paper Scissors revisited lesson](https://www.theodinproject.com/lessons/foundations-revisiting-rock-paper-scissors) and how these hold multiple *alternate reality* versions of our files. Now we're going to discuss what that actually means under the hood, and what it means for branches to be pointers.

Before we dive into branches, let's talk about commits. If you recall this [Git basics lesson from foundations](https://www.theodinproject.com/lessons/foundations-git-basics), they were described as Snapshots. If it helps, think of this in a very literal sense. Every time you type in `git commit`, your computer is taking a picture of all the file contents that have been staged with `git add`. In other words, your entire tracked workspace gets copied.

So what is a branch? Based off of your exposure, you might be visualizing a branch as a group of commits. This actually isn't the case! **A branch is actually a pointer to a single commit!** Hearing this, your first thought might be *"Well if a branch is just a finger pointing at a single commit, how does that single commit know about all the commits that came before it?"* The answer to this question is very simple: Each commit is also a pointer that points to the commit that came before it! Wow. This might be a lot to take in, so let's take a moment to absorb that fact.

Now that you've had a second to gather your thoughts and attempt to wrap your head around this concept, it might help to go back and look at a concrete example of pointers we used in this lesson. Let's think back to our use of `git rebase -i HEAD~2`. If you can remember, this command lets us edit the last two commits. Do you have any guesses on how Git knew which two commits to edit? That's right, by using pointers! We start at HEAD, which is a special pointer for keeping track of the branch you're currently on. HEAD points to our most recent commit in the current branch. That commit points to the commit made directly before it, which we can call commit two. That's how `git rebase -i HEAD~2` starts with a HEAD pointer, and then follows subsequent pointers to find which two commits to edit.

You might be feeling overwhelmed at this point, so let's recap what we've learned. A branch is a pointer to a single commit. A commit is a snapshot, and it's a pointer to the commit directly behind it in history. That's it!

### Assignment

<div class="lesson-content__panel" markdown="1">

1. Hopefully you'll have made use of branches in your workflow since Revisiting Rock Paper Scissors, but whether you have or have not, refresh on [Basic Branching and Merging](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging).
2. Read the chapter on [Reset covered by git-scm](https://git-scm.com/book/en/v2/Git-Tools-Reset-Demystified) for a deeper dive into `git reset`.

<div class="lesson-note lesson-note--tip" markdown="1">

#### Reminder: Default Git Branch Name Change

In modern Git setups, the default branch is typically called `main` instead of `master`.

</div>

</div>

### Knowledge check

The following questions are an opportunity to reflect on key topics in this lesson. If you can't answer a question, click on it to review the material, but keep in mind you are not expected to memorize or master this knowledge.

- [How can you amend your last commit?](#changing-the-last-commit)
- [What does it mean for branches to be pointers?](#branches-are-pointers)

### Additional resources

This section contains helpful links to related content. It isn't required, so consider it supplemental.

- Read this [Git Cheat Sheet](https://www.atlassian.com/git/tutorials/atlassian-git-cheatsheet) if you need a reference sheet.
- Read the chapter on [Branches covered by git-scm](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell) if you want an even deeper dive into Branches.
