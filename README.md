# gs-rest-service

## What is this repo?
This repo is a sample project for you to try out the Predix CI/CD platform. You can create a branch or modify a file to trigger a new build in the pipeline.

## How to obtain the Demo Account Token
http://info.ci.build.ge.com/services/predix-cicd-beta/

## How to create a branch and push change from command line using Demo Account Token
1. git clone https://[demo-token]:x-oauth-basic@github.build.ge.com/predix-cicd-demo/gs-rest-service.git 
2. git checkout -b [new-branch-name]
3. make a change to any file [file-name]
4. git add [file-name]
5. git commit -m "modify file in new branch"
6. git push --set-upstream origin [new-branch-name]

## How to delete a branch locally and then remotely from command line
git branch -d [branch-name]
git push origin :[branch-name]

## Who will use this repo?
This repo is available to all Predix CI/CD (Beta) customers. 

## Any question or feedback?
To submit your question or feedback, go to http://info.ci.build.ge.com/ and click 'Support | Predix CI/CD (Beta) | Ask a Question'.
