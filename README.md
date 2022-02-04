# Manually Created Git Repository Using Plumbing commnds

### Description:

* Generally users/developers make use of a graphical user interface built on top of the Git cli. This makes it easier to work with Git as one does need to not understand what commands is running in the background to achieve functionality such as committing and pushing commits to a remote server. If you brave enough, you could use the Git cli directly and run commands such as "git init", "git add *" and "git commit -m 'Initial Commit'". The Git cli is not the lowest level you can use to work with Git however. The Git cli is commonly known as the "Porcelain" layer as it's actually built on top of another layer known as the "Plumbing" layer which executes low-level commands. This repository was created using the low-level commands.

### Steps to follow to create Git repository and make first commit using Plumbing Commands:

* To create a bare minumum Git repository one has to create the following folders, files and content:
	* .git -> folder
		* heads -> folder
		* refs -> folder
		* HEAD -> echo "ref: refs/heads/master"
			* This file needs to be utf-8 encoded. You can open it in Notepad++ and click on Encoding -> utf-8 to do this.
			* If it's not utf-8 encoded Git will still pick up the the directory as not a Git directory.
	* To create a directory run the following command: "mkdir {folder/directory}".
	* To create a file run the following command: "type nul > {filename}.{extension}".
		* Note that files created via "Command Prompt" is utf-8 encoded and via "Powershell" is utf-16 BE BOM.
* After creating the above folders, files and content, running "git status" in the root directory of your project should show that this directory is now version controlled and display that you on the master branch and currently have no tracked changes.
* Next run "echo git is awesome | git hash-object --stdin -w"
	* hash-object creates a hash. --stdin tells hash-object to create the hash from the standard input (cli). -w tells Git to write the blob to Git Object Database.
* Running "tree /f .git" should show the .git directory and it's contents. You should see that a folder containing a file has been created in objects directory.
	* The new folder actually gets it's name from the first two characters in the generated hash and the new file inside the new folder gets it's name from the rest of the hash.
	*  It does this because consider a fairly big repository, one that has 300,000 objects (blobs, trees, and commits) in its database. To look up a hash inside that list of 300,000 hashes can take a while. Thus, Git simply divides that problem by 256. To look up the hash above, Git would first look for the directory named 54 inside the directory .git\objects, which may have up to 256 directories (00 through FF). Then, it will search that directory, narrowing down the search as it goes.
* To check the type of object, run "git cat-file -t {blob-hash - combine the folder and file name to get the original hash}".
	* The object should be a blob in this instance as it was created from "git is awesome" text.
	* To see the value of the blob, run "git cat-file -p {blob-hash - combine the folder and file name to get the original hash}"
* Running "git status", you will see that the object is not picked up as a changed.
	* Running "git update-index -add --cacheinfo 100644 {blob-hash} my_file.txt" stages the changes.
		* This information is stored now in a file called index in the .git root directory.
		* This allows Git to register the change and says that the hash/blob is contained in the file "my_file.txt"
	* You will notice that the file "my_file.txt" is picked up as both 'Changes to be committed" and deleted. It's picked up as deleted because the file "my_file.txt" does not actually exist.
* Create the "my_file.txt" by running "git cat-file -p {blob-hash} > my_file.txt".
	* It should create the file "my_file.txt" and if "git status" is run, it should not be picked up as deleted anymore. You might need to change the files encoding to utf-8 depending on the encoding that was used when generating hash.
	* You could create the file manually also and add the content "git is awesome" but every character and file encoding, file type all needs to be exact to generate the same hash which git requires to determine changes.
* Now that we have our changes stage, we need to create a tree object which is required by a commit object.
	* Running "git wrtie-tree" creates the tree object based on the information from the index file which contains the information about the staged changes.
	* The tree object contains the details of what blobs/trees have been staged.
	* Running " git cat-file -t {tree-hash}" should show the object is a tree.
* The next step is to create a commit object. You can do that by running "git commit-tree {tree-hash} -m "First Git commit using low-level (plumbing) commands"".
	* The commit object contains a reference to the root directory (root tree object), commit date, commit author.
	* Running " git cat-file -t {commit-hash}" should show the object is a commit.
* Running "git status" still shows the file to be staged, this is because in order for git to know that our changes have been committed, it needs to know about our latest commit.
	* Running "echo {commit-hash} > .git/refs/heads/master" tells our master branch out our latest commit.

### Creating another branch and making changes and committing using Plumbing Commands:

* To create another branch, we need to run "echo {commit-hash} > .git/refs/heads/{branch-name}".
	* If you now run "git status", you will see that not only does the master branch point to the commit but also your new branch.
* To switch to the new branch, we need to run the following: "echo ref: refs/heads/test > .git/HEAD".
	* Running "git status" should show that you now pointing to the new branch.
* To make a change(blob-object), run "echo Testing | git hash-object --stdin -w".
	* This creates a blob-object that has a hash based on "Testing" content.
* For Git to pick up that change run: "git update-index --add --cacheinfo 100644 {blob-hash} test.txt(file that contains content which was hashed from blob)".
* To indicate to Git that the file was only added and not deleted, run "git cat-file -p {blob-hash} > test.txt".
* Create tree-object based on the staged changes by running "git write-tree".
* To create a ommit-object, run "git commit-tree {tree-hash} -m "Testing" -p {previous-commit-hash}".
* To point the branch at the latest commit, run "echo {commit-hash} > .git/refs/heads/test".

### Some Git commands:

* Porcelain commands:
	* git init
	* git add
	* git commit
	
* Plumbling commands:
	* echo ref: refs/heads/master > .git/HEAD
	* echo git is awesome | git hash-object --stdin -w
	* git cat-file -t {hash}
	* git cat-file -p {hash}
	* git update-index -add --cacheinfo 100644 {blob-hash} my_file.txt
	* git wrtie-tree
	* git commit-tree {tree-hash} -m "Commit message"
	

### References:
* https://www.freecodecamp.org/news/git-internals-objects-branches-create-repo/amp/