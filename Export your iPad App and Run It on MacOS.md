# Export your iPad App and Run It on MacOS

## 0. Necesssities

First of all, I assumed that this operation is suitable for only Apple Silicon Mac. If you still use Intel Mac, this operation is not available because of CPU Instruction Sets Architecture.

Then, what necessities does this operation have? It's OK for us to build macOS app. It's also very easy for developers to build their iOS app and macOS app simultaneously right now. However, there are some benefits that can encourage you to do so:

1. Unified and modern UI;
2. Suitable for the new standard since macOS 11(Big Sur);
3. Less bug;
4. Less alternation for multiplatform development.

I want to emphasise the importance of the fourth benefit. When you choose to run your iPad app on macOS, you can just considerate the performance of iOS. In fact, iPadOS's SDK is the same as iOS's. In addition, macOS and building tool can do a lot of jobs to make your app compatible automatically when you do so.

## 1. Steps

### 1.1 Set your taget's emulator

The default emulator for you to test your iOS/iPadOS app is the latest iPhone. You should choose the emulator "Mac(built for iPad)".

### 1.2 Run it

Click the run button to test your application to test your application.

### 1.3 Find the App in Finder

After the app is opened, you can see the app icon on your dock. Right-click(or click using double fingers) the icon, find 'Options' and click 'Show in Finder'. Then you can see your app in some folder.

### 1.4 Copy Your App in Application Folder

When you find your app in somewhere(because the storage location changes), you should drag your app to the Application folder. Then you will see your app appearing in your launcher.

### 1.5 Trust Your App

If you want to open it for the first time, you will see an alert showing that "this app isn't identified". Don't worry about that, you can trust your app easily in the Security and Privacy of System Preferences. Maybe you should unlock the lock in the lower-left corner of the panel in advance.

### 1.6 Enjoy It

When you complete all the steps metioned above, your app, which is built for iPadOS, can run on macOS! You can see that the UI is much more modern than app which is built by macOS naive SDK.

## 2. Disadvantages

I musk acknowledge that there are some disadvantages of running iPadOS app on macOS. Here are some:

1. Some restrictions because of features lacking in iOS SDK;
2. Possibly more memory needed;
3. Potential bugs;
4. Validation problems because of unenrollment of Apple Developer Program($99);

However, it is very rare to meet because engineers in Apple considered a lot to make the converting framework very robust.

## 3. Conclusion

In this article, I discuss why we should run iPadOS app on macOS and how to do that, while I list some disadvantages of doing so. In sum, it's a good practice for most developers in most situations. Apple also encourages developers to do so in order to unify the style of their apps in iOS, iPadOS and macOS. 
