# gs-rest-service

## What is this repo?
This repo is a sample project for you to try out the Predix CI/CD platform. You can create a branch or modify a file to trigger a new build in the pipeline.

## How to create a branch from UI
To create a branch, click the down arrow on the Branch button. Type in the new branch name and hit enter.

## How to create a branch and push change from command line
1. git checkout -b <new-branch-name>
2. make one change to <file-name>
3. git add <file-name>
4. git commit -m "modify file in new branch"
5. git push origin <new-branch-name>

## How to delete a branch from UI
Click the branches tab. Click the 'trash bin' icon to the right of the branch you want to delete.

## How to delete a branch locally and then remotely from command line
1. git branch -D <branch-name>
2. git push origin :<branch-name>

## Who will use this repo?
This repo is available to all Predix CI/CD (Beta) customers. You can register to be a demo user at http://info.ci.build.ge.com/services/predix-cicd-beta/.

## Any question or feedback?
To submit your question or feedback, go to http://info.ci.build.ge.com/ and click 'Support | Predix CI/CD (Beta) | Ask a Question'.
