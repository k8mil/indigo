Hi! Today topic will be about [TeamCity](https://www.jetbrains.com/teamcity/) and how to provide continous integration in your iOS project.

## Motivation

I configured a TeamCity many times and for many projects. There are many adventages of using Continous Integration system in your project development process. Also, there is a lot of alternatives to TeamCity like [CircleCI](https://circleci.com/), [TravisCI](https://travis-ci.org/) and many more. But in this post I want to share with you with experience that I gained in [Bright Inventions](https://brightinventions.pl) with TeamCity.

Every project that we starts - we starts from configuring Continous Integration stuff and in our case we use a TeamCity to handle that.

This post will be more like a tutorial that will guide you through all basic and most important steps in iOS project configuration. Also, I assume that you already download, and host you teamcity service.

Hope you will like it!

# Step 1: Create a root project

Firstly, you need to go to page where your TeamCity is hosted. After log-in, go to the Administration Page, click `Projects` tab in `Project-related Settings` section and click `Create project`

![project settings](./teamcity-for-ios-project/create_project_step1.png)

after that you should see configuration screen for Version Control that is used in your project.

![create project version control](./teamcity-for-ios-project/create_project.png)

I prefer a way that, I will configure everything manually, but of course you can go with predefined sections like : `From GitHub`. `From Bitbucket Cloud` etc.
All you need to do in this step is to provide a Name of your project and then tap `Create`

# Step 2: Add VCS root

Of course, in order to build our project we need to provide a sources to build.
Our TeamCity service should be able to fetch changes from repository. If you're using a GitHub, BitBucket or platforms similar to these, you have two ways:

* Give a credentials to account which has access to the repository

or

* Generate a SSH key and use it to authorize TeamCity in GitHub/Bitbucket

In this post I will show you how to configure it with uploading SSH key.

### Generate new SSH key

*If you haven't heard about generating SSH keys, or you don't know what SSH keys really are. Check [this link](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/)*

To generate new SSH keys you can use terminal command:

```
ssh-keygen -t rsa
```

next, provide a name for new key, and optional passphrase, then in directory in which you ran `ssh-keygen -t rsa` command you should see two files. One is a public key with `.pub` extension, and second - private.
The public one will be used in your repository on github/bitbucket. The private key will be used in TeamCity service.

### Use generated key in TeamCity

Go to already created project, settings page and click `VCS SSH Keys` tab.

![ssh section in TeamCity](./teamcity-for-ios-project/ssh_section.png)


Click on `Upload SSH Key`. After that you should see a pop-up window which allows you to upload the previously created SSH Key.
*Please keep in mind to upload a private part of your key, without `pub` extension*. If you choose correctly, click save and you should see screen like this:


![uploaded ssh key](./teamcity-for-ios-project/ssh_uploaded_key.png)

As you can see in `Usage` tab, the key is not used in the configuration yet.
In order to use it - you have to go through the next steps...


### Configure VCS root

Go to already created project, settings page. As you can notice in `SSH Keys` tab - appeared number '1' - it means that we have one SSH key uploaded which is ready to use.

![create project version control](./teamcity-for-ios-project/vcs_root_section.png)

Click on `VCS Roots` tab, and then `Create VC Root`.

In our case, in `Type of VCS` select `Git`

![type of vcs](./teamcity-for-ios-project/type_of_vcs.png)

In `VCS root name` provide a name which will be:

>A unique name to distinguish this VCS root from other roots.

Next, in `Fetch URL` paste a link to your repository. *Please remember to paste here a SSH link type e.g `git@github.com:yournickname/yourrepositoryname.git`*

![fetch url in vcs config](./teamcity-for-ios-project/fetch_url.png)

Next, most important, in `Authentication method` select `Uploaded Key` and choose previously uploaded private ssh key for you repository.

![fetch url in vcs config](./teamcity-for-ios-project/ssh_choose_uploaded_key.png)

Almost done. Now go to the end of the page and click `Test Connection`.

If you see screen like this:

![fetch url in vcs config](./teamcity-for-ios-project/connection_failed.png)

it means that our public part of generated SSH key is not used in repository, and that's why you get `Auth failed` error. So, all you need to do is to add public part of SSH Key in `Access keys` or `Deploy keys` in your repository.

Here you have links for bitbucket and github instructions how to do that:

[BitBucket - Use access keys](https://confluence.atlassian.com/bitbucket/use-access-keys-294486051.html)

[GitHub - Deploy keys](https://developer.github.com/v3/guides/managing-deploy-keys/#deploy-keys)

If you successfully uploaded public part of SSH Key, click `Test Connection` again, and I hope you be able to see `Connection successful` alert. It means that TeamCity has an access to read your repository.

![fetch url in vcs config](./teamcity-for-ios-project/connection_successful.png)


# Step 3: Create build configuration
Ok, our VCS is configured. Now it's time to create build configuration in TeamCity project. Build configuration is a kind of lane which specify what type of build you provide in this lane.
It could be lane for: compile your project and run unit tests or just compile project or compile project then create .ipa files and send it to iTunesConnect or even separte lane for running UI tests.

Go to `General Settings` in you already created project and click `Create build configuration`

![build configuration step 1](./teamcity-for-ios-project/create_build_configuration.png)

In next screen, once again, choose `Manually` option and name you new build configuration. In our case let's name it `[Develop] Build & Test`. The name is meaning full and means that our lane will build iOS project with develop configuration - `Develop` and also, provides an short information what this lane will do - `Build & Test` which means that we compile our project and run unit tests.

![build configuration step 2](./teamcity-for-ios-project/build_configuration_name.png)

Click `Create` and after that you should see:

![build configuration step 2](./teamcity-for-ios-project/attach_vcs_root.png)

Here, select previously created `VCS Root` and click `Attach`.

All done our build confiuration is connected with VCS.

# Step 4: Configure build steps for configuration

Now it's time to define a steps in our build configuration. What the build steps are? It's sequence of instructions which TeamCity will run on our agent machine. Put it simply, it could be something like:

1. Fetch new changes from repo
2. Install dependencies (cocoapods, bundle install and stuff like that)
3. Compile project using script (xcodebuild, fastlane)

*Please remember that build steps depeneds on how you configured you iOS project. In my example I used [Fastlane](https://fastlane.tools), and [Bundler](http://bundler.io/) to manage versions of gems installed in iOS project.*

In order to create build steps go `Build configuration Settings` and tap `Build Steps`

![build configuration step 2](./teamcity-for-ios-project/build_steps_1.png)

Click on `Add build step`, and on the next screen select a `Command line` runner type.
In `Step name` name your build step(in my case it will be `Install Dependencies`). In `Custom script` type a script that will be executed in this build step. Again, in my case it will be

```
bundle install
```

![build configuration step 2](./teamcity-for-ios-project/command_line_build_step_configuration.png)

click `Save` and your first step is ready!

I also added another command line build step called `Build and tests` which will run command:

```
bundle exec fastalne build_and_test
```

*`build_and_test` is the name of the lane in `Fastfile`. If you're not familiar with Fastlane, please have a look on this. It's a great tool [Fastlane](https://fastlane.tools)*


So, all build steps created.

![build steps created](./teamcity-for-ios-project/build_steps_final.png)


# Step 5: Triggers

Have you wonder how TeamCity knows when to fetch new changes from repository and build it? Triggers is an answer.

I prefer to use two types of triggers. One of these is called `VCS Trigger` which means, that TeamCity checks automatically if something has changed in your repository and if yes then TeamCity will start a build configuation which contantais that type of trigger.
`VCS Trigger` is used in configurations like `Compile & Test` for example. Because we want to compile and run tests after every push on repository.

Second one is the `Schedule Trigger`. It is simple trigger which could say : *Run this configuration at every Monday at 7:00AM*

Go to `Triggers` section in Build Configuration main page. Click `Add new trigger` and select `VCS Trigger` and simply click `Save`

![vcs trigger](./teamcity-for-ios-project/vcs_trigger.png)

`VCS Trigger` configured successfully, easy right?.

Next, do the same, `Add new trigger` -> `Schedule Trigger` and chose options that will suit to your requirements. In my case I choose a daily trigger at 04:00 AM and click `Save`.

![time trigger](./teamcity-for-ios-project/time_trigger.png)

All triggers created!

# Step 6: Add Build Features

Build features are cool stuff. For example, while using build features you can create a condition that will check if on agent machine is installed correct ruby version, or you can create condition that will check if there is an avaialble space on you machine. It is super useful if you want to produce `.ipa` files and you know that you need at least 100MB free space. In this post I will show you how to configure two build features one is `XML report processing` and the second one `Ruby enviroment configurator`.

*  `XML report processing`

Go to `Build Features` section in build configuration main page, and click `Add build feature`. Choose `XML report processing`, select `Ant JUnit` and in `Monitoring rules` paste a path for `report.junit` file which is generated by Fastlane after `scan` action.
In `report.junit` you can find out how many tests have been run, how many tests failed, how many tests have been completed successfully.

![xml reporting](./teamcity-for-ios-project/xml_reporting.png)

* `Ruby environment configurator`

This time, select `Ruby environment configurator` in `Add build feature` window. In `gemset` define which ruby version you need to have in my case it was `ruby-2.3.3`.

This feature will check if on agent machine is available this version of ruby and if not it will not start a build configuration. It is optional step, but sometimes in is super useful - especially, if you use multiple ruby versions or someone else could change a global version of ruby on agent machine.

![xml reporting](./teamcity-for-ios-project/ruby_feature.png)

# Step 7: Parameters

There are 3 types of parameters for build configuration or even root project.

* Configuration Parameter

* System properties(system.)

* Environment Variables(env.)

In this post I will focus on `Environment variables`. That type of variables are created after build start and they can be accessed via Command line with `$` prefix. One example of Environment variable could be an XCode path. In order to compile our project we need XCode path which will be used by Fastlane tool. So in `Fastfile` I add line:

```
xcode_select ENV["XCODE_PATH"]
```

which means - select XCode from path that you can find under `ENV["XCODE_PATH"]` variable.

On build configuration main page select `Parameters` and click `Add new parameter`. In `Name` type `env.XCODE_PATH`, TeamCity should automatically change Kind to `Environment variable` and in value provide a path to XCode.app on your agent machine.


![xcode path](./teamcity-for-ios-project/xcode_path.png)

Using parameters you can pass many useful values like, build number, or name of scheme that should be build and many more. I encourage you to check it out :)

# Step 8: Failuire Conditions

Failuire conditions should be used in case which you want to force fail your build. Great example for that is a timeout. Let's imagine that something bad happen on your agent machine and your build is hanging over 2 hours, when normally it tooks a few minutes. Failuire conditions comes with help! You can set up here that if build lasts above `n` minutes then it should fail.

In main page of build configuration go to `Failuire Conditions` and on line `if runs longer than specified limit in minutes` put a value(in minutes). In my case it will be 60 minutes.

![failuire condition](./teamcity-for-ios-project/failuire_condition.png)


# Step 9: Agent machine

As you probably noticed, I often mention about `agent machine`. Agent, **in iOS case**, it is a computer(Macbook, MacMini, etc) with macOS system. Agent is connected via script to your TeamCity page. TeamCity can communicate with agent in order to use it for execute build steps from build configuration. Important thing is that your agent should be turned ON all the time to provide continous integration.

Ok, but how to configure an Agent?

Go to `Agents` tab in TeamCity page. You can find it at the top.

![agent tab](./teamcity-for-ios-project/agents.png)

In new created TeamCity there are no available agents yet. Let's click on `Install Build Agents`.

![failuire condition](./teamcity-for-ios-project/install_build_agent.png)

I prefer a way to install it via `Zip file distribution`. After you click on that your web brower will download the all files that are necessary to run agent.

Great instruction how to configure mac agent you can find at [TeamCity docs here](https://confluence.jetbrains.com/display/TCD10//Setting+up+and+Running+Additional+Build+Agents#SettingupandRunningAdditionalBuildAgents-InstallingviaZIPFile). Here are steps from this documentation:

```
1.Make sure a JDK (JRE) 1.8 (versions 1.6-1.8 are supported, but 1.8 is recommended) is properly installed on the agent computer.

2.On the agent computer, make sure the JRE_HOME or JAVA_HOME environment variables are set (pointing to the installed JRE or JDK directory respectively).

3.In the TeamCity Web UI, navigate to the Agents tab.

4.Click the Install Build Agents link and select Zip file distribution to download the archive.

5.Unzip the downloaded file into the desired directory.

6.Navigate to the <installation path>\conf directory, locate the file called buildAgent.dist.properties and rename it to buildAgent.properties.

7.Edit the buildAgent.properties file to specify the TeamCity server URL and the name of the agent. Please refer to Build Agent Configuration section for details on agent configuration.

8.Under Linux, you may need to give execution permissions to the bin/agent.sh shell script.
```

After these steps you can start the agent via command:

```
 pathToDownloadedUnzippedBuildAgentFiles\bin\agent.sh start
```

The next thing to do - is to go to TeamCity page on `Agents` tab again. Wait a bit, and after some time you should see one agent available under `Unauthorized` tab. Only thing to do is to `Authorize` agent. After this you will see you agent under `Connected` tab.

![agent connected](./teamcity-for-ios-project/agent_connected.png)

# Step 10: Setup Agent requirements for build configuration

The last thing...You need to specify which agent should build your configuration. If  your TeamCity contains a projects for iOS and Android, probably it will have two agents - one for Android, and second for iOS. Of course we don't want to start our **iOS build configuration** on computer for Android project which probably will not have an XCode, or even macOS. So, in order to provide proper agent for you project you need to use `Agent requirements` for build configuration.

In main page of build configuration go to `Agent requirements` tab and then:

* Select `Add new requirement`
* In `Parameter Name` type `teamcity.agent.jvm.os.name`
* In `Condition` select  `equals`
* In `Value` select  `Mac OS X`
* Save

![agent connected](./teamcity-for-ios-project/agent_requirements.png)

The requirement which we already created means that agent os name should be Mac OS X because we configure an project for iOS.

# Step 11: Ready for build - RUN! 🎉

All configured, you're ready to start your build via TeamCity. Go to main TeamCity page and tap `Run` on your freshly created build configuration.

![start build](./teamcity-for-ios-project/run_build.png)

After that, you will be able to see the progress:

![build running](./teamcity-for-ios-project/build_running.png)

If you want to see a progress in current build, just click on `Running` label and go to the `Build log` section. It is very useful if some error ocurred while compiling.

![build running](./teamcity-for-ios-project/build_log.png)

Finally, after all build steps you be able to see:

![build success](./teamcity-for-ios-project/build_success.png)

As you can see, `XML Processing Report` Build Feature provides cool output about unit tests: `Test passed: 24`.

# Step 12: Artifacts!

Last, optional, step. I can imagine that you can find many cases that you want to compile a project, create a `.ipa` file and send it to the client. Artifacts are made for it.
To do this: Go to build configuration settings by click `Edit Settings`:

![build configuration settings](./teamcity-for-ios-project/edit_settings.png)

in `General Settings` under `Artifacts paths` type a path to `.ipa` file which will be generated by your build scripts. I recommend to use Fastlane again. Fastlane action called `gym` will build your project and create `.ipa` file in output directory. More about `gym` you can [read here](https://docs.fastlane.tools/actions/gym/).

![build configuration settings](./teamcity-for-ios-project/artifacts_path.png)

If you do this, after next build you be able to download `.ipa` via `Artifacts` page on TeamCity.

![build configuration settings](./teamcity-for-ios-project/artifacts.png)

# Conclusion

TeamCity is a great platform to provide Continous Integration in your project. In combinantion with Fastlane it saves you a many hours of manual deploying, testing, compiling.

Also, if you will configure Artifacts in build configuration, you can easily send a link to your TeamCity page to your client, and give him instructions how to download the `.ipa` with a few clicks. So, you don't have to worry about sending `.ipa` files every time you created a new one version.

Hope you like the post. Feel free to comment and share :)














