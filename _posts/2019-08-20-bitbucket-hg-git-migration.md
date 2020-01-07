---
permalink: bitbucket-hg-git-migration
title: Bitbucket Mercurial to Git Migration
layout: post
date: 2019-08-20 20:00:00
published: true
tags: bitbucket hg git mercurial
---

Back when [distributed version control systems](https://en.wikipedia.org/wiki/Distributed_version_control) were new, there were a few options. Nowadays, most developers would assume you're already talking about git. However, back in the day, Mercurial (hg) had much better tooling for windows than git. So I started moving everything from SVN and Source Safe to Mercurial.

Over the years I accumulated a few dozen Mercurial repositories and eventually starting using git like everyone else (thanks GitHub). But today I got an email from Bitbucket letting me know that [they're officially dropping support for Mercurial in June of 2020](https://bitbucket.org/blog/sunsetting-mercurial-support-in-bitbucket). It's the right move, but I still had a number of Mercurial repositories in Bitbucket.

They're encouraging everyone to migrate to git, but they don't offer an automated way to that. Thankfully, it's not too difficult.

## Migrating hg to git

The process is rather straight forward. Create a new git repository, clone your hg repository, use hg-git to push your hg commits to git, and then push to a remote git repository.

```powershell
# create a new git repository
git init --bare .\git-repo

# clone the hg repository
hg clone ssh://hg@bitbucket.org/username/repository hg-repo

cd hg-repo

# create a branch called hg
hg bookmarks hg

# push the hg repository to the local git repository
hg push ../git-repo

cd ..

cd git-repo

# push to a remote git repository
git remote add origin git@bitbucket.org:username/repository
git push --all origin
```

See Mark Heath's [How to Convert a Mercurial Repository to Git on Windows](https://markheath.net/post/how-to-convert-mercurial-repository-to) for more detailed information.

## Migrating with scripts

So it's simple, but what if you're like me and have dozens of repositories to convert? That's too much typing for me. I created a couple powershell scripts to do all of the heavy lifting:

<script src="//gist-it.appspot.com/http://github.com/jrummell/bitbucket-hg-migration/blob/master/migrate.ps1"></script>

<script src="//gist-it.appspot.com/http://github.com/jrummell/bitbucket-hg-migration/blob/master/push.ps1"></script>

So now I just needed to do the following for each repository:

1. `.\migrate.ps1 -BitbucketAccount username -Repository repo-name`
1. Rename the hg repository in Bitbucket
1. Create a new git repository in Bitbucket
1. `.\push.ps1 -BitbucketAccount username -Repository git-repo-name`

In case anyone else finds this useful, I've published the scripts to [Bitbucket](https://bitbucket.org/jrummell/bitbucket-hg-migration) and [GitHub](https://github.com/jrummell/bitbucket-hg-migration).
