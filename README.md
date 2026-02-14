# MobilePT-cheatsheet
Mobile (Android&amp;IOS) Penetration Tester Specialist Cheatsheet 



**Table of Contents**
- [Android](#Android)
  - [Start Android Studio For Kali](#start-android-studio-for-kali)
  - [adb](#adb)
  - [apktool](#apktool)
  - [Intents](#intents)



# Android
## Start Android Studio For Kali
```
# Open Andriod studio for kali
cd /opt/android-studio/bin
./studio.sh

# Open Andriod Emulator
cd ~/Android/Sdk/emulator
./emulator -list-avds
./emulator -avd Device_name_here

# Open Genymobile Emulator
/opt/genymobile/genymotion/genymotion

```
## adb
```
adb version
adb devices
adb -s emulator-5554 shell ---> Specify the active device using the -s parameter
adb -d shell ---> Specify to use a single USB device using the -d parameter
adb push <local_file_on_computer> <target_path_on_device> ---> upload file from your Pc to your device
adb pull <file_path_on_device> [<optional_target path_on_the_computer>] ---> to download file
adb install <path to .apk>
adb shell pm list packages ---> Lists all installed packages - including system packages.
adb shell pm list packages -3  ---> List only third party packages.
adb shell pm path <package_name> ---> get the path to the APK.
adb shell pm clear <package_name> ---> Clear the application data without removing the actual application.
adb shell dumpsys package <package_name> ---> List information such as activities and permissions of a package.
adb shell am start <package_name>/<activity_name> ---> Starts the activity of the specified package.
adb uninstall <package_name>
Refference ---> https://developer.android.com/tools/adb#pm

adb logcat -v <log_format_like_'brief'>
adb logcat "MainActivity:V *:S"
adb logcat -v brief "MainActivity:V *:S"
```
## apktool
```
### apktool to desassmple
sudo apktool d  io.hextree.reversingexample.apk
### apktool to rebacking the files to be apk 
apktool b

### Creating a Keystore
keytool -genkey -v -keystore research.keystore -alias research_key -keyalg RSA -keysize 2048 -validity 10000
### Signing an APK
# jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore research.keystore app.apk research_key
# On newer Android versions SHA1 signatures still get rejected. In that case simply use the default algorithms
jarsigner -verbose -keystore research.keystore app.apk research_key

```
## Intents
```
## Declares our intention (Intent) to view (ACTION_VIEW) the URL
Intent browserIntent = new Intent(Intent.ACTION_VIEW, Uri.parse("https://hextree.io/"));
startActivity(browserIntent);

## To receiving Intent
1) First go to manifist.xml:-
 <activity
            android:name=".SecretIntent"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.SEND" />
                <data android:mimeType="text/plain" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </activity>
2) Then create new activity:-
        Intent recieveIntent= getIntent();
        String sharedText = recieveIntent.getStringExtra(Intent.EXTRA_TEXT);
        if(sharedText!=null){
            TextView debugText = findViewById(R.id.debug_test);
            debugText.setText("shared: "+sharedText);
        }
        
```

