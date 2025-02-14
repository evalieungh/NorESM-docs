.. _gitbestpractice:

Git recipes and best practices
==============================

Here is collected various recipes and suggestions for using git. This is not
specific to NorESM, and more comprehensive guides can be found in the :ref:`git references <git-references>`.


Obtain a copy of the model (using git)
''''''''''''''''''''''''''''''''''''''

You can obtain the obtain the code from: https://github.com/NorESMhub/NorESM  
::

  git clone https://github.com/NorESMhub/NorESM.git

The last point will create a new directory called ``NorESM`` in the place you
checked out the model. Go to that directory before executing any git-commands.

-  Also do the following on all machines where you use git:

  * **Make sure you have a version of git >= 2.0** (add the line "module load git" to your .bashrc files on the HPC machine e.g. Betzy, Fram)
  * **git config - -global push.default simple** (Will edit your ~/.gitconfig file to a safer way to share your modifications, see http://stackoverflow.com/questions/13148066/warning-push-default-is-unset-its-implicit-value-is-changing-in-git-2-0)

For further details on downloading the NorESM code and managing externals, please see :ref:`download_code`


Verify that you have the correct checkout
'''''''''''''''''''''''''''''''''''''''''

When you have cloned the model, check that you have gotten what you
wanted!

Check that your favourite branch is available using the command 
::

   git branch --all 

(You should see the branch "master" on top with a star next
to it. This is the branch you get by default. The other branches are
listed below with remotes/origin/branchName, but you can not work on
them until you check them out, see below)

To check out (locally) your favourite branch and to start working on it,
write 
::

   git checkout -b myBranchName origin/myBranchName 

(Note that myBranchName must be one of the branches listed by the above 
command)

If you don't user the "-b" option, you will get something which is not
correct. Make sure you are tracking a remote branch. You can write 
::

   git branch -vv 

to see which remote branch you are tracking. The output will
be something like: 
::

   \* myCheckedOutBranchName 1a08184 [origin/myCheckedOutBranchname] LatestCommitMessageOnBranch

Note that once a branch has been checked out using the -b option, you
can switch between any of your checked out branches using the command
::

   git checkout aCheckedOutBranchName

Note that in git, switching to a new branch change the files in your
working directory. Git will warn you if you have any modified files
before switching to a new branch. This is different from how svn works.

For further details on downloading the NorESM code and managing externals, please see :ref:`download_code`


Modify files
''''''''''''

Modify the code (for example a file named myChangedFile.F90) and send
back to your local repository through 
::

  git add myChangedFile.F90 
  git commit -m "aMessage"

The message should link to the issue on github, so if you fix issue
number 100 by this code change, you would probably write something like
::

  git commit -am "Did part of the work to resolve metno/noresm#100"

Verify, using the tool "gitk" that the changes make sense.


Get modifications from github
'''''''''''''''''''''''''''''

Get modification from remote source by
::

  git pull

To be absolutely sure about branch names etc, you can do
::

  git pull remoteName remoteBranchName:myLocalBranchName 

which if your are picking up changes the master-branch would 
translate to 
::

  git pull origin master:master


Send modifications to github
''''''''''''''''''''''''''''

This command assumes that your changes go to the remote branch named
like your branch (which is most of the times the case) 
::

  git push

You can also do (to be completely sure): 
::

  git push remoteName myLocalBranchName:remoteBranchName 
  
which if your are changing the master-branch would translate to 
::

  git push origin master:master 
  
(The above command means push my changes to the remote named "origin" from my
local branch named master to the remote branch named master. If you are
changing another branch than master, you must obviously not write
"master".)


Merge a single file
'''''''''''''''''''

To merge a single file from another branch into current branch
::

  clone, git pull, go to directory where to put file
  git checkout -b newbranch
  git checkout master file
  git commit -m "add file"
  git pull
  git push


.. _feature-branches:

Why use feature branches?
'''''''''''''''''''''''''

It is generally a good practice, especially in fork-based workflows, to start a
code change session by creating a new feature branch, either from an existing
branch in the local repository or from a branch on GitHub. This serves two
purposes. Firstly, by keeping local copies of shared branches clean from any
local changes, it is easy to keep these branches synchronized with the main
repository. Secondly, it is often useful to be able to change commits or the
entire commit history when doing code development (e.g. if a code bug is
discovered late in the process). To roll back committed changes on
``my-features`` branch to an old state with commit hash ``old-hash``, but leave
the changed files intact, do a mixed reset on the ``my-feature`` branch
::

   git log                        # find reference for <old-hash>
   git reset --mixed <old-hash>


Development branch vs. continuous integration tool (CI)
'''''''''''''''''''''''''''''''''''''''''''''''''''''''
When working using the forking workflow and committing code through reviewed pull requests, there will still be times when code changes will break the software build for various reasons. It is therefore common to merge PR's into a **development branch** in the upstream repository, rather than directly to **master**. This adds additional management, because administrator must merge the development branch into master frequently and regularly, unless the build is broken. The gain is that **master** *always should work*.

An alternative to this scheme is to configure the workflow using a **CI/CD tool** that automates this process. I.e. when the pull request is created, the branch will automatically checked out on a dedicated build server and built. The pull request will not be published before the build is successful on the build server. On github, this is possible with **Github Actions** https://help.github.com/en/actions. It requires effort to get this in place for complex projects, but is normally worth it for large projects.

Another huge benefit of using a CI-tool is that it can automatically run test-suites in your project. E.g. a limited test-suite after successful build (part of evaluating that the build was OK), and a larger set test-suite after nightly builds.


Hotfix branches
''''''''''''''''
A **hotfix** branch is created to fix a specific problem or bug. It should normally branch off and merge back to **master**, but may also merge to **development** or **release** branches. The procedures for hotfix branches are the same as feature branches in terms of creation and merging through pull requests. The main difference is if a single bug fix should be introduced in multiple branches.

To introduce a fix in multiple branches, the **hotfix** branch should be initiated at a common ancestor for all the branches, usually the last commit common to all branch histories. This preserves the development history for the fix and avoids the potential problem of propagating code between branches unintentionally. Fortunately, git can help to identify this point using the command **git merge-base**. In the most general case, introducing a fix in multiple branches, one would check out a new hotfix branch
::

  git checkout -b hotfix/x.x.x-yy $(git merge-base --octopus branch1 branch2 ... branchN)

The naming convention for the hotfix branch is "hotfix/<latest-NorESM-version>-<hotfix-number>". The "--octopus" flag is used if the merge-base involves more than two branches. In practical terms one would normally just include the hotfix for **master** and the latest release, e.g. the first **hotfix** branch for noresm2.0.2 would be
::

  git checkout -b hotfix/2.0.2-1 $(git merge-base master noresm2)

After introducing the fix in the code, the hotfix branch should be merged to all relevant branches through normal pull requests.


Tips and Gotcha's when working with Git
'''''''''''''''''''''''''''''''''''''''
Git is a very complex system, and combining it with a complex workflow, it can be overwhelming. Here are some tips to make things easier:
  
  * **Limit number of simultaneous work branches**. The system can technically handle huge number of branches, but mentally it is very difficult to remember what exactly the different branches contain, especially if they are not sync with the master branch. Try not to have more than two feature branches alive at any time.
  * **Make branches short-lived**. Unless you are making huge refactoring changes in the code (which should have been accepted by the team beforehand), you should generally always create feature-branches that are small enough to be finished within a day or two. When you are not able to finish the feature this rapid, create a **work-in-progress (WIP) pull request** so that the team is informed about what you work on and its progress.
  * **Don't underestimate the value of publishing your commits**. Public commits to git is very often the most valuable communication asset to the rest of the team (in some periods, the only way you communicate). To view what others are doing is key to make your own commits consistent and in sync with others and the whole project. This is another important reason why you should avoid working privately on your own branches for prolonged periods. As mentioned above, also unfinished features are worthy a WIP pull request.

