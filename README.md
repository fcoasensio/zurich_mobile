Zurich Mobile App
=================

A mobile app for www.zueriwieneu.ch based on mSociety's FixMyStreet platform.

Setup
-----
This project uses Apache Cordova to produce Android and iOS apps. There is
some mildly complicated configuration and setup required to be able to develop
with it. The following all assumes you're working on a Mac.

1. Make sure you have the latest versions of XCode, JDK 1.8, the Android SDK, node and
npm installed. It's a very good idea to have installed the Intel HAXM versions
of the Android emulator because they're about 100 times faster to run. You need
to download it from the Android SDK Manager (run `android` on the command line)
and then actually run the `.dmg` that this creates in your sdk folder. (Alternatively `brew cask install intel-haxm` if you use [Homebrew Cask](http://caskroom.io).)

2. Install the cordova CLI with npm: `npm install -g cordova`
Note that this is not the same as the phonegap CLI and the two should not be
mixed up. The latter gives you access to Adobe's proprietary phonegap build
service, which we **don't** use!

3. Install the latest android api and build tools packages within the Android
SDK Manager (run `android` on the command line to launch it)

4. Checkout the project

5. `cd` into the project directory and run `cordova prepare` to load up the
cordova platforms and plugins we use.

7. Install Apache Ant (`brew install ant`, if you use Homebrew) for Android build support, and `ios-sim` (`npm install -g ios-sim`) for iOS simulator support.

8. Create a new 'Android Virtual Device' for emulating a real device by running `android avd` and using one of the 'Device Definitions' on the second tab as a template. It doesn't matter which one, but set the CPU type to 'Atom (x86)' otherwise it will be very very slow. Enable 'Use Host GPU', if available, to massively speed up the UI. Ticking 'Hardware keyboard present' will allow you to use your keyboard instead of hunting-and-pecking the on-screen keyboard.

9. Copy `www/js/config.js-example to www/js/config.js` and edit if needed

10. To run the project on one of the platforms, use: `cordova emulate ios` or `cordova emulate android`

Tips and Tricks
--------------
- Make sure you read the documentation for Cordova from http://cordova.apache.org/
**not the Phonegap site** - the two vary in infuriating and subtle ways and much
of the stackoverflow-esque info on the web is confused about which one it's for.
Particularly in the options for things in `config.xml` which is where all the
magic happens.
- You can use `ios-sim` to launch the iOS emulator directly with something like:
`ios-sim launch platforms/ios/build/emulator/Zuri\ Wie\ Neu.app --devicetypeid "com.apple.CoreSimulator.SimDeviceType.iPhone-6, 8.0"` after you've built the project via a previous
emulator run or a direct build via `cordova build ios`. This allows you to
specify a different device than the default one. To see the available options
for `--devicetypeid` run `ios-sim showdevicetypes`.
- You can open the iOS project in XCode if you prefer to run it that way, the
project file is in `platforms/ios`
- The `platforms`, `plugins` and `hooks` folders are auto-generated by Cordova
no need to check them in (hence why they're .gitignored), you should only need
to check in the `www` folder and `config.xml`, plus possibly `/merges` if you
ever use that functionality.
- To check the console log output when emulating iOS, run: `tail -f console.log`
Cordova by default writes it out to that file in your project root
- To check the console log output when emulating Android, cd to
`platforms/android/cordova` and run `./log`. I found that I needed to be in the
directory for it to actually print anything, YMMV.
- Leave the emulators running once they start, it's much quicker!
- If you're using your own FMS backend, you'll need to add an `<access origin="" />` tag to `config.xml` to allow access from within the app. **Make sure** you remove any such lines before building/committing!

Upgrading
---------
Cordova now includes version numbers for the platforms and plugins in
`config.xml` so it's possible to use the command line tools to update
everything.

1. Update the CLI: `npm update cordova`
2. Update each platform: `cordova platform update ios --save`, `cordova platform update android --save`
3. Update each plugin. Unfortunately, it doesn't seem possible to upgrade all
   of the plugins in one go, so you'll have to type out
   `cordova plugin update cordova-plugin-name --save` for every single one.
   You can get a list that's easy to edit into a script from
   `cordova plugin list`.
4. Refer to the upgrade guides:
   https://cordova.apache.org/docs/en/latest/guide/platforms/android/upgrade.html
   and
   https://cordova.apache.org/docs/en/latest/guide/platforms/ios/upgrade.html
   for anything you need to change between versions. The plugins helpfully
   print notices for some things that have changed when you install them.
5. Test the changes

Releasing
---------
### Android
To release the app on Android, you need to do the following:

1. Change your config.js to include production settings

2. Bump the version code in config.xml, both the main one and the android specific one

2. Build a release version of the app: `cordova build android --release`

3. Sign that `.apk` (the cordova command tells you where it put it):
    1. Clone the mySociety keys repository
    2. `cd` into the folder containing your new release `.apk`
    3. Sign the .apk with our key: `jarsigner -verbose -keystore <path-to-keys-repo>keys/android/android_keystore -sigalg SHA1withRSA -digestalg SHA1 ZuriWieNeu-release-unsigned.apk fixmyzurich` Double check that you're signing the right .apk here as there
    will be debug ones too.

      This will ask first for a password for the keystore (it's in the usual place
      if you're a mysociety developer), then a password for the app specific key,
      (`fixmyzurich` in the command above is a special shortname for the app that
      identifies which key to use.)

4. Verify that the signing was ok: `jarsigner -verify -verbose -certs ZuriWieNeu-release-unsigned.apk` (The signing doesn't change the name of your `.apk`). You should
see `sm` next to every file.

5. Align the `.apk` using `zipalign` (Note, you might have to manually find `zipalign` in `build-tools` inside the sdk-folder): `zipalign -v 4 ZuriWieNeu-release-unsigned.apk ZuriWieNeu.apk`

Note: most of this comes from: http://developer.android.com/tools/publishing/app-signing.html#signing-manually, you can also do it via Eclipse or Android Studio if you wish.


### iOS

#### App Store

To release the app in the iTunes App Store you need to do the following:

1. Change your config.js to include production settings

2. Bump the version code in config.xml, both the main one and the android specific one

3. Run the emulator to make sure you've built the latest version of the app: `cordova emulate ios`

4. Open the app in XCode (the xcodeproj project file you need is in `platforms/ios`)

5. Select `Product > Archive` from the XCode menu

6. In the "navigator" window that pops up, select the latest build and then hit "Validate" in the top right. It'll ask to access your keychain, so you'll need to make sure you've installed the latest certificates there already.

7. Once the validation has finished, hit "Submit" and pick the certificate again to actually send it to Apple.

8. Now you need to log into iTunes Connect and add a new version of the app for this build, then submit it for review.


#### Notes and observations from doing the release process:

 * Whilst the PhoneGap strings in the app are already in German, the native UI presented by iOS (e.g. photo selection) won't be. Add a 'Localizations' key (of type array) to 'Custom iOS Target Properties' in Xcode, with a single string entry of 'German'.

#### Ad-Hoc Distribution

iOS allows you to distribute builds of your app directly to selected testers, either by sending them the `.ipa` file for installation via iTunes or via a specially-crafted web page they visit from their device. [More info](http://help.apple.com/deployment/ios/#/apda0e3426d7).

 1. Gather the device UDIDs from testers and add them to the ['devices' section](https://developer.apple.com/account/ios/device/) of the developer center.
 1. You'll probably have to re-download the provisioning profile for the app into Xcode so subsequent builds include the new device UDIDs.
 1. Open the `.xcodeproj` file in Xcode and run `Product > Archive`
 1. Select the archive in the Organizer window that subsequently pops up, then click `Export` and select `Save for Ad Hoc Deployment`.
 1. Follow the wizard, selecting `Export one app for all compatible devices`, and `Include manifest for over-the-air installation`.
 1. The wizard will ask you to provide values for the App URL, and a couple of image URLs. Put dummy values in if you don't know the final URLs for the `.ipa` and images yet, you can edit the output manifest file later.
 1. Copy the resulting `.ipa` and manifest to your webserver. Make sure they're served over HTTPS or iOS will refuse to install the app.
 1. Users can install the app on their devices by going to a URL of the form `itms-services://?action=download-manifest&url=[MANIFEST URL HERE]`
 1. The `ios_adhoc/beta.html` file might a useful starting point for a more friendly page that your users can be directed to.
