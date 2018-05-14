# A Guide to Using Git and Git-TFS for Local Development with TFVC Codebases

- [A Guide to Using Git for Local Development for TFVC Codebases](#a-guide-to-using-git-for-local-development-for-tfvc-codebases)
    - [Introduction](#introduction)
    - [Why Use Git-TFS?](#why-use-git-tfs)
    - [Using Git-Tfs to Work Locally with Git](#using-git-tfs-to-work-locally-with-git)
        - [Getting up and Running with Git-Tfs](#getting-up-and-running-with-git-tfs)
            - [Prerequisites](#prerequisites)
            - [Installing Git for Windows](#installing-git-for-windows)
            - [Installing Git-Tfs](#installing-git-tfs)
        - [Cloning a TFS Root Branch](#cloning-a-tfs-root-branch)
        - [Cloning an Existing `git-tfs` Repository](#cloning-an-existing-git-tfs-repository)
        - [Copying an Existing git-tfs git Repository](#copying-an-existing-git-tfs-git-repository)
        - [Creating Helpful Command Aliases](#creating-helpful-command-aliases)
        - [Git-Tfs Commit Messages](#git-tfs-commit-messages)
            - ["Good" Git-TFS Commit Messages](#good-git-tfs-commit-messages)
        - [Git-TFS Recipes](#git-tfs-recipes)
            - [I've Just Cloned Git-TFS Git Repository and Checked Out a Branch Which Exists in TFS; What Do I Do Next?](#i-ve-just-cloned-git-tfs-git-repository-and-checked-out-a-branch-which-exists-in-tfs-what-do-i-do-next)
            - [I Need to Checkout an Existing TFS Branch That's Not Part of the Git Repository Yet](#i-need-to-checkout-an-existing-tfs-branch-that-s-not-part-of-the-git-repository-yet)
            - [I Need to Implement a New Feature and Merge it into the TFS Branch](#i-need-to-implement-a-new-feature-and-merge-it-into-the-tfs-branch)

## Introduction
This document will help you learn how you can use the Git-TFS bridge to work locally using the git version control system while still enabling you to "push" your changes back to the main TFS project's TFVC repository.


## Why Use Git-TFS?
This is a phenomenal tool that I found online that, while still technically alpha/beta, is stable enough to be generally used for day-to-day work. When new versions come out, there are sometimes breaking changes to command line parameters. But beyond that, this tool has been a real time-saver.
Git-TFS allows you to "clone" a Team Foundation Version Controlled Team Project into a local git repository and maintain bindings/mappings back to the TFS branches. Why would you want to do this?
For one, you must understand how branching works in TFS. Branching in TFS copies every single file to a completely new location within the TFVC system. Therefore, branching is time-consuming and space-inefficient. Also, because of TFVC's centralized nature, it's very hard to avoid the "big-bang" merge scenario.
With git, you get cheap branching. Because there's no centralized server, each individual has their own copy of the source code with complete change history on their local machine. Branching is (I'm over simplifying this) just a labeled commit. As you create new commits, the HEAD of the branch gets updated. This makes branching extremely cheap--you're just creating a label at a specific point in time. And through operations like rebasing and merging, that label can move to keep you "up to date" with the parent branch which can help avoid the "big-bang" merge scenario.
So what does using Git locally provide for you when your main source code control repository is TFVC? Well, locally, you get cheap branching: you can branch, try stuff out, and throw it away, all without contacting any centralized server. Using git-tfs, should you want to keep the changes, the git-tfs bridge allows you to synchronize your git repository with the TFVC repository while maintaining authorship, git commit sha to TFS changeset ID linkages, and work item tracking.

> NOTE:  
> Once you have finished reading this guide, please make sure you understand how the various scripts work. All the scripts accept a `--help` argument.

## Using Git-Tfs to Work Locally with Git

### Getting up and Running with Git-Tfs

#### Prerequisites
You must have the following pre-requisite software installed to use Git-Tfs:
- Visual Studio (2010 through 2015)
- [Git for Windows](https://git-for-windows.github.io/)

#### Installing Git for Windows
When you install Git for Windows, be sure to use MinTTY as the terminal emulator.

#### Installing Git-Tfs
1. First, you should clone the repository containing this How-To document.
2. In the folder containing this document, there is a `Scripts` sub-folder which contains various Git-TFS helper scripts that can make working with Git-TFS, TFS, and the centralized Git server much easier than having to remember the sequence of various commands that need to be executed in order to keep everything in synch. After cloning, you should add the full path to the `Scripts` folder to your `PATH` User Environment variable.
3. Download and install the latest version of [Git for Windows](https://github.com/git-for-windows/git/releases).
    * When installing, ensure that you select the option to use git from both bash and the Windows Command Prompt.
    * Leave all other options at their default value.
4. [Download the Zip Archive file containing the latest git-tfs release.](https://github.com/git-tfs/git-tfs/releases).
5. Create a folder on your `D:\` drive called `bin`.
6. Create a folder in `D:\bin` called `git-tfs`.
7. Unzip the contents of the Zip Arhcive you downloaded in **step 2**.
8. Move the contents of the Zip Archive folder into the folder created in **step 4**.
9. Add the path `D:\bin\git-tfs` to the end of your machine/system **PATH** environment variable.
10. Add the environment variable **`GIT_TFS_CLIENT`** to your machine/system environment and set its value to the version of Visual Studio you are using. For example, if you're using Visual Studio 2015, then set it's value to `2015`.
11. Open PowerShell as an Administrator
12. Run the following commands in PowerShell:

    ```powershell
    PS C:\Windows\System32> D:
    PS D:> cd bin\git-tfs
    PS D:\bin\git-tfs> Get-ChildItem . -Recurse -File | For-Each { Unblock-File $_.FullName }
    ```

### Cloning a TFS Root Branch
You will only need to do this if there's no shared Git repository already available for cloning. If there's already a shared Git repository which has been initialized from a root TFS branch, then you should skip these instructions and proceed to <u>Cloning an Existing Git Repository</u> below.

> **WARNING**  
> This will take a very long time. Note that this is not I/O bound, it's compute bound. In order to "link" everything up, each TFS changeset must be put through the SHA1 hashing algorithm to compute the Git commit ids. If you already know someone who has cloned the TFS root branch you're interested in, you should see the next section, <u>Copying or Cloning an Existing git-tfs git Repository</u> below.

1. Open a **Visual Studio 2015 Developer Command Prompt** and execute the following commands: 

> NOTE:  
>  
> * Ensure you replace `<PROJECT-COLLECTION>` with your TFS Project Collection name.  
> * Ensure you replace `<TFS-SERVER>` with the name of your TFS Server and `<URL-ENCODED-TFS-COLLECTION-NAME>` with your TFS collection name. If your TFS collection name has, for example, a space character in it, you should URL encode it, e.g. `My Collection` becomes `My%20Collection`.)
>  
> The last command will take quite a few minutes to run.  

    ```bash
	C:\> D:
	D:\> mkdir repos
	D:\> cd repos
	D:\> tf history $/<PROJECT-COLLECTION> /collection:"http://<TFS-SERVER>:8080/tfs/<URL-ENCODED-TFS-COLLECTION-NAME>" /recursive /format:detailed > authors_tmp.txt
    ```

2. Open **Git Bash** and run the following commands:

> NOTE:  
> * Ensure you replace `<YOUR-WINDOWS-USER-DOMAIN>` with your Windows domain when written in the `DOMAIN\Username` syntax.  
> * Ensure you replace `<YOUR-EMAIL-DOMAIN>` with your organization's email address domain.  
> * Ensure you replace `<TFS-SERVER>` and `<URL-ENCODED-TFS-COLLECTION-NAME>` in the same manner as above.  
> * Ensure you replace `$/<PROJECT-COLLECTION>/<BRANCH-PATH>` with the appropriate TFS Team Project collection name and branch path.  
> * _(Optional)_ Specify the git repository name. It defaults to last segment of the `<BRANCH-PATH>` name if not specified.
>  
> The `git tfs clone...` command may take a really long time to run depending on how much history there is to clone. I've had it take 3 days on my organization's rather large TFS project. Smaller TFS projects take only a few minutes to a few hours to run. See `git-tfs help clone` for additional options for cloning your TFS project collection.  

    ```bash
    $ cd /d/repos
    $ grep -Ex "User: (.*), (.*)" authors_tmp.txt | sort | uniq | sed -r 's/User: (.*), (.)(.*)/<YOUR-WINDOWS-USER-DAMIN>\\\2\1 = \2\3 \1 <\2\3\.\1@<YOUR-EMAIL-DOMAIN>\.com>/' > authors.txt
    $ MSYS_NO_PATHCONV=1 git tfs clone -l -x --branches=all --authors=authors.txt "http://<TFS-SERVER>:8080/tfs/<URL-ENCODED-TFS-COLLECTION-NAME>" "$/<PROJECT-COLLECTION>/<BRANCH-PATH>" <GIT-REPO-NAME>
    $ cd <GIT-REPO-NAME>
    $ git remote add origin <url-to-origin>
    $ git config --local --add remote.origin.fetch +refs/notes/*:refs/notes/*
    $ git config --local core.autocrlf false
    $ git config --local core.whitespace trailing-space,space-before-tab,cr-at-eol
    $ git push -u origin refs/notes/* master
    ```

Now you have a fully cloned copy of the TFS Team Project and the root branch you specified. You should treat the `master` branch as if it were an actual git `master` branch and **not make any checkins directly to this branch**. This will be discussed further, below.

### Cloning an Existing `git-tfs` Repository
Cloning from an existing git repository is a bit more complicated than simply copying someone's git-tfs repository because you need to run some extra commands to bind all the branches back up to the TFS branches. But, this method is the preferred method to obtain a git-tfs git repository.

First, run the instructions to create an `authors.txt` file (see above). Then, run the following set of commands in a command shell of your choice:

```bash
$ cd /d/Repos
$ git clone -c core.autocrlf=false \
            -c core.whitespace=trailing-space,space-before-tab,cr-at-eol \
            -c remote.origin.fetch=+refs/notes/*:refs/notes/* \
            -c git-tfs.ignore-branches=False \
            -c git-tfs.export-metadatas=true \
            git@someplace:shared/repo.git

$ cd repo
$ cp ../authors.txt .git/git-tfs_authors        # Save the authors.txt file in the .git folder as git-tfs_authors 
$ git checkout master                           # You should already be on master, but just in case...
$ git fetch origin +refs/notes/*:refs/notes/*   # This gets work-item tracking history for the commits
$ git tfs-remote -v bootstrap                   # Hook the branch back up to TFS and add special "tracking" information to your git configuration
$ git tfs fetch -x                              # Get the latest TFS changesets
$ git tfs pull -x -r                            # Pull and rebase the changesets.
```

### Copying an Existing git-tfs git Repository
If someone you know already has a git repository cloned from the TFS root branch you're interested in working out of, it will be much faster for you to copy their repository (or set up a remote and clone from it) in order to get your initial local repository setup. If you want to clone from this repository (instead of copy it) please see above for cloning git-tfs git repositories.

Copy the existing git-tfs git repository to your local machine in a desired location. It's a good idea if the location is as close to the root of a drive as possible due to path length limitations of .NET Framework (and Windows Explorer on Windows 7 and below). Note that you may need to perform the following commands after copying the existing repository:

```bash
$ cd <PATH TO YOUR COPIED GIT-TFS GIT REPOSITORY>
$ git reset --hard
$ git checkout master
$ git tfs bootstrap
$ git tfs fetch
$ git tfs pull -r
```

### Creating Helpful Command Aliases
To make working with git-tfs a bit more painless, it's helpful to create some aliases for common git-tfs operations that make working with git and git-tfs a bit more seamless.

We'll be making aliases for the following commands:

| Alias        | Full Command                                         | Aliased Command                       | Description                                                                                                                                                     |
|:-------------|:-----------------------------------------------------|:------------------------------------- |:----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `co`         | `git checkout <branch>`                              | `git co <branch>`                     | Checkout a git branch                                                                                                                                           |
| `cob`        | `git checkout -b <new branch>`                       | `git cob <new branch>`                | Create branch `<new branch>` and check it out                                                                                                                   |
| `ci`         | `git add <files>`                                    | `git ci <files>`                      | Adds changed/untracked files to the working index (stage)                                                                                                       |
| `lol`        | `git log --oneline -40`                              | `git lol`                             | Logs 40 commits (by default) using the `oneline` format.                                                                                                        |
| `tree`       | `git log --decorate --oneline --graph -40`           | `git tree`                            | Show a tree view of the most recent 40 commits. You can override the number of commits, e.g. git tree -10 shows the last 10 commits as a tree.                  |
| `st`         | `git status`                                         | `git st`                              | Show the status of your working index                                                                                                                           |
| `tfind`      | `git tfs checkout -n <TFS Changeset ID>`             | `git tfind <changeset>`               | Finds the git commit associated with the specified `<TFS changeset ID>`.                                                                                        |
| `tbootstrap` | `git tfs-remote -v bootstrap`                        | `git tbootstrap`                      | Bootstraps your Git branch with the TFS branch.                                                                                                                 |
| `tremote`    | `git tfs-remote [-v] [add | bootstrap]`              | `git tremote [-v] [add | bootstrap]`  | A helper script to add a "TFS tracking remote" to a git branch.                                                                                                 |
| `tfetch`     | `git tfs fetch -x`                                   | `git tfetch`                          | Fetch the latest changesets from the TFS branch remote.                                                                                                         |
| `tpull`      | `git tfs pull -r -x`                                 | `git tpull`                           | Pull the latest changes from the TFS branch remote to your local branch                                                                                         |
| `tsync`      | `git tfs-sync`                                       | `git tsync`                           | A helper script to synchronize your local git branch and the upstream git branch with changes to the TFS branch.                                                |
| `tmerge`     | `git tfs-merge-into`                                 | `git tmerge`                          | A helper script to update your feature's parent TFS branch and check-in your feature branch to the parent, keeping TFS and the centralized Git server in synch. |
| `tco`        | `MSYS_NO_PATHCONV=1 git tfs branch --init <branch>`  | `MSYS_NO_PATHCONV=1 git tco <branch>` | Checks out the branch from TFS and pulls down the changesets to your local repo.                                                                                |

Open a git bash prompt and run the following commands:

```bash
$ git config --global alias.co checkout
$ git config --global alias.cob "checkout -b"
$ git config --global alias.ci "add"
$ git config --global alias.lol "log --oneline -40"
$ git config --global alias.tree "log --decorate --oneline --graph -40"
$ git config --global alias.st "status"
$ git config --global alias.tfind "tfs checkout -n"
$ git config --global alias.tbootstrap "tfs-remote -v bootstrap"
$ git config --global alias.tremote "tfs-remote"
$ git config --global alias.tfetch "tfs fetch -x"
$ git config --global alias.tpull "tfs pull -x"
$ git config --global alias.tsync "tfs-sync"
$ git config --global alias.tmerge "tfs-merge-into"
$ git config --global alias.tco "tfs branch --init"

# Run this command if you want to make sure you typed everything correctly:
$ git config --global --list | grep -e "alias"
```

One final note regarding the `git tco` alias. Git for Windows uses something called _MSYS_, which is a minimal Gnu system for Windows.
It has a nasty habit of "helpfully" translating things that "look like" a path. When attempting to initialize a TFS branch (whether or not you are using the alias),
MSYS will helpfully convert your TFS branch specification, e.g. `$/ProjectCollection/Feature/my-awesome-feature-branch` into `$C:/Program Files/Git/ProjectCollection/Feature/my-awesome-mfeature-branch`,
at which point `git tfs` will complain that the TFS branch cannot be found. Therefore, you must run the `git tfs branch` or `git tco` (aliased) command as follows to turn off this
automatic "helpful" path translation:

```
$ MSYS_NO_PATHCONV=1 git tco <branch>
```

as was shown in the alias table above.

### Git-Tfs Commit Messages

Git-Tfs is a "bridging" technology, much like `git-svn`, which is a similar bridge between git and SVN. Because TFVC is a centralized source code control system, TFS branches are represented very differently than git branches. Behind the scenes, for each TFS branch, there is an associated remote (but not quite the same as a standard git remote). Using the git-tfs commands, however, this is all transparent to you. What you get by using git-tfs is the ability to use a git-like workflow on your local development machine, while maintaining changes between your local git repository and the centralized TFS repository (and branch).

#### "Good" Git-TFS Commit Messages
A "good" git commit message is one which allows others (including future versions of yourself!) to quickly understand a changeset without requiring one to dive into the diff itself.

> **The seven rules of a great git commit message:**  
>
> 1. Separate subject from body with a blank line
> 2. Limit the subject line to 50 characters
> 3. Capitalize the subject line
> 4. Do not end the subject line with a period
> 5. Use the imperative mode (or present tense) in the subject line, not past tense
> 6. Wrap the body at 72 characters
> 7. Use the body to explain what and why vs. how (i.e. git commit messages are not code comments!)

In addition, if you may want to modify the standard git-tfs `work-item-regex` so that you can more easily link work items to your commits. For example, you could run the following command:

```bash
$ git config --global git-tfs.work-item-regex (?:(?<item_type>(?i:UC|CR|REQ|BUG|TASK))|#)\s*(?<item_id>\d+)
```

This regular expression would allow you to specify work items in your git commit messages with the following format:

```text
[ITEM_TYPE][ITEM_NUMBER]
```

Where ITEM\_TYPE is case-insensitive and one of UC, CR, REQ, BUG, TASK or # followed by zero or more spaces followed by ITEM\_NUMBER, which represents the TFS work item ID. The UC, CR, REQ, BUG, and TASK monikers are optional, but may help clarify your git commit message. If you need to refer to a specific changeset or build id, then use just the `#` symbol, but it might be helpful to indicate that you're referring to a changeset or build.

One main difference between the standard git commit message template and this git-tfs commit message template will be some git-tfs specific items you can place in the commit message so you can link your commits back to TFS work items. The following template shows the overall format of a good git-tfs commit message and is a good guideline to follow for "grok-able" commit messages.

```text
Capitalized, short (50 chars or less) summary

More detailed explanatory text, if necessary.  Wrap it to about 72
characters or so. By default, Git for Windows includes a template
which Vim uses that helps you conform to these best practices.
In some contexts, the first line is treated as the subject of an
email and the rest of the text as the body.  The blank line
separating the summary from the body is critical (unless you omit
the body entirely); tools like rebase can get confused if you run the
two together.

To link this commit message to a TFS work item, you could just say that
this commit is related to CR 12345. Putting this in the commit message
will not resolve the item, but will provide a link from the TFS change-
set created from this commit, to the mentioned CR. You could also have
just as easily said "This commit is related to #12345", but it's not
clear what type of item #12345 is. To resolve the item, you must use
another form of work-item linking shown below. NOTE: you should link
each commit with one or more TFS work items.

Further paragraphs come after blank lines.

- Bullet points are okay, too

- Typically a hyphen or asterisk is used for the bullet, preceded by a
single space, with blank lines in between, but conventions vary here.

- Use a hanging indent, as shown above in the previous bullet

Here are a few additional things you can place in your commit message.
These styles of work item linking should appear at the end of your
commit message. But one blank line must appear below your git commit
message before adding these git-tfs specific helper messages.

# This associates a work item with this commit.
git-tfs-work-item: 12345 associate

# This resolves the work item with this commit.
git-tfs-work-item: 45678 resolve   # NOTE: resolve is the default and so can be omitted

# Likewise, you can specify code rewievers and what-not.
git-tfs-code-reviewer: George Washington
git-tfs-security-reviewer: John Adams
git-tfs-performance-reviewer: Thomas Jefferson

```

### Git-TFS Recipes

#### I've Just Cloned Git-TFS Git Repository and Checked Out a Branch Which Exists in TFS; What Do I Do Next?
Now that you've cloned the git-tfs repository, you should be on the `develop` branch. Once you checkout a different branch which also exists in TFS, you need to "hook it back up" with TFS. Execute the following commands:

```bash
$ git co Dev/Your/Awesome/TFS-Branch   # git checkout Dev/Your/Awesome/TFS-Branch
$ git tbootstrap    # This hooks up the TFS "remote" to your git branch

# Synchronize your local git branch AND the upstream git branch with the
# latest changes from the TFS server.
#
# Note that there is the potential for a race-condition if someone else
# is synchronizing while you are. This is normally a very rare condition,
# but you should be aware of it. If in doubt, please send an IM to your
# team and notify them that you are synchronizing the branch to avoid
# potential problems between team members and the upstream git server.
$ git tsync      # <-- aliased git tfs-sync
```

Now you're ready to create a local git feature branch and begin working with TFS using Git.

> **WARNING**  
> **Never, _ever_** commit directly to a branch from within Git that exists in TFS. While it won't
> permanently "mess up" anything, it results in duplicate changesets being made in TFS. The first
> changeset is the git commit, and the next changeset is the changeset created by git-tfs during
> `rcheckin`.
> 
> Always create topic branches off of a TFS-backed git branch and commit to the topic branches. Then
> use `git tfs-merge-into <git-tfs-branch-name>` (or the alias `git tmerge <git-tfs-branch-name>`)
> to "merge" (in reality, checkin) the commit (changeset) to TFS.

#### I Need to Checkout an Existing TFS Branch That's Not Part of the Git Repository Yet
This recipe is for when you have cloned a Git-Tfs repository and the TFS branch you want to work off of is not yet part of the git repository, e.g. the branch was created in TFS _after_ the git-tfs repository was cloned.

> **NOTE**  
> Depending on how much history the branch has, this could take a while. Usually, though, for a branch that just started life, this takes just a few seconds.

```bash
# To initialize the branch with the same name as on TFS:
# `tco` here is an alias for a function which calls: MSYS_NO_PATHCONV=1 git tfs branch --init <TFS Branch Path>
# This is roughly analogous to a `git checkout <branch>`, hence the `tco` alias (tfs checkout).
git tco ${TeamProjectCollectionName}/Path/To/Branch

# To initialize the branch with a locally different branch name (this is not recommended):
git tco ${TeamProjectCollectionName}/Path/To/Branch my-local-branch-name
```

#### I Need to Implement a New Feature and Merge it into the TFS Branch
This recipe is used to begin implementaing a new feature. This recipe makes use of helper scripts located in the `Scripts` directory where this document lives.
These scripts ensure that your feature commits are properly "merged" into/checked in to TFS.

> **NOTE**  
> This recipe uses the aliases defined above in this document, as well as a helper
> script called `git-tfs-merge-into` to synchronize your local copies of the remote TFS
> branch with TFS and the centralized Git server and your feature branch and "merge"
> (check in) your feature branch into the parent (TFS) branch.

```bash
# Make sure the TFS branch you're creating the feature off of is up to date with the remote.
# Remember, 'master' is the "source branch" for your TFS changesets--we never want to work
# directly on this branch or any other TFS branch which is part of our git-tfs repository.
#
# If you're working on a feature branch, for example, you might issue the command
$ git co <TFS Branch>

# If you get the following error after issuing the command above:
#
#     error: pathspec '<TFS Branch>' did not match any file(s) known to git.
#
# it's because this branch was created AFTER you originally cloned your repository.
# Issue the following command to bring down the branch and its TFS changesets as git commits:
$ git tco $/TeamProjectCollection/Path/To/Branch
$ git co Path/To/Branch
$ git tbootstrap
$ git push -u origin Path/To/Branch

# Create a local feature/topic branch in which to implement your new feature
$ git cob feature/topic_description           # git checkout -b feature/topic_description

# Implment changes; make commits as you go along -- use these commits to help describe how things are changing
# Ensure when you commit that your commits contain lines starting with 'git-tfs-work-item: <WorkItem_ID>' so
# that your commits are associated with a TFS work item. Optionally, within your commit message, you can say
# things like 'Adding change to satisfy #12345. This is related to CR 78901.' and Git-TFS will associate the
# commit (later, TFS changeset) with work items with ID 12345 and 78901.

# Make sure your working index looks good (optional)
$ git st                                      # git status
# This is a good practice to get into to make sure you don't unintentionally add changes
$ git add -p
$ git commit

# Follow the Git-Tfs Commit Message Guidelines when creating your git commit message.

# For longer running feature branches, periodically incorporate changes from the parent TFS branch into the feature branch
$ git co Path/To/Branch                        # git checkout Path/To/Branch
# Optional, but recommended: notify your team that you are synching Path/To/Branch from TFS
$ git tsync
$ git co feature/topic_description
$ git rebase Path/To/Branch

# When the feature work is complete create the final commit
$ git st      # (optional)
$ git add -p  # Stage the feature commits.
$ git commit

# Follow the Git-Tfs Commit Message Guidelines when creating your git commit message.

# Make sure that the feature branch is up to date with the parent TFS branch
$ git co Path/To/Branch                        # git checkout Path/To/Branch
# Optional, but recommended: notify your team that you are synching Path/To/Branch from TFS
$ git tsync
$ git co feature/topic_description
$ git rebase Path/To/Branch

# Do any rebasing you need to do to clean up your history  (OPTIONAL)
# If you're unsure about this step:
#   1. Then don't do it!
#   2. Ask someone to help you....
$ git rebase -i --autosquash HEAD~x  # where x is the number of commits back you need to fixup.

# Run the tests, verify functionality, etc.

$ git push origin feature/topic_description

# Create a pull request in your git server.
# Once code review is complete, DO NOT merge the branch
# into it's parent TFS git branch using git. Follow the steps below.

# Merge the feature/topic branch back into the parent TFS branch using git-tfs-merge-into.
# You should be on your feature/topic branch that you want to merge.
$ git tmerge Path/To/Branch                    # git tfs-merge-into or even git-tfs-merge-into

# You should now be on Path/To/Branch
```
