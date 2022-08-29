---
layout: post
title:  "Migrating Github/Gitlab Projects and Repositories"
categories: GitHub/GitLab Administration
author: Uchechukwu Onyekwuluje
tags: Administration
---
In this post we will look into how to migrate a Repository from GitHub to GitLab. The same process works from GitLab to GitHub

## Clone the source repository
Clone the source repository on your workstation
```
git clone --bare git@github.com:uonyekwuluje/devops-labs.git
```

## Push & update target repository
The target repository should be created and setup
```
# Set one or the order
git@gitlabci-server.ucheonyekwuluje.com:app-code/apptest.git
https://gitlabci-server.ucheonyekwuluje.com/app-code/apptest.git

cd devops-labs.git
git push --mirror git@gitlabci-server.ucheonyekwuluje.com:app-code/apptest.git
OR
git push --mirror https://gitlabci-server.ucheonyekwuluje.com/app-code/apptest.git
<enter username>
<enter password>
```

## Reference Links
* https://blog.knoldus.com/how-to-migrate-a-repository-from-github-to-gitlab/
