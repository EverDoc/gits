# Migrate TFS to Git

## Install git-tfs

[git-tfs](https://github.com/git-tfs/git-tfs) is a two-way bridge between TFS (Team Foundation Server) and git, similar to git-svn. It fetches TFS commits into a git repository, and lets you push your updates back to TFS.

Download a binary, find it on the [release page](https://github.com/git-tfs/git-tfs/releases). Then have git-tfs.exe in path (and git, too),

> NOTE: You need .NET 4 and either the 2010, 2012 or 2013 version of Team Explorer installed (or Visual Studio).

## Config Workspace

> **Skip this step if the TFS repository mapping exists in the workspace.**

Launch Team Explorer and connect to the TFVC, for example <https://baikangwang.visualstudio.com/>, and map the repository, `$/Learn` <=> C:\Learn

## Authorization Mapping

The first thing to do is map usernames. TFVC is fairly liberal with what goes into the author field for changesets, but Git wants a human-readable name and email address. You can get this information from the `tf` command-line client, so like :

```powershell
PS> cd C:\Learn
PS> tf history $/Learn -recursive > AUTHORS_TMP
```

> NOTE: Run the command `tf` in the Visual Studio native command-line if the PS doesn't recognize it.

This grabs all of the changesets in the history of the project and put it in the AUTHORS\_TMP file that we will process to extract the data of the_User_column \(the 2nd one\). Open the file and find at which characters start and end the column and replace, in the following command-line, the parameters `11-22` of the `cut` command with the ones found:

```bash
cd C:\Learn
cat AUTHORS_TMP | cut -b 11-22 | tail -n+3 | sort | uniq > AUTHORS
```

> NOTE: The term `cut` is Unix command which wouldn't work within pure powershell, if get error run the command within Git-Bash please

The`cut`command keeps only the characters between 11 and 20 from each line. The`tail`command skips the first two lines, which are field headers and ASCII-art underlines. The result of all of this is piped to`sort`and`uniq`to eliminate duplicates, and saved to a file named`AUTHORS`. The next step is manual; in order for git-tfs to make effective use of this file, each line must be in this format:

`baikangwang = baikangwang <baikangwang@hotmail.com>`

The portion on the left is the “User” field from TFVC, and the portion on the right side of the equals sign is the user name that will be used for Git commits.
Once you have this file, the next thing to do is make a full clone of the TFVC project you’re interested in:

```powershell
PS> git tfs clone --branches=all --authors=AUTHORS https://baikangwang.visualstudio.com/DefaultCollection $/Learn/Trunk Learn_git`
```

> * If get error, `TFS repository can not be root and must start with "$/"`, which is a bash bug in version \>2.5, run the command in the PS or cmd
> * Change `$/Learn/Trunk` to `$/Learn`, if the repository isn't in branches structure

Next you’ll want to clean the `git-tfs-id` sections from the bottom of the commit messages. The following command will do that:

```bash
git filter-branch -f --msg-filter 'sed "s/^git-tfs-id:.*$//g"' '--' --all
```

That uses the`sed`command from the Git-bash environment to replace any line starting with `git-tfs-id:` with emptiness, which Git will then ignore.
Once that’s all done, you’re ready to add a new remote, push all your branches up, and have your team start working from Git.

## Add a remote toward git central repository

Add a remote in your local repository toward an empty git (bare) central repository :

```bash
git remote add origin https://github.com/user/project.git
```

## Push all the source history

Push all the branches on your remote repository:

```bash
git push --all origin
```

## Migration is done!

## References

* git-tfs: <https://github.com/git-tfs/git-tfs>
* Migrate toward external git repository: <https://github.com/git-tfs/git-tfs/blob/master/doc/usecases/migrate_tfs_to_git.md>
* Migrate from Tfs to Git (ProGit v2 Book): <https://git-scm.com/book/en/v2/Git-and-Other-Systems-Migrating-to-Git#TFS>