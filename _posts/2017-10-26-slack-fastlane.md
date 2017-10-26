---
layout: post
title: 'Slack + Fastlane = ❤️. Talk about one of puzzles of Continous Integration.'
author: kwysocki
date: 2017-10-30 17:40
blog: true
start: false
hidden : true
tag:
- swift
- ios
- fastlane
- slack
- continuous integration
---

![](/assets/posts/slack-fastlane/puzzle.jpg)

Professional development process consist of many puzzles. Some of these puzzles can be testing, choosing good architecture, clean code, continuous integration and much more. 
In this post, I want to focus on one of these puzzles - Continous Integration(CI). An integral part of CI on iOS Development process is a great tool called [Fastlane](https://fastlane.tools/).
Fastlane is a powerful tool which handles a number of tasks like dealing with code signing, creating .ipa files, generating screenshots to AppStore and much more. One of the cool feature of Fastlane is a Slack integration - and that is what I wanted to write about.


# 💪 Motivation 💪

In Bright Inventions, I'm working on a few projects. Every project we start, we start with the few basic steps: create a new repository, basic application setup and most important... Continous Integration. In iOS applications, it starts from installing the Fastlane, creating some lanes in Fastfile, then pushing it to our new repository. Next step is to configure a new TeamCity(a tool that we are using for CI) with new agent machine for the project. And after that... our CI build system is able to trigger a build for our clients, or just build the application and run our unit/UI tests to check if everything working fine.

But what if something went wrong...

![](/assets/posts/slack-fastlane/error.jpg){: .center-image}

Let's say that we were doing some code refactoring, we commit the changes and push it into our repository. Next, our build system discovers that are available new changes on repository - so it starts to build them and run unit tests. And here some tests failed.

![](/assets/posts/slack-fastlane/test-failed.png){: .center-image}

Of course, I don't have open web browsers with our TeamCity website in every time when I pushed something to the repository. I want to be informed if something goes wrong like, failed unit tests or timeout or compilation error happen. And here is the key-word INFORMED -  How our build agent can inform us about error occurred?

### Emails

![](/assets/posts/slack-fastlane/email.jpeg){: .center-image}

We use email client which is built-in into TeamCity service. Every build lane has a rule which says 'send email to all developers when something goes wrong and build fails'. This solution working fine and it's commonly used in many projects and companies. But personally, I'm not 100% satisfied with that solution. If you are work in several projects, you get more and more emails from clients, Jira, team etc. And let's add to that new emails from the build system. Of course, I can create filters and group all this stuff(which I did), but even then it's too much for me. Besides, there is a new thing - if some builds failed - in most cases, it is important to **fix it quick**. So I prefer another way to be notified if something bad happened.


# ❤️ Fastlane + Slack ❤️

## 1. Create your Fastfile in right way

Let's consider a simple example. One lane which compiles the project and runs the unit tests:


```ruby
platform :ios do
desc "Runs all the tests"
lane :test do
    begin
        test_lane()
    rescue => exception
        on_error(exception)
    end
end
end
```

As you can see the body of `:test` lane consist of `begin-rescue-end` structure. It is ruby specific construction and after `begin` you put some code that may fail. After the `rescue => exception` line you put the code that should be executed if something goes wrong. In our case, it will be `on_error(exception)` function. So the Fastfile should look like this:

```ruby
platform :ios do
    desc "Runs all the tests"
    lane :test do
        begin
            test_lane()
        rescue => exception
            on_error(exception)
        end
    end
end

### Methods 

def test_lane
    cocoapods
    clear_derived_data
    scan(scheme: "YourProjectSchemeName", configuration: "Debug")
end

def on_error(exception)
    slack(
        message: "Some thing goes wrong",
        success: false,
        slack_url: "https://your slack incoming webhook url",
        attachment_properties: {
            fields: [
                {
                    title: "Build number",
                    value: ENV["BUILD_NUMBER"],
                },
                {
                    title: "Error message",
                    value: exception.to_s,
                    short: false
                }
            ]
        }
    ) 
end
```

## 2. Generate slack URL

 As you probably noticed `slack` method takes a `slack_url` parameter. But how can I get one?

### Create incoming webhook

 Go to [slack incoming weebhook webiste](https://my.slack.com/services/new/incoming-webhook/), login, and after that you will be able too se screen like this: 

![](/assets/posts/slack-fastlane/webhook-slack-url.png){: .center-image}

Choose your channel (for test purposes, I recommend choosing a direct message to yourself). Add click `Add incoming WebHooks integration`.
After you learn how it works, you can generate a URL for specially created Channel in your slack team.

Next step is to copy the Webhook URL

![](/assets/posts/slack-fastlane/webhook-slack-url2.png){: .center-image}

And that's all for know. You're able to set `slack_url` parameter in `Fastfile`.

## 3. Build Slack message

 First of all, [here](https://docs.fastlane.tools/actions/slack/) you can find a official documentation for Slack action in Fastlane tool. In this section, I'll try to give you a little closer look on that. 

![](/assets/posts/slack-fastlane/basic-message.png){: .center-image}

This is how the message from Fastlane could look like. Code for that is really simple: 


```ruby
slack(
    message: "CIM successfully uploaded to iTunesConnect.",
    success: true,
    slack_url: "https://your slack incoming webhook url"
)
```

# 🔧 Customizing slack message 🔧

### 👉 `deafult_payloads`

As you can see by default you get an information about `Git Commit`, `Git Commit Hash`, `Lane`, `Result`, `Git Author`. As we can read in the documentation:

> Don't add this key or pass nil if you want all the default payloads. The available default payloads are: `lane`, `test_result`, `git_branch`, `git_author`, `last_git_commit_message`, `last_git_commit_hash`.

Personally, I think that are very important informations, but if you want to customize the message by removing some of those you can look at this example:

```ruby
slack(
    message: "App successfully uploaded to iTunesConnect.",
    success: true,
    slack_url: "https://your slack incoming webhook url",
    default_payloads: [:git_branch, :last_git_commit_message]
)
```

Here is how a message with customized `default_payloads` looks like:

![](/assets/posts/slack-fastlane/default-payload-message.png){: .center-image}

### 👉 `attachment_properties`

Here real customizing begins. By using this property you can add any field to your slack message. Let's say that you want to add BUILD_NUMBER and URL to artifacts.

```ruby
slack(
    message: "App successfully uploaded to iTunesConnect.",
    success: true,
    slack_url: "https://your slack incoming webhook url",
    default_payloads: [:git_branch, :last_git_commit_message],
    attachment_properties: {
        fields: [
            {
                title: "Build number",
                value: ENV["BUILD_NUMBER"],
            },
            {
                title: "Artifacts URL",
                value: "https://url-to-your-artifacts.com",
            }
        ]
    }
)
```

#### Message with custom fields:

![](/assets/posts/slack-fastlane/message-with-custom-fields.png){: .center-image}

## 👉`success`

You can also define is that message will be successful or not. Success messages can be used if your app was successfully uploaded to iTunesConnect or many more cases. I prefer to use success for only that case. Because I think I don't need to know about every successful build or every successful test run. All above examples were examples when `success` parameter was set to `true` (it is done by default.).

Second option is to set `success` to `false`, and the a message will look a bit different:

![](/assets/posts/slack-fastlane/message-fail.png){: .center-image}

The red color suggests that something went wrong and you have to fix is a great way to notify you about it.

### 👉 other flags...

In this post, I focused, in my humble opinion, on most important keys which allows you to configure the slack message. Others are: `channel`, `message`, `use_webhook_configured_username_and_icon`, `icon_url`, `payload`. As I wrote above, more information about these keys you can find on [official documentation](https://docs.fastlane.tools/actions/slack/). 
    
# 🎉 Conclusion 🎉
    
I ❤️ Fastlane tool. It helps all developers to save a lot of time while development process. In my humble opinion - one of the puzzles of this process is a Continous Integration in the full sense of the word. How do I understand the Continous Integration? As you can read about it on [wikipedia](https://en.wikipedia.org/wiki/Continuous_integration)

>In software engineering, continuous integration (CI) is the practice of merging all developer working copies to a shared mainline several times a day.

it 100% true, but for me, it is also a state when I as a developer can be notified by CI agent about successes and errors without special checking them before pushing changes to the repository. But please don't get me wrong. I don't recommend committing and pushing without compiling them.I'm talking about... For example, Imagine the situation that you have a number of projects that contains a number of tests (Yes, I assume you're writing tests 😉).And I think you don't want to run them every time before I'll push new changes to the repository. That's the way you configure whole CI stuff to avoid it. Lets CI agent does it for you. In most cases all the tests will succeed 😉), so you can work ***continuously***. But if somehow it will happen and the tests will fail - let the CI agent ping you on a Slack 😉

This post was primarly posted on my company [blog](https://brightinventions.pl/blog/slack-fastlane/)