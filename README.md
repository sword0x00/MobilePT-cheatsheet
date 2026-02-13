# MobilePT-cheatsheet
Mobile (Android&amp;IOS) Penetration Tester Specialist Cheatsheet 



**Table of Contents**
- [Android](#Android)
  - [Start Android Studio For Kali](#start-android-studio-for-kali)



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
```
## adb
```
adb version
adb devices
adb -s emulator-5554 shell ---> Specify the active device using the -s parameter
adb -d shell ---> Specify to use a single USB device using the -d parameter

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

