Hi! Today topic will be about [TeamCity](https://www.jetbrains.com/teamcity/) and how to provide continous integration in your iOS project.

## Motivation

I configured a TeamCity many times and for many projects. There are many adventages of using Continous Integration system in your project development process. Also, there is a lot of alternatives to TeamCity like [CircleCI](https://circleci.com/), [TravisCI](https://travis-ci.org/) and many more. But in this post I want to share with you with experience that I gained in Bright Inventions.

Every project that we starts - we starts from configuring Continous Integration stuff and in our case we use a TeamCity to handle that.

This post will be more like a tutorial that will guide you through all basic and most important steps due to configure iOS project. Also, I assume that you already download, and host you teamcity service.

Hope you will like it!

![start image](./teamcity-for-ios-project/start.jpg)

## Step 1: Create a root project

Firstly, you need to go to page where your TeamCity is hosted. After log-in, go to the Administration Page, click `Projects` tab in `Project-related Settings` section and click `Create project`

![project settings](./teamcity-for-ios-project/create_project_step1.png)

after that you should see configuration screen for Version Control that is used in your project.

![create project version control](./teamcity-for-ios-project/create_project.png)

I prefer a way that, I will configure everything manually, but of course you can go with predefined sections like : `From GitHub`. `From Bitbucket Cloud` etc.
All you need to do in this step is to provide a Name of your project and then tap `Create`

## Step 2: Add VCS root

Of course, in order to build our project we need to provide a sources to build.
Our TeamCity service should be able to fetch changes from repository. If you using a GitHub or BitBucket or platforms similar to these you have two ways:

* Give a credentials to account which have access to the repository

or

* Generate a SSH key and use it to authorize TeamCity in GitHub/Bitbucket

I this post I will show you how to configure it with upload SSH key.

### Generate new SSH key

To generate new SSH keys you can use terminal command:

```
ssh-keygen -t rsa
```



So, go to already created project, settings and click `VCS Roots` tab.

![create project version control](./teamcity-for-ios-project/vcs_root_section.png)

