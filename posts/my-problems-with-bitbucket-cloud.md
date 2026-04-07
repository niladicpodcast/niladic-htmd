---
author: Cody
title: My Problems with Bitbucket Cloud Pull Requests
_hash: b518ab2d21e6c2113d1f80118e643cee85201b5aa3ce76468deb1af065d1cc93
published: 2017-04-19T11:52:42.486942+00:00
updated: 2026-04-07T09:03:47.412804+00:00
...

I use Mercurial with bookmarks instead of named branches and a repo (fork) per user. I only have the default named branch. Development happens in a fork and a pull request is created.

This is an example of my <a href="https://bitbucket.org/Siecje/hg_setup/src/default/workflow.md?fileviewer=file-view-default" target="_blank">workflow</a>.

These are the problems I have encountered while using Bitbucket Cloud.

## 1. You can't get the code of a pull request from a private fork.

The diff of a pull request is not the greatest, especially for large diffs.
Sometimes you want to have the code of a pull request locally,
to run tests or look at the code in your preferred editor with syntax highlighting, the ability to jump to definition, and search for other uses.

If you have read access to the source repo then you can clone the source repo. If it is a public repo then everyone has read access by definition.
However if you are working with a private repo with private forks then by default you won't have access.
You can view the changes in the pull request but you can't get them locally.

Each private fork needs to add read access for the rest of the team.
Fortunately, you can add a group from the original repo, but it would be nice if you could enforce that all forks have certain user and group access.

GitHub creates a branch in the pull request destination repo that you can pull down from the destination remote repo, so you don't need access to the source repo.

## 2. You can't have a pull request source be a specific commit

A pull request source must be a head. So you can't create a pull request and prevent it from automatically updating
and then add additional commits to the commit in the pull request, without having them show up in the pull request.

The use case arises when you need a new feature to be reviewed but you need to use the first feature for another feature.

You can create another fork and push the additional commits to the new fork.
Or you can do `hg graft` and then rebase the new commits after the pull request is merged
and `hg prune` the duplicate commits from the graft.

Ideally, a pull request source could be set to a bookmark, and if you move the bookmark then the pull request would update, if you don't move the bookmark but add commits on top of that commit, the pull request should not update.
Since Mercurial supports merging non head commits, it would be nice if you could do it in a pull request.

Just to show that Mercurial supports this

```shell
$ hg init
$ touch a
$ hg add a
$ hg commit -m "initial"
$ hg la
@   0 cody tip (2017-03-10)
   initial
$ echo a > a
$ hg commit -m "mainline has moved"
$ hg update -r .^
1 files updated, 0 files merged, 0 files removed, 0 files unresolved
$ echo b > b
$ hg add b
$ hg commit -m "requesting if b can be added"
created new head
$ echo "changing contents of b" > b
$ hg status
M b
$ hg commit -m "changing b"
$ hg la
@   3 cody tip (2017-03-10)
|  changing b
o   2 cody (2017-03-10)
|  requesting if b can be added
| o   1 cody (2017-03-10)
|/   mainline has moved
o   0 cody (2017-03-10)
   initial
$ hg update 1
1 files updated, 0 files merged, 1 files removed, 0 files unresolved
$ hg merge -r 2
1 files updated, 0 files merged, 0 files removed, 0 files unresolved
(branch merge, don't forget to commit)
$ hg la
o   3 cody tip (2017-03-10)
|  changing b
@   2 cody (2017-03-10)
|  requesting if b can be added
| @   1 cody (2017-03-10)
|/   mainline has moved
o   0 cody (2017-03-10)
   initial
$ hg status
M b
$ hg ll
@   1 cody (2017-03-10)
|  mainline has moved
o   0 cody (2017-03-10)
   initial
$ hg commit -m "merge"
$ hg la
@     4 cody tip (2017-03-10)
|\   merge
| | o   3 cody (2017-03-10)
| |/   changing b
| o   2 cody (2017-03-10)
| |  requesting if b can be added
o |   1 cody (2017-03-10)
|/   mainline has moved
o   0 cody (2017-03-10)
   initial
$
```

## 3. Pull requests don't show the destination bookmark

When you create a pull request you set the source and destination, but when you are viewing an existing pull request you can't see the destination bookmark, only the branch name.
Since I only have the default branch and use bookmarks for multiple heads, I can't see the destination bookmark. So I have to click merge without knowing where the commits are going to end up.
It also doesn't show the bookmark of the source in the case you want to clone the source repo and manually merge, you have to click on the commit and see the bookmark or use the commit hash.

## 4. Merging pull requests doesn't always move the destination bookmark.

My pull requests always have a destination bookmark, but unless the source commit has the same bookmark and there is no merge commit added by merging the pull request, the bookmark doesn't move.
I think it should always move the bookmark, because the bookmark is essentially the "branch" and should be on the head of the "branch". Though there are cases where you want to have a bookmark for a commit that is not a head.

The default behaviour should be to move the destination bookmark and maybe add a checkbox to the pull request to prevent the destination bookmark from moving.

The cases where the destination bookmark does not move are as follows:

1. When there is no merge commit but source and destination bookmarks are different
2. When there is a merge commit but with different bookmarks
3. When there is a merge commit with the same bookmarks

The case when the destination bookmark does move:

1. No merge commit and source bookmark is the same as the destination bookmark

<a href="https://bitbucket.org/site/master/issues/12287/pull-requests-merged-ontop-of-a-mercurial" target="_blank">Bitbucket Issue</a>

## 5. When you push a successor commit the pull request doesn't always update

With Mercurial evolution support in Bitbucket Labs, you can now push a successor commit to replace a commit, which makes the original commit obsolete.
However pull requests do not automatically update, they still have the old and now obsolete commits as the source.

You can edit the pull request and select the new commits as the source.

Ideally you should be able to set the source to be a bookmark and when that bookmark changes (ex. to a successor commit) then the pull request should update.

I have had this happen once, though it is not clear why it doesn't always happen.

## 6. Updating a pull request is buggy

When you update a pull request the destination is set to the first commit sorted by commit hash alphabetically, which is weird because when you create a pull request the destination is the newest commit in that branch.

This means you have to be careful and set it back to what it was before you clicked *Edit*.

Also sometimes editing the pull request causes the destination to be the wrong commit and doesn't work even though you changed the destination.
Or the source is set to default and shows the most recent commits added to the repo and you can't change it.
When this happens I just make a new pull request and close the bugged one. This causes two emails to be sent to the reviewers.

You can't just edit the pull request description, there is only the one edit form and as previously mentioned it changes the destination and you risk the pull request becoming bugged.

**Update**: 2017-08-16 When editing a pull request the destination is no longer the first alphabetically sorted commit, it is the head that was most recently updated.

## 7. Can't build CI for a pull request from another repo

Bitbucket Pipelines won't build a pull request from a fork. You can enable pipelines in the fork, but you can only trust the passing pipeline if there won't be a merge commit when you merge the pull request, since it hasn't tested the code when it is merged.

Bitbucket pipelines is pretty new, and even Jenkins doesn't have a plugin that can merge and run on a pull request from another repo.
There is a plugin that looks like it will do this and even triggers when you create a pull request, but it just pulls down the destination repo and does a build, which is not obvious when you start using it.
https://github.com/nishio-dens/bitbucket-pullrequest-builder-plugin

<a href="https://bitbucket.org/site/master/issues/13162/pull-requests-from-forks" target="_blank">Bitbucket Issue</a>

## 8. Pull requests say they have been updated when they have not

I have multiple pull requests for heads in the default branch. When I push to the default branch in the source repo it says all of the pull requests have been updated.

## 9. You can merge a pull request without viewing the current version

If you are viewing a pull request and someone adds a commit and updates the pull request, you can merge the pull request without viewing the new commit. When you merge the pull request both commits are merged.

## 10. You can merge a pull request with obsolete commits

If you are viewing a pull request with draft commits and they have been replaced with a successor commit it is still possible to merge the pull request with the obsolete commits.

<a href="https://bitbucket.org/site/master/issues/14613/hg-prevent-merging-a-pr-with-obsolete" target="_blank">Bitbucket Issue</a>

## 11. You can't create a pull request for another fork of the main repo

If you are reviewing a pull request for a repo and you want to suggest changes to the pull request, what can you do?

I thought you could simply create a pull request from your fork to the other person's fork. But you can only create a pull request from your fork for the main repo.

On GitHub you go to the destination repo to create the pull request so the experience is a bit cleaner.

As a workaround you can create another fork of the fork.

## 12. No rebase when a pull request is accepted

If I work on a feature and create a pull request, I can update to its parent and work on the second feature and create another pull request.
But when the second pull request is accepted there will be a merge commit in the repo. To avoid merge commits I would need to rebase after the first pull request is accepted.
This means reviewers could only review one pull request at a time.

As a workaround you can just commit the second feature as child of the first feature and then only have one pull request
and ask that the reviewer look at each commit as if it is its own pull request.

This is not ideal because it is good to have small commits that can be reversed if needed and this encourages one commit per feature.

Once Mercurial has an in-memory merge then rebase can be done without the working directory of the repo.
