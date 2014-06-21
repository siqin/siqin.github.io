---
layout: post
title:  "Basic life cycle of an iOS app"
date:   2014-06-21 19:30:00
categories: iOS.io
tags: iOS multitasking
---

This article is about state changes of an iOS application, and how we should respond to those changes.

- Main function and delegate callbacks
- Responding to the launch
- When becomes active
- When interrupted
- Entering the background
- Returning to the foreground
- Being teminated

##Main function and delegate callbacks

When we create a new iOS application project, for example using the `Single View Application` template, Xcode will create several files for us, such as main.m, AppDelegate.m and so on.<br/>
In the `main.m`, we create the very important UIApplication object and set up the main run loop, along with the app delegate to handle the state transitions:

{% highlight objc %}
int main(int argc, char * argv[])
{
    @autoreleasepool {
        return UIApplicationMain(argc,
                                 argv,
                                 nil,
                                 NSStringFromClass([AppDelegate class]));
    }
}
{% endhighlight %}

In an autorelease pool, the `main` function just returns a never-return  `UIApplicationMain` function, which does the things described above. 

And in file `AppDelegate.m`, the following methods are implemented to respond state changes:

- `- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions`
- `- (void)applicationWillResignActive:(UIApplication *)application`
- `- (void)applicationDidEnterBackground:(UIApplication *)application`
- `- (void)applicationWillEnterForeground:(UIApplication *)application`
- `- (void)applicationDidBecomeActive:(UIApplication *)application`
- `- (void)applicationWillTerminate:(UIApplication *)application`

##Responding to the launch

First, the app is not running, that is in the `not running` state.<br/>
When the user taps the icon to launch the app, iOS will load the app into the memory. After some UI initialization that we can ignore, `- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions` will be called. The `launchOptions` may contain some useful information about why the app was launched, we can check it when needed.

If we don't use storyboards, we should prepare the views to display ourselves.<br/>
Besides, we can do some other initialization that is critical to our app, but we should keep in mind that the launch should be as lightweight as possible, because if our app could not start handling events in 5 seconds, it will highly probably be killed by the watchdog. So, we should not do synchronous networking request or large datasets migration in `application:didFinishLaunchingWithOptions:`.

##When becomes active

During the launch, the app's state is still `inactive`. When it becomes `active`, the `applicationDidBecomeActive:` will be called.

Like `viewWillAppear:` and `viewDidAppear:`, we should not do heavy jobs in `application:didFinishLaunchingWithOptions:`, but place time-consuming codes in `applicationDidBecomeActive:`, better providing a user-friendly interaction.

If the app becomes `active` from `background` or `interrupted`, we also need to resume tasks that have been paused before.

##When interrupted

Our app may be interrupted on some occasions and transitioning to `inactive` state, like locking the screen, ringing of the alarm clock, pulling down the notification center, or pulling up the control center.

When interrupted, the method `applicationWillResignActive` will be called, we can disable timers or pause the game here.

One important thing is to be careful with files protected by the `NSFileProtectionComplete` protection option. If the user locks the screen, we should close these files here and open them in `applicationDidBecomeActive:`.

##Entering the background

If the user presses the Home button, the app will resign `active` state and then transitioning to `background` state, and `applicationDidEnterBackground:` will be called.

Because apps in the background may be `teminated` without notice, we need to write the unsaved changes to the disk. However, as the same with `application:didFinishLaunchingWithOptions:`, we only have approximately 5 seconds to do things in `applicationDidEnterBackground:`, or our app will be killed due to being unresponsive.<br/>
If 5 senconds is not enough, we can call `beginBackgroundTaskWithExpirationHandler:` to request more time to execute long-running tasks in a secondary thread.

One reason, also the common reason, for being teminated in the background is the low memory, so we need to free up as much memory as we can, such as image objects and large data files on the disk.

The thing most likely to be ignored may be the snapshot automatically taken by the iOS. If our app contains sensitive information on current view, for example the bank account balance, we should hide it properly -- presenting a gesture lock view will be good.

##Returning to the foreground

When returns to the foreground, our app needs to respond to changes that can affect our app's appearance or state.

For instance, we were presenting a custom photo picker before entering the background. After the user pressed the Home button, he took some photos. So when our app comes to the foreground, we had better refresh our photo picker to show the newest photos.

Also, we should undo the changes made on entering the background in `applicationWillEnterForeground:`.

##Being teminated

If the user presses the Home button twice, he can teminate the app by swiping it out (iOS 7 and later) or long pressing to delete it (earlier than iOS 7). Under this circumstance, `applicationWillTerminate:` will be called, but this time we can not request extra execution time.

##Conclusion

By knowing the transitions of an iOS app, we can develop a more graceful and user-friendly app.

##References
[iOS App Programming Guide](https://developer.apple.com/library/ios/documentation/iphone/conceptual/iphoneosprogrammingguide/Introduction/Introduction.html)
