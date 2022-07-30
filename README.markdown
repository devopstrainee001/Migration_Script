## SVN to GIT Migration Shell Script 
=======================================

This Script helps us to automate the process of migration of SVN repository to GIT repository with history and branches. Its will automatically converts the SVN branches and tags to the GIT Branches and tags and push them to the Azure Repository

Explanation of Script
--------

Provide the name for your project and Email ID :


    PROJECT_NAME="myproject"
    EMAIL="abc@gmail.com" 


Add SVN repository URL which needs to be migrated:


    BASE_SVN="SVN-Repo-URL"


Provide the trunk, branches and tags directory name which is inside your SVN repository:


    BRANCHES="branches"
    TAGS="tags"
    TRUNK="trunk"


Add Azure repository URL where we will push our migrated code. 

   
    AZURE_URL="Azure-Repo-URL"


This command will create the authors file from the SVN commits. We need this file for git migration. It’s a big command, make sure you put all these in a single command:

  
    svn log -q $BASE_SVN | awk -F '|' '/^r/ {sub("^ ", "", $2); sub(" $", "", $2); print $2" = "$2" <"$2"@"$EMAIL">"}' | sort -u >> $AUTHORS



Clone the source SVN repo to a local git repo with all branches, tags, trunk and authors file:


    git svn clone --authors-file=$AUTHORS --trunk=$TRUNK --branches=$BRANCHES --tags=$TAGS $BASE_SVN $TMP


Getting the firts revision (make sure to put all these in single command):    


    FIRST_REVISION=$( svn log -r 1:HEAD --limit 1 $BASE_SVN | awk -F '|' '/^r/ {sub("^ ", "", $1); sub(" $", "", $1); print $1}' )


Add Azure Repository URL where we will store our code:


    git remote add origin $AZURE_URL


Coverts all SVN branches to the GIT branches: 

   
    for BRANCH in $(svn ls $SVN_BRANCHES); do
        echo git branch ${BRANCH%/} remotes/origin/${BRANCH%/}
        git branch ${BRANCH%/} remotes/origin/${BRANCH%/}
    done
>NOTE: If your svn branches are not converted into git branches, make sure your path(/remote/origin/branches) is correct. 

Coverts all SVN tags to GIT tags:

 
    git for-each-ref --format="%(refname:short) %(objectname)" refs/remotes/origin/tags | grep -v "@" | cut -d / -f 3- |
    while read ref
    do
      echo git tag -a $ref -m 'import tag from svn'
      git tag -a $ref -m 'import tag from svn'
    done

    git for-each-ref --format="%(refname:short)" refs/remotes/origin/tags | cut -d / -f 1- |
    while read ref
    do
      git branch -rd $ref
    done
>NOTE: Make sure your path (refs/remotes/origin/tags) is correct.

Push all your code and tags to your Azure repository

     git push origin --all --force
     git push origin --tags
> NOTE: We need to make sure that our workstation is alreday configured with the Azure repository.

### Execute below command to run your shell script:

    
     $ ./migration.sh


#

***NOTE*** : This Script will not work if SVN username and password Authentication is enabled or SVN client is not configured with the SVN server.

#
