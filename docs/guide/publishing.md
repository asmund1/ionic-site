---
layout: docs_guide
title: "Publishing your app"
chapter: publishing
---

Previous: <a href="building.html">Building out your app</a>

# Chapter 6: Publishing your app

Now that we have a working app, we are ready to push it live to the world! Since the Ionic team already submitted the Todo app from this guide to the app store, chances are you'll want to follow this chapter with a new app that you make on your own.

## Clean up

So first, we need to generate a release build of our app, targeted at each platform we wish to deploy on. Before we deploy, we should take care to adjust plugins needed during development that should not be in production mode. For example, we probably don't want the debug console plugin enabled, so we should remove it before generating the release builds:

```bash
$ ionic plugin rm cordova-plugin-console
```

## Configuration

We will generate a release build based on the settings in the `config.xml`. Your Ionic app will have preset default values in this file, but if you need to customize how your app is built, you can edit this file to fit your preferences. Check out [the config.xml file](http://cordova.apache.org/docs/en/latest/guide/platforms/android/config.html) documentation for more information.

## Icon

Instead of the Cordova/Ionic icon, you might want a to use a unique icon for your app. Follow the instructions to generate correct icon files for both Android and iOS using the `ionic resources` command line option described in [Icon and Splash Screen Image Generation](http://ionicframework.com/docs/cli/icon-splashscreen.html)

# Android Publishing

To generate a release build for Android, we can use the following ionic cli command:

```bash
$ ionic build --release android
```

Next, we can find our *unsigned* APK file in `platforms/android/build/outputs/apk`. In our example, the file was `platforms/android/build/outputs/apk/HelloWorld-release-unsigned.apk`. Now, we need to sign the unsigned APK and run an alignment utility on it to optimize it and prepare it for the app store. If you already have a signing key, skip these steps and use that one instead.

Let's generate our private key using the `keytool` command that comes with the JDK. If this tool isn't found, refer to the [installation guide](installation.html):

```bash
$ keytool -genkey -v -keystore my-release-key.keystore -alias alias_name -keyalg RSA -keysize 2048 -validity 10000
```

You'll first be prompted to create a password for the keystore. Then, answer the rest of the nice tool's questions and when it's all done, you should have a file called `my-release-key.keystore` created in the current directory.

__Note__: Make sure to save this file somewhere safe, if you lose it you won't be able to submit updates to your app!

To sign the unsigned APK, run the `jarsigner` tool which is also included in the JDK:

```bash
$ jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore my-release-key.keystore HelloWorld-release-unsigned.apk alias_name
```

This signs the apk in place. Finally, we need to run the zip align tool to optimize the APK. The `zipalign` tool can be found in `/path/to/Android/sdk/build-tools/VERSION/zipalign`. For example, on OS X with Android Studio installed, `zipalign` is in `~/Library/Android/sdk/build-tools/VERSION/zipalign`:

```bash
$ zipalign -v 4 HelloWorld-release-unsigned.apk HelloWorld.apk
```

Now we have our final release binary called `HelloWorld.apk` and we can release this on the Google Play Store for all the world to enjoy!

<small><i>(There are a few other ways to sign APKs. Refer to the official [Android App Signing](http://developer.android.com/tools/publishing/app-signing.html) documentation for more information.)</i></small>

## Google Play Store

Now that we have our release APK ready for the Google Play Store, we can create a Play Store listing and upload our APK.

To start, you'll need to visit the [Google Play Store Developer Console](https://play.google.com/apps/publish/) and create a new developer account. Unfortunately, this is not free. However, the cost is only $25 compared to Apple's $99.

Once you have a developer account, you can go ahead and click "Publish an Android App on Google Play" as in the screenshot below:

![New google play app](http://ionicframework.com.s3.amazonaws.com/guide/0.1.0/5-play.png)

Then, you can go ahead and click the button to edit the store listing (We will upload an APK later). You'll want to fill out the description for the app. Here is a little preview from when we filled out the application with the Ionic Todo app:

![Ionic Todo](http://ionicframework.com.s3.amazonaws.com/guide/0.1.0/5-play2.png)

When you are ready, upload the APK for the release build and publish the listing. Be patient and your hard work should be live in the wild!

# iOS Publishing

As we mentioned in [Chapter 4: Testing your app](testing.html), you will need to set up Xcode with a developer account and the proper certificates. Follow the [App Distribution Guide](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/Introduction/Introduction.html) and especially [Maintaining Identifiers, Devices, and Profiles](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingProfiles/MaintainingProfiles.html) to correctly set up your Developer Accoount, the App ID, and the provisioning profiles.

![Register App ID](img/docs/guide/register-app-id.png)

__Note__: You will have to update your config.xml file with the App ID that you create in the Developer Center.

Now that we have Xcode set up, create/update your Xcode project:

```bash
$ ionic prepare ios
```

Click the <yourappname>.xcodeproj under platforms/ios/ to open the project in Xcode.

To create an archive package that you can upload to the Apple App Store, first in the list of devices to build for, select a connected device or `Generic iOS Device`. Then select `Product` in the top level menu, and finally select `Archive`.

![Select device from dropdown](img/docs/guide/select-device.png)

![Click 'Archive' in 'Product' menu](img/docs/guide/select-archive.png)

This will compile the project into an iOS app and create an archive package. You might be asked to select the correct provisioning profile for the app, select the team and the profile you created. You might also be shown an error if Xcode can't locate the profile. Usually Xcode is able to remedy this problem itself when you click the `Fix Issue` button. 

![Compiling and Archiving app](img/docs/guide/building.png)

If this doesn't work, check that your profiles are downloaded by selecting `Preferences` from the top level menu, then select the `Account` tab and finally your account and press the `View Details...` button. Check that your profiles are shown and downloaded.

Another solution might be to manually set your team and provisioning by selecting the item matching you app name (item on the top of the tree) in the navigator on the left hand side of the main Xcode window. To choose your team, select the `Info` tab and choose your team in the `Team` dropdown in the `Identity` section. To choose provisioning profile, select the `Build Settings` tab and find `Provisioning Profile` in the `Code Signing` section.

After successfully creating the archive, the Organizer window will open. You can also open this window directly by selecting `Window` in the top level menu and then `Organizer`. In the Organizer, you will see a list of all the archives you have created. Select the archive you just made and then click on the `Submit to App Store...` button on the right hand side. You will again be asked to select your team and provisioning profile as well as some other questions, selecting the default works in most cases.

![Ionic TODO in The XCode Organizer](img/docs/guide/organizer.png)

![Upload app to Apple App Store](img/docs/guide/upload-dialog.png)

## Apple App Store / iTunes Connect

Your app is now uploaded to Apple. You now need to create and App Store listing, link your app to it and submit it to Apple's review process.

To do so you need to log in to [iTunes Connect](http://itunesconnect.apple.com). Select `My Apps` and then the `+` and `New App` in the upper left hand corner to create a new app. Choose a name for your app, matching the one in `config.xml`, and in the `Bundle ID` dropdown select the app ID you created in Developer Center earlier, again matching `config.xml`.

![Add App in iTunes Connect](img/docs/guide/add-app.png)

Select `1.0 Prepare for Submission` and fill in the relevant information, including description, screenshots, contact information and any special instructions for the review team. You might also want to upload a high definition App icon to be used in the App Store, you can do this in the `General App Information` section.

In the `Build` section you should be able to choose the archive that you uploaded from Xcode. Note that it usually takes some time from you upload an archive until it is available, this can be from 10 minutes up to 24 hours(!).

![Select uploaded build](img/docs/guide/select-build.png)

Finally, select the `Save` button and the `Send to Review`. Wait about 1 week for the review to complete and your app will be available in the App Store! Alternatively you will receive an email about an issue you have to correct for the app to be accepted, fix it and repeat the whole process.

# Updating your App

As you develop your app, you'll want to update it periodically.

In order for the Google Play Store and Apple App Store to accept updated APKs/IPAs, you'll need to edit the `config.xml` file to increment the `version` value, then rebuild the app for release and submit it to the respective store.

<!--[Chapter 6: Closing Thoughts](closing.html)-->
