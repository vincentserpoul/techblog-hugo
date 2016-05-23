+++
date = "2014-02-20T08:27:27+08:00"
title = "GIT basic usage"
description = "git usage for beginner"
tags = [ "git", "versioning", "beginner", "commands" ]
topics = [ "git", "development" ]
slug = "git-basic-usage"
+++

### Git basic commands

#### When you have a new repo
```
    git clone git@github.com:vincentserpoul/rbac.git
```

https or git? the git protocal provides better security and easiness if you are using a correct key.

If you use the git protocol, you can easily associate your private key in the ~/.ssh/config file with the following:

```
Host github
    Hostname github.com
    User vincentserpoul
    IdentityFile ~/.ssh/myprivatekey
    IdentitiesOnly yes
```

#### When you have a new modification ready for commit

```
git status
```

You will get an overview of what's new
If there is any new files (Untracked files), you will first need to add them:

```
git add mynewfile.ext
```

Then

```
git status
```

it will give you the list of the files you are about to commit (Changes not staged for commit).
You can then commit all the files or the selected ones.
Group the commits by functionalities and follow [this](http://chris.beams.io/posts/git-commit/) to write your commit message properly (TL;DR: use imperative for the verb - no ing - and keep it short for the subject line, 50 chars)

```
git commit -m "add feature A" -a
```

Once you reached that, you will need to first sync with the remote repo

```
git fetch origin master
git rebase origin/master
```

This will first get all the commits in the remote repo (supposingly called origin), replay all the commits that have been pushed there and replay yours above them.
If there are any issues, you will have to resolve them.
Modify the problematic file and then:

```
git add problematicfile.ext
git rebase --continue
```

You should be all good right now.
Let's look at our branching strategy.

### Git common flow

Nothing fancy here:
![gitflow](http://nvie.com/img/git-model@2x.png)