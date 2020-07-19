.. -*- coding: utf-8; mode: rst; -*-
.. git Cheat Sheet https://github.com/peterdv/CheatSheetsAndOtherRecipes

.. reStructuredText Markup Specification https://docutils.sourceforge.io/docs/ref/rst/restructuredtext.html
   
.. For the Python documentation, 
   this convention is used which you may follow:
    • # with overline, for parts
    • * with overline, for chapters
    • =, for sections
    • -, for subsections
    • ^, for subsubsections
    • ", for paragraphs


`git` Cheat Sheet
=================

This is a collection of frequently used
`git <https://git-scm.com/>`_
commands and 
Recipes
which I personally have found usefull enough
to document and store.

For detailled discussion, please consult the
`Pro Git book <https://git-scm.com/book/en>`_

.. contents:: 
   :local:
   :depth: 2  

	   
`git` recipes
-------------

Many of the processes follows
`Vincent Driessen's branching model <http://nvie.com/posts/a-successful-git-branching-model/>`_.

It is perfectly possible to use this branching model via
a sequence of pure `git` commands.

To achieve a consistent use of these commands
`git-flow <https://github.com/nvie/gitflow>`_
provides high-level repository operations
for Vincent Driessen's branching model.

Based on two central branches `master` and `develop`,
the overall processes of `git-flow` are:

#. A `develop` branch is created from `master`.
#. Feature branches are created from `develop`.       
#. When a feature is complete
   it is merged into the `develop` branch.
#. A release branch is created from `develop`.
#. When the release branch is done
   it is merged into `develop` and `master`.
#. If an issue in `master` is detected,
   a hotfix branch is created from `master`.
#. Once the hotfix is complete,
   it is merged to both `develop` and `master`.


  

Creating Projects
^^^^^^^^^^^^^^^^^

After creating an initial repository at github.com, containing only a README file, the
process of prepping the repository is:

#. Create a local copy of the remote repository.

#. Configure `git-flow` using the default values
   and create the local `develop` branch.

#. Create the newly created `develop` branch in the remote repository,
   so it is easily avaliable for your collaborators.
   Link the local branch with the remote branch


::
   
   git clone --recursive git@github.com/<username>/<repository-name>.git
   cd <repository-name>
   git flow init -d
   git push --set-upstream origin develop

You might want to check the result using
`git branch -a -vv`,
which should produce something like::

  $ git branch -a -vv
  * develop                b3b6205 [origin/develop] Merge tag '0.1.0' into develop
    master                 4f8cc3f [origin/master] Merge branch 'release/0.1.0'
    remotes/origin/HEAD    -> origin/master
    remotes/origin/develop b3b6205 Merge tag '0.1.0' into develop
    remotes/origin/master  4f8cc3f Merge branch 'release/0.1.0'


Creating a new feature
^^^^^^^^^^^^^^^^^^^^^^

* Develop new features for upcoming releases.

* The feature branches typically exist in developers repos only.

Each new feature should reside in its own branch,
which can be pushed to the central repository
for backup/collaboration.
But, instead of branching off of `master`,
feature branches use `develop` as their parent branch.
When a feature is complete, it gets merged back into `develop`.
Features should never interact directly with `master`.

Feature branches are generally created off the latest `develop` branch.


Start to implement a new feature following the process:

#. List any local feature branches
   (to ensure that you have not started on this once before).

#. Create a local feature branch based on `develop` and swith to it.

#. Then,
      
   * Do the work and commit your changes to the feature branch.
   * Test the feature.
   * Fix bugs and commit your changes to the feature branch.
   

#. Finish the feature:

   * Merge the feature branch into the local `develop` branch,
     
   * remove the local feature branch
	
   * and switch back to the `develop` branch.

#. Publish the `develop` branch. 

#. Publish your tags 


::
   
   git flow feature
   git flow feature start <your feature> [<base>]

   # Then, do the work 
   
   git flow feature finish <name>
   git push origin develop
   git push origin --tags
   
For feature branches, the `<base>` argument must be a commit on `develop`.

You could also decide to push a feature branch to the remote repository,
to allow collaboration with others via the remote repository

::
   
   git flow feature publish <your feature>

   
Make a release
^^^^^^^^^^^^^^

* The preparation of a new production release.

* Allow for minor bug fixes and preparing meta-data for a release.

* The release branche typically exist in
  developers local repositories *and* in a shared (central) repository.

Once develop has acquired enough features for a release
(or a predetermined release date is approaching),
you fork a release branch off of develop.
Creating this branch starts the next release cycle,
so no new features can be added after this point
— only bug fixes, documentation generation,
and other release-oriented tasks should go in this branch.
Once it's ready to ship,
the release branch gets merged into `master`
and tagged with a version number.
In addition, it should be merged back into `develop`,
which may have progressed since the release was initiated.

Using a dedicated branch to prepare releases
makes it possible for one team to polish the current release
while another team continues working on features for the next release.
It also creates well-defined phases of development
(e.g., it's easy to say, “This week we're preparing for version 4.0,”
and to actually see it in the structure of the repository).

To make a release, use the process:

#. Creates a release branch created from the `develop` branch.

#. It's wise to publish the release branch after creating it
   to allow release commits by other developers.
   You can track a remote release with the
   `git flow release track <remote release>` command.`

#. Then,
      
   * Do the work and commit your changes to the release branch.
     
   * Test the release.

   * Fix bugs and commit your changes to the release branch.
   

#. Finish up a release.
   This is one of the big steps in git branching.
   It performs several actions:

   * Merges the `release branch back into `master`.
     
   * Tags the release with its name.

   * Back-merges the release into `develop`.

   * Removes the release branch.

#. Publish the `master` branch. 

#. Publish the `develop` branch. 

#. Publish your tags 

::
   
   git flow release start <your release>  [<base>]
   git flow release publish <your release>

   # Then, do the work

   
   git flow release finish <your release>
   git push origin master
   git push origin develop
   # or if you did not track develop: git push --set-upstream origin develop
   git push origin --tags
   
   
For release branches, the `<base>` argument must be a commit on `develop`.

   
Make a hotfix
^^^^^^^^^^^^^

* Hotfixes arise from the necessity to act immediately
  upon an undesired state of a live production version.

* May be branched off from the corresponding tag on the master branch
  that marks the production version.

* The hotfix branches typically exist in developers repos only.

Maintenance or “hotfix” branches are used to quickly patch
production releases.
Hotfix branches are a lot like release branches and feature branches
except they're based on `master` instead of `develop`.
This is the only branch that should fork directly off of `master`.
As soon as the fix is complete,
it should be merged into both `master` and `develop`
(or the current release branch),
and `master` should be tagged with an updated version number.

To make a hotfix, use the process:

#. Creates a hotfix branch from the `master` branch.
   The `version` argument marks the new hotfix release name.
   Optionally you can specify a `basename` to start from.

#. Then,
      
   * Do the work and commit your changes to the hotfix branch.
	
   * Test the hotfix.

   * Fix bugs and commit your changes to the hotfix branch.
	

#. Finish up a hotfix.

   * The hotfix branch is merged back into develop and master.

   * The `master` merge is tagged with the hotfix `version`.

#. Publish `master` including the merged hotfix branch.
      
#. Publish `develop` including the merged hotfix branch.

#. Publish your tags 
      
::
   
   git flow hotfix start <version> [<basename>] 

   # Then, do the work

   git flow hotfix finish <version>
   git push origin master
   git push origin develop
   git push origin --tags


`git` Commands
--------------


`git` Getting and Creating Projects
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table:: Getting and Creating Projects
   :widths: 60 40
   :header-rows: 1

   * - Command
     - Description
   * - `git init`
     - Initialize a local Git repository.
   * - `git clone git@github.com/<username>/<repository-name>.git`
     - Create a local copy of an upstream remote repository.
   * - `git checkout -b develop origin/develop`
     - Create a local copy of the branch `develop` from
       the upstream remote repository.
       Assuming You are in a cloned repository.
   * - `git branch -vv`
     - check tracking branches.


`git` Basic work
^^^^^^^^^^^^^^^^

.. list-table:: Basic work
   :widths: 60 40
   :header-rows: 1

   * - Command
     - Description
   * - `git status`
     - Check status.
   * - `git add <file-name>`
     - Add a file to the staging area.
   * - `git add <directory-name>`
     - Add a directory and all of its contents to the staging area.
   * - `git add -A`
     - Add all new and changed files to the staging area.
   * - `git commit -m "<commit message>"`
     - Commit changes in the staging area to the local branch.
   * - `git rm -r <file-name>`
     - Stage a file (or folder) for removal.

git Branching and Merging
^^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table:: Branching and Merging
   :widths: 60 40
   :header-rows: 1

   * - Command
     - Description
   * - `git branch`
     - List branches (the asterisk denotes the current branch).
   * - `git branch -a`
     - List all branches (local and remote).
   * - `git branch <branch name>`
     - Create a new branch.
   * - `git branch -d <branch name>`
     - Delete a branch.
   * - `git push origin --delete <branch name>`
     - Delete a remote branch.
   * - `git checkout -b <branch name>`
     - Create a new branch
       and switch to it.
   * - `git checkout -b <branch name> origin/<branch name>`
     - Clone a remote branch and switch to it.
   * - `git branch -m <old branch name> <new branch name>`
     - Rename a local branch.
   * - `git checkout <branch name>`
     - Switch to a local branch.
   * - `git checkout -`
     - Switch to the branch last checked out.
   * - `git checkout -- <file-name>`
     - Discard changes to a file.
   * - `git merge <branch name>`
     - Merge a branch into the active branch.
   * - `git merge <source branch> <target branch>`
     - Merge a branch into a target branch.
   * - `git stash`
     - Stash changes in a dirty working directory,
       and revert all uncommitted changes
       in the working directory.
   * - `git stash list`
     - See which stashes you have stored.
   * - `git stash apply [<stash-name>]`
     - Re-modifies the files you reverted when you saved the stash.

       You can save a stash on one branch,
       switch to another branch later, and try to reapply the changes.
       You can also have modified and uncommitted files
       in your working directory when you apply a stash — Git gives you
       merge conflicts if anything no longer applies cleanly.
   * - `git stash clear`
     -  Remove all stashed entries.

       
       
`git` Inspecting and Comparing Projects
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table:: Inspection
   :widths: 60 40
   :header-rows: 1

   * - Command
     - Description
   * - `git log`
     - View changes.
   * - `git log--summary`
     - View changes (detailed).
   * - `git log--oneline`
     - View changes (one line summary).
   * - `git diff`
     - Preview changes before commiting.
   * - `git diff <source branch> <target branch>`
     - Preview changes before merging.
       

       
`git` Publishing and Updating Projects
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


.. list-table:: Publishing, Sharing and Updating Projects
   :widths: 60 40
   :header-rows: 1

   * - Command
     - Description
   * - `git status -uno`
     - Tell you whether the branch you are tracking is ahead, behind or has diverged.
       If it says nothing, the local and remote are the same.
   * - `git push origin <branch name>`
     - Push a local branch `<branch name>` to the remote repository.
   * - `git push origin --tags`
     - Push your tags to the remote repository.
   * - `git push --set-upstream origin <branch name>`
     - As you push local branch `<branch name>` with
       `--set-upstream` option,
       that local branch is linked with the remote branch automatically.
       The `--set-upstream` flag is used to set `origin`
       as the upstream remote in your git config.
       As you push a branch successfully or update it,
       it adds an upstream reference.
       Usefull if you crated the `develop` branch locally,
       and want to include it in the upstream remote repository.
       The `--set-upstream` and the `-u` flags
       should be equivalent.
   * - `git push`
     - Push changes to remote repository (remembered branch).
   * - `git push origin --delete <branch name>`
     - Delete the remote branch `<branch name>`.
   * - `git pull origin <branch name>`
     - Pull changes from remote branch `<branch name>`.
   * - `git pull`
     - Update local repository to the newest commit
       in the remote repository.
   * - `git remote add origin ssh://git@github.com/<username>/<repository-name>.git`
     - Add a remote repository.
   * - `git remote set-url origin ssh://git@github.com/<username>/<repository-name>.git`
     - Set a repository's origin branch to use the SSH protocol.
       
       
Configuration of `git`
^^^^^^^^^^^^^^^^^^^^^^


.. list-table:: Getting and Creating Projects
   :widths: 60 40
   :header-rows: 1

   * - Command
     - Description
   * - `git config --global user.name "My Name"`
     - Set your display name used by `git`.
       
       It is immutably baked into the commits you create.
       You need to do this only once if you pass the `--global` option,
       because then `git` will always use that information for anything
       your current operating system user do on that system.
   * - `git config --global user.email "my_email@example.com"`
     - Set your email address used by `git`.
       
       It is immutably baked into the commits you create.
       You need to do this only once if you pass the `--global` option,
       because then `git` will always use that information for anything
       your current operating system user do on that system.
   * - `git config --global branch.autosetuprebase always`
     - Use rebase instead of merge.
       
       Change all `git pull` commands to use `git rebase`
       instead of `git merge`.
       Rebasing is prefered over merging by many,
       it prevents unnecessary merge commits ensuring a linear history.
   * - `git config --global core.editor "vi"`
     - Set the editor to `vi` only for Git.
   * - `git config -l`
     - List all configurations for Git.

       

`git` Clarifications
--------------------

The difference between `origin/master`, `origin` and `master`
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


There are actually three things here:
`origin` and `master` are two separate things,
and `origin/master` is one thing.

We have Two branches:

* `master` is a local branch.
      
* `origin/master` A local representation of (or a pointer to)
  the remote branch.
  It is an entity
  (since it is not a physical branch)
  representing the state of the `master` branch on the remote `origin`.
  Think of it as a remote branch (like a local copy
  of the branch named "master" on the remote named "origin").

And one remote:

* `origin` is a remote.




Example: pull in two steps
""""""""""""""""""""""""""

Since `origin/master` is a branch, you can merge it.
Here's a pull in two steps:

Step one, fetch `master` from the remote `origin`.
The `master` branch on `origin` will be fetched
and the resulting local copy will be named `origin/master`.

::
   
   git fetch origin master

Step two, merge `origin/master` into the local branch `master`.

::
   
   git merge origin/master

Having completed the pull in two steps,
you can for example push your new local changes in `master`
back to the remote `origin`:

::
   
   git push origin master

Likewise you can push your local changes in the local `develop` branch
back to the remote `develop` branch on `origin`:

::
   
   git push origin develop

Usually after doing a `git fetch origin` to bring all the changes
from the server,
you would do a `git rebase origin/master`,
to rebase your changes and move the branch to the latest index.
Here, `origin/master` is referring to the remote branch,
because you are basically telling GIT
to rebase the `origin/master` branch onto the current branch.

   
More examples
"""""""""""""

You can fetch multiple branches by name...

::
   
   git fetch origin master stable oldstable

You can merge multiple branches...

::
   
   git merge origin/master hotfix-2275 hotfix-2276 hotfix-2290


.. EOF
