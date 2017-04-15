# Migrate TFS to Git

## Config Workspace

> **Skip this step if the TFS repository mapping exists in the workspace.**

Launch Team Explorer and connect to the TFVC, for example https://baikangwang.visualstudio.com/

The first thing to do is map usernames. TFVC is fairly liberal with what goes into the author field for changesets, but Git wants a human-readable name and email address. You can get this information from the`tf`command-line client, like so:

```
PS> tf history $/myproject -recursive > AUTHORS_TMP
```

This grabs all of the changesets in the history of the project and put it in the AUTHORS\_TMP file that we will process to extract the data of the_User_column \(the 2nd one\). Open the file and find at which characters start and end the column and replace, in the following command-line, the parameters`11-20`of the`cut`command with the ones found:

```
PS
>
 cat AUTHORS_TMP | cut -b 11-20 | tail -n+3 | sort | uniq 
>
 AUTHORS
```

The`cut`command keeps only the characters between 11 and 20 from each line. The`tail`command skips the first two lines, which are field headers and ASCII-art underlines. The result of all of this is piped to`sort`and`uniq`to eliminate duplicates, and saved to a file named`AUTHORS`. The next step is manual; in order for git-tfs to make effective use of this file, each line must be in this format:

```
DOMAIN\username = User Name 
<
email@address.com
>
```

The portion on the left is the “User” field from TFVC, and the portion on the right side of the equals sign is the user name that will be used for Git commits.

Once you have this file, the next thing to do is make a full clone of the TFVC project you’re interested in:

```
PS
>
 git tfs clone --with-branches --authors=AUTHORS https://username.visualstudio.com/DefaultCollection $/project/Trunk project_git
```

Next you’ll want to clean the`git-tfs-id`sections from the bottom of the commit messages. The following command will do that:

```
PS
>
 git filter-branch -f --msg-filter 'sed "s/^git-tfs-id:.*$//g"' '--' --all
```

That uses the`sed`command from the Git-bash environment to replace any line starting with “git-tfs-id:” with emptiness, which Git will then ignore.

Once that’s all done, you’re ready to add a new remote, push all your branches up, and have your team start working from Git.



