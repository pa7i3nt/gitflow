# Gitflow and Workflow

**Treat public history as immutable, atomic, and easy to follow.
Treat private history as disposable and malleable.**

---

# Requirements and Tips

- Use one branch per feature/bug (contained development).  
  Every tick/task/feature MUST be implemented on a separate branch. Basically each JIRA ticket should be represented by at least one branch.
- Only release managers are allowed to work on the central master branch.
- Cherry picking MUST not be used by any means.
- A good use of branches should prevent the need of "git cherry-pick".
- DO NOT create very large repositories.
- DO NOT commit large binary files.
- DO NOT commit any file, which can be regenerated or which is generated automatically by your development environment.
- Remember to rebase your feature branch before merging it to the Development branch.
- Specify the origin and branch when pushing (might avoid mistakes).
- Use "git pull --rebase" in order to avoid merges from upstream commits.

# Git branching model

### Without release branches

![Alt text](./images/gitworkflow_fig1.png)

### With release branches

![Alt text](./images/gitflow_fig1.png)

# Naming Convention

## Branch Naming

Must:

- Include a short descriptive summary in imperative present tense
- Use Hyphens for separating words

May:

- Include the work type: feature, refactor, bugfix, hotfix, etc.
- Include corresponding ticket/story id (e.g. from Jira, GitHub issue, etc.)

Suggested Format:  
_{work type e.g.}/{2-3 word summary}/{story or ticket id}_

Example:

```git
git checkout -b feature/oauth-migration/ATL-244
```

## Commit Message Naming

Consist of two parts:

- Subject: Short informative summary of the commit
- Body: More detailed explanatory text if needed

### Subject:

- Short and descriptive (max 50 chars)
- Capitalized
- In imperative present tense
- Not end with period

Example:

```
Implement access right management
```

### Body:

- Separated with a blank line from the subject
- Explain what, why, etc.
- Max 72 chars
- Each paragraph capitalized

Example:

```
Implement proper authorization for each service on development phase to validate during the API call.

Access right management is used to check proper authorization to access an API by an employee or the employer.
```

### Conventional Commits

Commit messages **MAY** use [Conventional Commits](https://www.conventionalcommits.org/en/) format. It provides guidelines to create a better commit history log, making easier to have automated tasks around it (e.g. automated changelogs). Commits would follow the format `<type>[optional scope]: <description>`, where `<type>` might be `feat`/`fix`/`chore`/`docs` etc. and breaking changes are indicated on the beginning of the optional body or footer section.

Example:

```
git commit -m "feat(survey): add nps survey to the home page

BREAKING CHANGE:  `survey` objects in xml file have been re-used in the global configurations.
```

Please refer to [Conventional Commits docs](https://www.conventionalcommits.org/en/) for more details

## Pull Request Naming

Consists of two parts:

- Title: Short informative summary of the pull request
- Description: More detailed explanatory text describing the PR for the reviewer

### Subject:

- Short and descriptive summary
- Start with corresponding ticket/story id (e.g. from Jira, GitHub issue, etc.)
- Should be capitalized and written in imperative present tense
- Not end with period

Suggested Format:  
_#[Ticket_ID] PR description_

Example:

```
#CLS-23 Add Edit on Github button to all the pages
```

### Description:

- Separated with a blank line from the subject
- Explain what, why, etc.
- Max 72 chars
- Each paragraph capitalized

Example:

```
This pull request is part of the work to make it easier for people to contribute to naming convention guides. One of the easiest way to make small changes would be using the Edit on Github button.

To achieve this, we needed to:
- Find the best Gitbook plugin which can do the work
- Integrate it in all the pages to redirect the user to the right page on GitHub for editing
- Make it visible on the page so users can notice it easily
```

## Tag Naming

May:

- Use [Semantic Versioning](https://semver.org/)
- Prefix the version number with the letter _v_

Suggested Formats:

- With prefix _{vX.Y.Z}_
- Without prefix _{X.Y.Z}_

Example:

```git
git tag -a 1.0.2 // or optionally v1.0.2
```

# The main branches

At the core, the development model is greatly inspired by existing models out there. The central repo holds two main branches with an infinite lifetime:

- `master`
- `develop`

![Alt text](./images/gitflow_fig2.png)

The `master` branch at `origin` should be familiar to every Git user. Parallel to the `master` branch, another branch exists called `develop`.

We consider `origin/master` to be the main branch where the source code of `HEAD` always reflects a _production-ready_ state.

We consider `origin/develop` to be the main branch where the source code of `HEAD` always reflects a state with the latest delivered development changes for the next release. Some would call this the “integration branch”. This is where any automatic nightly builds are built from.

When the source code in the `develop` branch reaches a stable point and is ready to be released, all of the changes should be merged back into `master` somehow and then tagged with a release number. How this is done in detail will be discussed further on.

Therefore, each time when changes are merged back into `master`, this is a new production release by definition. We tend to be very strict at this, so that theoretically, we could use a Git hook script to automatically build and roll-out our software to our production servers everytime there was a commit on `master`.

## master branch

Contains all the stable, released code.

- All released versions of all modules should be tagged in the master.
- No separate branches for the released versions.
- The master branch is ready to build at any moment.
- No development should be performed on the master branch directly.
- Only release managers have write permissions on it.
- The master branch rolls only forward, no history changes are allowed on the master branch.
- All new patches are introduced in the master branch only via "git merge --ff-only".

## dev branch

- The branch is inherited from the latest master.
- The dev branch is a development mainstream.
- A release manager defines a list of tasks for a development sprint. For each task, developers MUST create a separate branch inherited from the **dev** branch.
- Should be rebased from the **master** each time **master** is changed.

# Supporting branches

Next to the main branches `master` and `develop`, our development model uses a variety of supporting branches to aid parallel development between team members, ease tracking of features, prepare for production releases and to assist in quickly fixing live production problems. Unlike the main branches, these branches always have a limited life time, since they will be removed eventually.

The different types of branches we may use are:

- Feature branches
- Release branches
- Hotfix branches

Each of these branches have a specific purpose and are bound to strict rules as to which branches may be their originating branch and which branches must be their merge targets. We will walk through them in a minute.

By no means are these branches “special” from a technical perspective. The branch types are categorized by how we use them. They are of course plain old Git branches.

## Feature branches

May branch off from: `develop`

Must merge back into: `develop`

Branch naming convention: anything except `master`, `develop`, `release-_*`, or `hotfix-_*`

![Alt text](./images/gitflow_fig3.png)

Feature branches (or sometimes called topic branches) are used to develop new features for the upcoming or a distant future release. When starting development of a feature, the target release in which this feature will be incorporated may well be unknown at that point. The essence of a feature branch is that it exists as long as the feature is in development, but will eventually be merged back into `develop` (to definitely add the new feature to the upcoming release) or discarded (in case of a disappointing experiment).

It is recommended to keep feature branches even after their merge with **dev**. It will simplify fine tuning in case if the feature represented by the branch will be reverted from the **dev** for additional development or fixes/corrections.

Feature branches should be deleted as soon as their commits are merged into the **master** via the **dev** branch.

Feature branches typically exist in developer repos only, not in `origin`.

<div align="center" style="color:#db5208"><span><b>Creating a feature branch</b></span></div><br>

When starting work on a new feature, branch off from the `develop` branch.

```git
$ git checkout -b myfeature develop
```

<div align="center" style="color:#db5208"><span><b>Incorporating a finished feature on develop</b></span></div><br>

Finished features may be merged into the `develop` branch to definitely add them to the upcoming release:

```git
$ git checkout develop
$ git merge --no-ff myfeature
$ git branch -d myfeature
$ git push origin develop
```

Quá trình trên được thực hiện tương tự khi tạo một Merge Request (MR).

The `--no-ff` flag causes the merge to always create a new commit object, even if the merge could be performed with a fast-forward. This avoids losing information about the historical existence of a feature branch and groups together all commits that together added the feature. Compare:

![Alt text](./images/gitflow_fig4.png)

In the latter case, it is impossible to see from the Git history which of the commit objects together have implemented a feature—you would have to manually read all the log messages. Reverting a whole feature (i.e. a group of commits), is a true headache in the latter situation, whereas it is easily done if the `--no-ff` flag was used.

Yes, it will create a few more (empty) commit objects, but the gain is much bigger than the cost.

## Release branches

May branch off from: `develop`

Must merge back into: `develop` and `master`

Branch naming convention: `release-*`

Release branches support preparation of a new production release. They allow for last-minute dotting of i’s and crossing t’s. Furthermore, they allow for minor bug fixes and preparing meta-data for a release (version number, build dates, etc.). By doing all of this work on a release branch, the `develop` branch is cleared to receive features for the next big release.

The key moment to branch off a new release branch from `develop` is when develop (almost) reflects the desired state of the new release. At least all features that are targeted for the release-to-be-built must be merged in to `develop` at this point in time. All features targeted at future releases may not—they must wait until after the release branch is branched off.

It is exactly at the start of a release branch that the upcoming release gets assigned a version number—not any earlier. Up until that moment, the `develop` branch reflected changes for the “next release”, but it is unclear whether that “next release” will eventually become 0.3 or 1.0, until the release branch is started. That decision is made on the start of the release branch and is carried out by the project’s rules on version number bumping.

<div align="center" style="color:#db5208"><span><b>Creating a release branch</b></span></div><br>

Release branches are created from the `develop` branch. For example, say version 1.1.5 is the current production release and we have a big release coming up. The state of `develop` is ready for the "next release" and we have decided that this will become version 1.2 (rather than 1.1.6 or 2.0). So we branch off and give the release branch a name reflecting the new version number:

```git
$ git checkout -b release-1.2 develop
$ git commit -a -m "Bumped version number to 1.2"
```

After creating a new branch and switching to it, we bump the version number. Suppose we change some files in the working copy to reflect the new version. (This can, of course, be a manual change—the point being that some files change.).Then, the bumped version number is committed.

This new branch may exist there for a while, until the release may be rolled out definitely. During that time, bug fixes may be applied in this branch (rather than on the `develop` branch). Adding large new features here is strictly prohibited. They must be merged into `develop`, and therefore, wait for the next big release.

<div align="center" style="color:#db5208"><span><b>Finishing a release branch</b></span></div><br>

When the state of the release branch is ready to become a real release, some actions need to be carried out. First, the release branch is merged into `master` (since every commit on `master` is a new release by _definition_, remember). Next, that commit on `master` must be tagged for easy future reference to this historical version. Finally, the changes made on the release branch need to be merged back into `develop`, so that future releases also contain these bug fixes.

The first two steps in Git:

```git
$ git checkout master
$ git merge --no-ff release-1.2
$ git tag -a 1.2
```

The release is now done, and tagged for future reference.

To keep the changes made in the release branch, we need to merge those back into `develop`, though. In Git:

```git
$ git checkout develop
$ git merge --no-ff release-1.2
```

This step may well lead to a merge conflict (probably even, since we have changed the version number). If so, fix it and commit.

Now we are really done and the release branch may be removed, since we don’t need it anymore:

```git
$ git branch -d release-1.2
```

## Hotfix branches

May branch off from: `master`

Must merge back into: `develop` and `master`

Branch naming convention: `hotfix-*`

![Alt text](./images/gitflow_fig5.png)

Hotfix branches are very much like release branches in that they are also meant to prepare for a new production release, albeit unplanned. They arise from the necessity to act immediately upon an undesired state of a live production version. When a critical bug in a production version must be resolved immediately, a hotfix branch may be branched off from the corresponding tag on the master branch that marks the production version.

The essence is that work of team members (on the `develop` branch) can continue, while another person is preparing a quick production fix.

<div align="center" style="color:#db5208"><span><b>Creating the hotfix branch</b></span></div><br>

Hotfix branches are created from the `master` branch. For example, say version 1.2 is the current production release running live and causing troubles due to a severe bug. But changes on `develop` are yet unstable. We may then branch off a hotfix branch and start fixing the problem:

```git
$ git checkout -b hotfix-1.2.1 master
```

Change some files...

```git
$ git commit -a -m "Bumped version number to 1.2.1"
```

Don’t forget to bump the version number after branching off!

Then, fix the bug and commit the fix in one or more separate commits.

```git
$ git commit -m "Fixed severe production problem"
```

<div align="center" style="color:#db5208"><span><b>Finishing a hotfix branch</b></span></div><br>

When finished, the bugfix needs to be merged back into `master`, but also needs to be merged back into `develop`, in order to safeguard that the bugfix is included in the next release as well. This is completely similar to how release branches are finished.

First, update `master` and tag the release.

```git
$ git checkout master
$ git merge --no-ff hotfix-1.2.1
$ git tag -a 1.2.1
```

Next, include the bugfix in `develop`, too:

```git
$ git checkout develop
$ git merge --no-ff hotfix-1.2.1
```

The one exception to the rule here is that, **when a release branch currently exists, the hotfix changes need to be merged into that release branch, instead of** `develop`. Back-merging the bugfix into the release branch will eventually result in the bugfix being merged into `develop` too, when the release branch is finished. (If work in `develop` immediately requires this bugfix and cannot wait for the release branch to be finished, you may safely merge the bugfix into `develop` now already as well.)

Finally, remove the temporary branch:

```git
$ git branch -d hotfix-1.2.1
```

# References

- http://nvie.com/posts/a-successful-git-branching-model/
- https://help.github.com/articles/using-pull-requests#shared-repository-model
- https://github.com/AnarManafov/GitWorkflow/blob/master/GitWorkflow.markdown
- https://github.com/naming-convention/naming-convention-guides/tree/master/git
