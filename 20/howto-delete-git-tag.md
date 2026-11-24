---
title: "Quick tip: How to delete a tag from a Git repository?"
url: "https://bosnadev.com/2016/08/20/howto-delete-git-tag"
author: "Mirza Pašić"
date: "2016-08-20"
topic: "Git"
tags: ["tagging", "versioning"]
summary: "Deleting a wrongly placed Git tag, first from the local repository with git tag -d and then from the remote, such as GitHub, with git push."
---

# Quick tip: How to delete a tag from a Git repository?

> Outdated: Written in 2016. Versions, APIs and advice may have changed since.

The ability to tag specific changes in commit history has become very important. Programmers use git tag to mark the releases (_aka versions_) of their applications. That way they can specify a desired version in their dependency tools.

Sometimes we make a mistake; we are all humans after all. We tag a wrong change set, or we tag a version on a wrong branch, or we even tag a version of other component in a current working directory. It happened to me, and I’m confident that some of those things happened to you to.

I’m sure most of you are very familiar with Git and tagging, but if you are not you should check the Git [documentation page](https://git-scm.com/book/en/v2/Git-Basics-Tagging). If you are just looking for the way to delete the wrongly marked tag, then here’s the quick tip for you; so you don’t have to search through extensive documentation page.

##### Removing a Tag on the local repository

Deleting a tag on the local Git repository is pretty straightforward:

```bash terminal
$ git tag -d 1.15
```

You can easily remember it because it’s so declarative. We can read this command as:  _use_ git tag _command to delete_ -d _tag with a name_ 1.15 \_\_.

That will delete **only** the tag on your **local repository**. If you have already pushed your tag to the remote repository (to the Github for example), then you should also remove a tag on the remote repository.

##### Removing a tag on the remote repository

To delete a tag on the remote repository, we just need to push information about deleted tag to the remote:

```bash terminal
$ git push origin :refs/tags/1.15
```

or just:

```bash terminal
$ git push origin :1.15
```

And that’s it.
