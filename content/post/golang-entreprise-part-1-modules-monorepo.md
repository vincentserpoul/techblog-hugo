+++
date = "2022-04-30T17:01:21+08:00"
title = "golang enterprise, part 1, using a modules monorepo"
description = "go modules monorepo"
tags = [ "golang", "enterprise", "git", "monorepo" ]
topics = [ "golang", "development" ]
slug = "golang-entreprise-part-1-modules-monorepo"
+++

## Leveraging go workspaces

Since go 1.18, we have a nice features, which is [go workspaces](https://go.dev/doc/tutorial/workspaces).

One thing that this brings is allowing to work nicely with multiple modules within one single workspace.
Guess what it brings you? having multiple modules in one single monorepo!

Take a look at [this example](https://github.com/vincentserpoul/testwrksp)

## Benefits

* Have all the important reusable company modules at the same place. Bringin a better collaboration, and easier discovery.
* Mutualize the CI (which is pretty easy)
* Have good folder structure, allowing the grouping of modules in topics

## Drawback

The only drawback I could see until now it the coverage during the CI.
It took quite some time to configure [coveralls.io](https://coveralls.io) to display one number which was the aggregate of all modules.
Which also has the consequence of having to run all coverages, each time there is one change in one of the repo (it can probably be mitigated with some cache management).

## Versioning

How do you then version your modules independantly?
Simple: use semantic versioning including the path. For example, for the module z located in x/y/z, you do:

```bash
git tag x/y/z/v1.0.0
```

and that's it, you have differentiated versioning!
You can even pull another version of a module in one of the other modules in the same repo.

## Bonus: Docs

For the documentation, you often don't have access to a pkg.dev portal.
Easy way out: generate your doc in markdown thanks to [gomarkdoc](https://github.com/princjef/gomarkdoc).