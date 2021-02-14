# Git Workflow Etiquette

## Repository Organization for Clean Codebases

![Colorful Paint Header Image][gitlogo]

Version control with git provides for powerful collaboration, whether it be a tight-knit tech team or a distributed network in an open source format. Still, proper care must be taken. [Git][git] is used widely within the software development field and can manage a constantly evolving codebase. Most of its usefulness comes from the ability to branch off new work from an existing project in order to work on it in isolation until ready to integrate back into the project.

Good quality software consists of, among other things, code which is robust, resilient, secure, and performant. These attributes are achievable by maintaining a foundation of good quality code and a solid history of documentation. Anyone should be able to join the process and easily figure out and track the status of the project. This is where git and [GitHub][github] come in.

_Skip to the following_:

→ [Branch vs. Fork][branchfork]

→ [Rebase vs. Merge][rebasemerge]

→ [CLI][cli]

→ [Commit (Messages)][commitmsg]

→ [Pull Requests (and Reviews)][prreview]

→ [GitHub Issues][githubissue]

→ [Open Source][oss]

#### Approach

My experience as a contributor to the [Accord Project][apsite], an open source project for smart legal contracts, has led me to adopt and implement a standard of git workflow and etiquette which I find thorough and effective.

I will be sharing this as both a useful reference for others, as well as a historical reference for myself in the future. As is readily apparent by now, I am utilizing both git and GitHub. For perspective, you can reference GitHub’s official workflow proposal ([GitHub flow][ghflow]).

The majority of my work at the Accord Project is in [JavaScript][jsref], with some domain specific language mixed in, but the principles displayed here should apply to any language.

---

### <a name="branchFork"></a> Branch + Fork:

#### Branching

**Pros**

`+` All project work is centralized

`+` Ease of collaboration

`+` Single remote to handle

**Cons**

`—`  Deprecated branches may clutter easily

#### Forking

**Pros**

`+` Increased separation between user branches

`+` Primary repository cleanliness

**Cons**

`—` Difficulty to track branches

`—` Collaboration requires extra steps

`—` Accessibility is lower for less experienced git users

While there are benefits to both of these approaches, my general rule consists of forking in an open source project and branching in a smaller or insular team. There is more incentive for keeping the main repository clean and tidy in open source, and less chance of quick-and-dirty collaboration on a branch. Conversely, a tech team would benefit from a centralized repository with trackable branches. Either way will necessitate strict organization.

As an open source project, the Accord Project follows the fork workflow model. Still, the `main` branches should be kept in sync ([guide for syncing][syncguide]), and every feature, bug fix, release, or code change should occur in a different branch. This will be discussed more in the [next section][rebasemerge].

Naming a new branch will be one of the first steps in keeping a consistent tracking system of Issues, pull requests, and git history. The important factor here is consistency.

`name/issue-tracker/short-description`

**`name`**: Anything from initials to full name to GitHub username

**`issue-tracker`**: Reference the issue from GitHub or some other agile user stories source

**`short-description`**: One to three words describing the main goal of this branch, separated by hyphens

**Example**:

`irmerk/i7/new-feature`

In the case of collaborating on a single feature, maintain a single, `main` branch for the feature and individual branches from it. This can follow the previous naming convention:

`main/i14/routing-service // team branch`

`irmerk/i14/routing-service // my branch`

`someone/i14/routing-service // someone else's branch`

Personal branches can be merged into the `main` team branch, which will then be merged with the overall `main` through a [pull request][prreview]. Delete branches after they are merged.

### <a name="rebaseMerge"></a> Rebase + Squash

Common practice for teams is to squash or condense long commit message chains into one or a few commits before merging into `main`. This is useful when, like me, someone commits frequently and thus would clutter a git log. Squashing serves to maintain a readable git log.

Prior to merging a feature branch into the main (`main`) branch, it should be rebased from the up-to-date `main`. A [pull request][prreview], discussed later, will be where all the commits of this branch are squashed down to a single buildable commit and merged into `main`. Rebasing essentially ports a branch (`main`) into your current branch by applying all of your commits on top of that branch (`main`), then replacing your branch with the revised version. This catches you up to a branch (`main`) by rewriting git history in your local git.

Think of it as moving your branch to the tip of `main` instead of having branched off from an earlier version of `main`.

Instead of needing to heavily investigate individual commits from a branch, each merge commit to `main` should contain all the code for a feature or bug fix. This allows for a much easier investigation process.

While squashing prior to rebasing reduces conflicts due to fewer steps of conflict resolution, it literally changes the history of the repository as documented on GitHub for everyone, and thus is not the most accurate representation. Rebasing before squashing retains the git log tidiness and you don’t change history prior to documenting it on GitHub.

#### Interactive Rebase

`git rebase -i` is super useful in several circumstances, such as needing to quickly remove a commit. If your team has a policy in which any feature commit must also contain tests for that feature, squashing several commits into one can be helpful. This would involve `git rebase -i HEAD~n` and replace `n` with the number of commits — replace `pick` on those commits’ lines to `squash`.

If your project, like the Accord Project, requires a [Developer Certificate of Origin][dco] sign-off, you may find yourself needing to rapidly change the messages on a series of commits. Similarly to before, change `pick` to `edit` on the commits needing to change and simply `git commit --amend -s` and `git rebase --continue` for each commit. `git push -f` into the branch for the pull request. **Caution**: Force pushing can have dire consequences if not used properly, consult others if unsure how to use this.

Generally, if you do not have large, confusing conflicts, `-i` (interactive rebase) will be overkill.

#### Rewriting History

While rewriting history with `git rebase` is extremely useful in some cases, caution should still be taken. Make sure to not interrupt other people’s history and commits on accident. **Caution**: Avoid force pushing to a remote branch other people are working on.

#### Fear

I still fear `rebase` and `squash`. Merge conflicts can be more frequent and seem more difficult. I’ve had experiences of losing work due to incorrectly rebasing.

However, if you frequently commit ongoing work, rebasing complications should be infrequent. Fixing conflicts and `git rebase --continue` can feel intimidating at first, but continue working with it. A merge commit will be made with these conflicts, but that is important history on how a conflict was resolved. You can always try again if you feel it is going poorly with `git rebase --abort` and try again — this reverts too before the `rebase` attempt.

#### Flow

My recommendation for a general workflow:

1. Ensure you are currently in `main`
   → If working in a `fork`, fetch:

```shell
git checkout main
git fetch --all --prune
git rebase upstream/main
git push origin main
```

→ If working in a `branch`, pull:

```shell
git checkout main
git pull origin main
```

2. Create a new branch for your feature or bug fix

```shell
git checkout -b branchName
```

3. Make changes with as many commits as necessary. The final commit should build and pass tests.
4. Make sure your branch sits on top of `main` (as opposed to branch off a branch). This ensures the reviewer will need only minimal effort to integrate your work by fast-fowarding `main`

```shell
git rebase upstream/main
```

5. If you have previously pushed your code to a remote branch, you will need to force push. If not, omit the -f tag.

```shell
git push origin branchName -f
```

6. Open a pull request in GitHub from this forked branch. Once this has been merged to `main`, remember to clear out your branch locally

```shell
git branch -D branchName
```

#### Merge

As a non-destructive operation, merging can be nice. However, if the `main` you are working on is quite active, the git log can be polluted quickly by the extra commit created by a merge operation.

While many developers are uncomfortable with rebasing and resort to merge `main` on top of their changes, merging is not always the best option. If merging locally instead of rebasing, at the very least squash a pull request to merge into `main` on GitHub. This allows for greater control of the commit messages in `main` for the git log.

---

### <a name="cli"></a> CLI

The Command Line Interface (CLI) is a place you can run _all_ git commands. Git graphical user interfaces (GUI) such as [GitKraken][gitkraken] and [Tower][tower] are great. I opt to not use them due to them being priced as well as the vast majority of solutions found online involving git are for CLI.

Moreover, knowing the CLI method of git allows you to easily navigate a GUI, but the reverse is not necessarily the case. Learning to interact with git and GitHub via the CLI will be a great use of your time, especially if you work with open source projects.

#### Diff

A good habit to adopt is utilizing `git diff` prior to committing anything. This allows you to ensure the code being committed is what you expect, all debugging statements are removed, and no junk is included.

#### Log

Logs reveal a history of everything that happens in a repository. This tool has many options for displaying commit history in specific ways. A full log contains a commit hash, author, date, and message. My preferred way of logging builds an ASCII graph representing the branch structure

```shell
git log --graph --decorate --pretty=oneline --abbrev-commit
```

#### Blame

A method for inspecting who changed what and when in files is `git blame`. If you code in [VSCode][vscode], I strongly recommend looking into [GitLens][gitlens], which makes this inspection inline and extremely efficient.

---

### <a name="commitMsg"></a> Commits

A single logical change should be captured in a commit. More than one logical changes in a commit — an instance in which you may find yourself writing _“and”_ in a commit message — is a good indicator of needing to split into two separate commits.

Commit often — do not let yourself get too far without committing. Small, incremental and self contained commits are easier to follow or revert in the future.

While ordering commits logically would be ideal, I recommend committing in the order in which you are working — this is a chronological history of what you did.

#### Messages

Take a moment for this process — a commit message should not be rushed. The description of a commit should be well documented, and thus will prove invaluable to whoever reads this in the future in attempt to understand why a change was made — even if they have little to no context. This accessibility is a vital goal for a thorough git history and workflow.

Include external information references such as Issues or pull requests. Anything that will be helpful to others or your future self should be reasoned out now. The long term success of a project relies on the maintainability of the code and log. A hassle at first pays off as a healthy habit.

Use the terminal, not the editor, when writing a commit message. Committing from the terminal encourages a mindset of describing a change in an incremental way, as well as keeping commits atomic — commits should not need a paragraph of explanation. This will assist you in creating a pull request message, where an overall change should be captured. More on this later.

Concise and consistent commit messages should be captured by always including the `-m <msg>` flag to a `git commit`.

#### Formatting

A properly formed `git commit` subject line should always be able to complete the following sentence:

> If applied, this commit will **`your subject line here`**

`type(scope): subject — footer`

**Types**:

- `feat`— A new feature

- `fix`— A bug fix

- `docs`— Changes to only documentation

- `style`— Changes to formatting (missing semi colons, etc.)

- `refactor`— A code change that neither fixes a bug nor adds a feature

- `test`— Adding missing or correcting existing tests

- `chore`— Change to build process or auxiliary tools, or maintenance

**Scope**:

- Focal point of new code or best description for where changes can be found.

**Subject**:

- Imperative description of changes, kept under 50 characters (not capitalized and no period)

**Footer**:

- GitHub Issue reference ID

Examples

```shell
feat(utilities): include optional state for test utility - I22
    // implement a feature in src/utilities/test/sagaTest.js
test(sagas): initiate test for failed templates fetch - I22
    // tests run in and of src/sagas/*
chore(package): install saga testing library - I22
    // edits to dependencies in package.json
docs(README): add netlify badge and update - I27
    // alter documentation in the readme file
```

### <a name="prReview"></a> Pull Requests

A pull request (PR) is one of the best ways to share information. While an Issue describes what may be wrong or a feature, a PR provides a medium for what changes are actually occurring to the codebase. Moreover, it is excellent for peer reviews and accountability, as it encourages quality commits. When done well, the commits that construct a PR tell the whole story to those who review the code or examine it in the future.

PRs should consist of a complete addition to the code which contains value. Because the commits inside follow a pattern, the title should be an extension or summary of all the commits inside. Thus, earlier when I spoke of squashing, a git log will retain the pattern of each commit even after it is tidied up. To emphasize, **a PR title should follow commit message formatting described [above][commitmsg]**.

As a GitHub workflow tool, the innards of a PR are less important than the title maintaining the consistency and efficiency of formatting. This allows git logs to remain efficient with or without GitHub.

Similar to commits, PRs should be small. A PR which attempts to do multiple things (find yourself writing _“and”_ in the title?) should be split up.

#### Cleanup

As discussed previously, rebasing prior to creating a PR is a good tidying habit. Take a moment to merge any extra commits created along the way, or reword commits for clarity. Every commit in a PR should be directly working towards the goal of the title of the PR, or even the related GitHub Issue.

#### Formatting

```md
<!--- Provide a formatted commit message describing this PR in the Title above -->
# Closes #<CORRESPONDING ISSUE NUMBER>
<!--- Provide an overall summary of the pull request -->

### Changes
<!--- More detailed and granular description of changes -->
<!--- These should likely be gathered from commit message summaries -->
- <ONE>
- <TWO>

### Flags
<!--- Provide context or concerns a reviewer should be aware of -->
- <ONE>
- <TWO>

### Screenshots or Video
<!--- Provide an easily accessible demonstration of the changes, if applicable -->

### Related Issues
- Issue #<NUMBER>
- Pull Request #<NUMBER>

### Author Checklist
- [ ] Ensure you provide a [DCO sign-off](https://github.com/probot/dco#how-it-works) for your commits using the `--signoff` option of git commit.
- [ ] Vital features and changes captured in unit and/or integration tests
- [ ] Extend the documentation, if necessary
- [ ] Merging to `main` from `fork:branchname`
- [ ] Manual accessibility test performed
    - [ ] Keyboard-only access, including forms
    - [ ] Contrast at least WCAG Level A
    - [ ] Appropriate labels, alt text, and instructions
```

GitHub PRs are in Markdown
Keep in mind: What does this change do to address the issue and what side effects may it have? Why was it necessary? Try to preempt a reviewer needing to ask questions, be thorough in your information.

#### Drafts

GitHub offers a useful option for a PR which is not ready to be reviewed quite yet. If you want to have a source of truth in a PR for others to interact with prior to actual review, or even just to ensure the code is saved on GitHub and not only on your local machine, open a draft pull request.

#### Reviews

Setting a standard for PR reviews is important, and being thorough equally so. A reviewer is a guardian of the git history and code quality. This cannot be stressed enough. What seems obvious now will surely not be so in months’ or years’ time. Do not feel bad for requesting changes or having changes requested of you. It is better to have pristine code merged into `main` than to rush through out of desire to be done with a feature.

Strike a balance between the flow of getting PRs through and not holding up further edits or production, and maintaining quality. Every reviewer should make a judgment on whether an issue is sufficient enough to block a PR. However, anyone who contributes to the PR should not be reviewing what is merged into `main`. Keep a healthy separation between these roles. Generally, it is faster for the PR author to fix the code than for a reviewer to be involved.

---

### <a name="githubIssue"></a> GitHub

#### Issues

Very easy to maintain, Issues should be used liberally. Any question, idea, or bug — duplicate or not — should be reason enough to open an Issue. These are the foundation of conversations in projects, so insert points of view and establish a concrete record of discussions which can be searched and linked.

Speaking of which, search through a project before opening an Issue. While duplicates can be easily closed, this is a courtesy to other contributors. Also, be thorough and provide as much context as possible. Maybe even take a moment to read the project’s documentation on contributing, they may have guidelines on formatting an Issue.

Example feature request formatting of an Issue:

```md
<!--- Provide a general summary of the feature in the Title above -->
# Feature Request 🛍️
<!--- Provide an expanded summary of the feature -->

## Use Case
<!--- Tell us what feature we should support and what should happen -->

## Possible Solution
<!--- Not obligatory, but suggest an implementation -->

## Context
<!--- How has this issue affected you? What are you trying to accomplish? -->
<!--- Providing context helps us come up with a solution that is most useful in the real world -->

## Detailed Description
<!--- Provide a detailed description of the change or addition you are proposing -->
```

#### Noise

Stay mindful of what you are presenting for others to read. Is it worth their time? Avoid posting short, rushed answers or responses. Take the time to be worthwhile to yourself and others in your contributions. Provide as much context as possible while still being helpful. Too little and too much context are both quite awful.

If a significant amount of time has passed, send a gentle bump to reviewers. It can be easy to forget about an active PR.

#### <a name="oss"></a> Open Source

I recommend this [guide][ossguide] for contributing to open source. GitHub provides great functionalities which should be utilized for the best possible experience of contributing developers. Issues can be tagged for ease of navigation, protections can be enforced to prevent direct commits to `main`, and multiple reviewers can be required.

This guide is aimed at providing a framework which fosters a healthy, collaborative atmosphere. All of this can be applied and tweaked for any open source or proprietary tech team environment.

---

### Conclusion

Contribute the work you would like to see. Politeness, respect, thoroughness, and efficiency are all things we appreciate, so develop and maintain good habits with git and collaborations will be the best they can be.

Perception is important in open source, and how others view the git log can make a big difference. Take time to care for the log with efficient commits and git etiquette. This is also true for a company and a new hire.

Feel free to contact me with any questions or feedback.

[gitlogo]: ../images/gitLogo.png
[git]: https://git-scm.com/
[github]: https://github.com/
[branchfork]: git-workflow.md#branchFork
[rebasemerge]: git-workflow.md#rebaseMerge
[cli]: git-workflow.md#cli
[commitmsg]: git-workflow.md#commitMsg
[prreview]: git-workflow.md#prReview
[githubissue]: git-workflow.md#githubIssue
[oss]: git-workflow.md#oss
[apsite]: https://www.accordproject.org/
[ghflow]: https://guides.github.com/introduction/flow/
[jsref]: https://developer.mozilla.org/en-US/docs/Web/JavaScript
[syncguide]: https://help.github.com/en/articles/syncing-a-fork
[dco]: https://developercertificate.org/
[gitkraken]: https://www.gitkraken.com/
[tower]: https://www.git-tower.com/
[vscode]: https://code.visualstudio.com/
[gitlens]: https://gitlens.amod.io/
[ossguide]: https://opensource.guide/how-to-contribute/
