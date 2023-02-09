# 19 Backups: Advanced Git and Borg
### Merging Git Branches
Often it is useful to have at least two branches for each project: one that is the *stable* branch (often ```master```) which has the currently correct and useable versions of your files and another that is the *unstable* branch (for example ```working```) where you have the files that you are currently at work on. When the working files are completed and ready for the stable branch you merge them into that branch. If you are the only person working on the project and you always use the ```working``` branch for ongoing work and commits then there should be no changes or commits in the ```master``` directory since the last merge and therefore merging ```working``` into ```master``` is easy, it is simply replaying all the changes in ```working``` in the branch ```master```. As mentioned earlier this is called a *fast-forward* in git. We now try making some changes in our new branch of the ```myproject``` repository and merging them back into ```master```. Check out the ```working``` branch and use ed to create a new file
```
git checkout working
ed
a
a fourth file
.
w test4.txt
q
git add test4.txt
git commit -a -m "a fourth file was added"
```
Now try switching to the ```master``` branch and looking at the files
```
git checkout master
ls
```
There should be a file in the master branch called ```test3.txt``` but the file ```test4.txt``` is not present, it is currently only present in the ```working``` branch. We now try merging the ```working``` branch back into the ```master``` branch
```
git merge working
```
This will produce the output
```
Merge made by the 'recursive' strategy.
 test4.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 test4.txt
```
Now try ```ls``` and you will see that ```test4.txt``` is part of the ```master``` branch. Also try ```git log```. You should see that the merge is listed in the log as well as the commit that was done in branch ```working```. One strategy is to never make direct changes to ```master``` and rely on merges to bring your changes into the ``master``` repository. If you try to merge a pair of branches that both include changes to the same files you will have to manually resolve any conflicts that occur in the modified files. We will not go into merge conflict resolution in this course. By following the strategy above you can avoid conflicts entirely. 

### Remote Repositories
This tutorial is not quite over. Thus far, everything we have done with our git repositories has been on our local machine. You now have some great skills for file management. You know how to create a repository for your files, add files to the repository, commit your file changes, create one or more working (unstable) branches, merge the unstable branches back into the stable branch, and recover older versions of your work from previous commits. But what if your hard drive fails? You will lose your local git repository and all your work! You do not yet have a backup. It is quite possible to just create an archive of your git directory every evening (using tar for example) and then copy the archive to a remote machine (using sftp for example). That works strategy works, but you will need an account on a machine that allows you to store your data and copying a large tarred git directory every day may be slow (even if the tar archive is compressed). A more efficient strategy would be to use a tool like rsync that is specifically designed to allow incremental backups by synchronizing files between machines. But wouldn't it be better to make incremental backups of just the bits of files that have changed and do that every time you commit your git files so you are sure to have a backup of your current commits always? Yes it would, and git can help you do that (and it can be done for free!) -- the solution is to create a remote repository. 

Recall that when we cloned ```myproject``` the ```origin``` variable of the clone was the parent repository of the clone. The same thing happens if you create a local repository by cloning a repository on a remote machine. The ```origin``` is now the remote machine. That is the simplest way to create a new repository with a remote origin to which changes can be pushed: simply create an empty repository on a remote machine and clone it on your local machine. How do you create a repository on a remote machine for cloning? One easy way is to create a free account on github or bitbucket. Students can apply for a free [Github Student Developer Pack](https://education.github.com/pack) that allows an unlimited number of public and private repositories. A free GitHub account normally only allows public repositories. There are also free accounts available on [bitbucket](https://bitbucket.org/product/) that allow unlimited private repositories for sharing with up to 5 team members. 

### Creating a Remote Repository
To create your git repository with a remote repository at origin first create an account at a site such as GitHub. Log into your account on a web browser and follow the instructions to create a new empty repository. When the repository has been created you will see a link with a web address that you will copy for cloning. Create a folder for your git repositories then ```cd``` to the new folder and type ```git init``` to initialize the folder as a repository. Then enter the command ```git clone <address>``` where ```<address>``` is the repository web address that you copied. This will create a local clone of the repository. Put some files in the repository then commit and push them. To push files to a repository you must have permissions so you will be prompted for a password for your online git account when you push. If it is a private repository you will also be prompted for a password when you clone or pull the repository. Public repositories can be cloned and changes pulled without a password. If you create a new branch for your project called ```working```, check out the branch and commit some changes to it, you need to use the following commands to create the branch on the remote and push the changes
```
git push --set-upstream origin working
```
The lectures for EVE 198 are on a public repository on github. To clone the repository to your local machine use the command
```
git clone https://github.com/brannala/scb.git
```
I will periodically add files to the repository during the course so whenever you want to work with files in the directory you should first get the latest commits by using the command
```
git pull
```
If you have made changes to files in the directory use stash to store your changes before pulling. Alternatively, you could simply delete the previously cloned directory and clone a new copy.

### Exercises
Create a local git repository on your machine that has a remote ```origin```, copy some of your important files to the repository and commit and push the changes. Then create an unstable working branch of your repository, make some edits, and then commit and push the changes.
