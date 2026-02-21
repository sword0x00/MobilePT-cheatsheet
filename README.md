# MobilePT-cheatsheet
Mobile (Android&amp;IOS) Penetration Tester Specialist Cheatsheet 



**Table of Contents**
- [Android](#Android)
  - [Start Android Studio For Kali](#start-android-studio-for-kali)
  - [adb](#adb)
  - [apktool](#apktool)
  - [jadx](#jadx)
  - [Network Interception](#network-interception)
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
--------------------------------
export QT_QPA_PLATFORM=xcb
./emulator -avd Pixel_6_API_33 -no-snapshot -scale 0.5 -memory 2048

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
jarsigner -verbose -keystore research.keystore /dist/app.apk research_key
sudo /home/kali/Android/Sdk/build-tools/36.1.0/apksigner sign --ks ../research.keystore /dist/io.hextree.reversingexample.apk
sudo [...]/build-tools/34.0.0/zipalign -p -f -v 4 ./dist/<apktool_build>.apk aligned.apk

```
## jadx
```
### Open jadx via GUI
 jadx-gui '/io.hextree.weatherusa_update1.apk' 

### compare two apks for on application
/opt/jadx/bin/jadx io.hextree.weatherusa_update1.apk -d app2
/opt/jadx/bin/jadx io.hextree.weatherusa_update1.apk -d app1
then open vs code
install  Compare Folders 
you will green lines and red lines

-----------------------------------------------------------------

## run the native lib against it self  & defeat a basic JNI obfuscation by calling the same functions from a custom app we build
- copy all native libiraries from obfusticated app , and paste it in /Projectview/app/src/main/jniLibs in our custom and small app (PoC)
- go to /src/main/java and creat a class look like the class in the obfuscated app ex:- io.hextree.weatherusa.InternetUtil , to be
        ===================
          package io.hextree.weatherusa;// obustcated app package name
          public class InternetUtil { // name of class where the native lib loaded in obfustcated app
              private static native String getKey(String str);
              public static String solve(){
                  System.loadLibrary("native-lib");
                  return getKey("moiba1cybar8smart4sheriff4securi");
              }
          }
        ==================
- then retriev the funcion you creat from the preivous class into main class
        =====
        val homeText = findViewById<TextView>(R.id.home_text_view)
        homeText.text = "proffffffffffff of C PoC"+ InternetUtil.solve();
        ====
```
## Network Interception

![image](https://github.com/user-attachments/assets/7b10d3be-8fef-41e4-88e0-713ce7733d32)
<img width="1658" height="907" alt="image" src="https://github.com/user-attachments/assets/25de1f2c-593f-49df-af57-9c1d658d1716" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7e60e759-0edf-444b-a09e-9d7756c59fb3" />


```
# Capute tcp trafic via dumping
./emulator -tcpdump packets.cap -avd Emulator_API_37

# when apps do ignore the proxy settings, we have to use other techniques
--> Patching with apktool.
--> Dynamic instrumentation.

----------------------------------------------------------------------------

# Install System Certs on Android 13 (install your user certificate in system CA)
1- configure you wifi proxy
2- install burp in user certificate
3- adb root
4- adb shell
5-
    # Backup the existing system certificates to the user certs folder
    cp /system/etc/security/cacerts/* /data/misc/user/0/cacerts-added/
    
    # Create the in-memory mount on top of the system certs folder
    mount -t tmpfs tmpfs /system/etc/security/cacerts
    
    # copy all system certs and our user cert into the tmpfs system certs folder
    cp /data/misc/user/0/cacerts-added/* /system/etc/security/cacerts/
    
    # Fix any permissions & selinux context labels
    chown root:root /system/etc/security/cacerts/*
    chmod 644 /system/etc/security/cacerts/*
    chcon u:object_r:system_file:s0 /system/etc/security/cacerts/*
    
----------------------------------------------------------------------------

# Install System Certs on Android 14
1- configure you wifi proxy
2- install burp in user certificate
3- copy below script and name it to be "certs.sh"
    =======================================================================
    # Create a separate temp directory, to hold the current certificates
    # Otherwise, when we add the mount we can't read the current certs anymore.
    mkdir -p -m 700 /data/local/tmp/tmp-ca-copy
    
    # Copy out the existing certificates
    cp /apex/com.android.conscrypt/cacerts/* /data/local/tmp/tmp-ca-copy/
    
    # Create the in-memory mount on top of the system certs folder
    mount -t tmpfs tmpfs /system/etc/security/cacerts
    
    # Copy the existing certs back into the tmpfs, so we keep trusting them
    mv /data/local/tmp/tmp-ca-copy/* /system/etc/security/cacerts/
    
    # Copy our new cert in, so we trust that too
    cp /data/misc/user/0/cacerts-added/* /system/etc/security/cacerts/
    
    # Update the perms & selinux context labels
    chown root:root /system/etc/security/cacerts/*
    chmod 644 /system/etc/security/cacerts/*
    chcon u:object_r:system_file:s0 /system/etc/security/cacerts/*
    
    # Deal with the APEX overrides, which need injecting into each namespace:
    
    # First we get the Zygote process(es), which launch each app
    ZYGOTE_PID=$(pidof zygote || true)
    ZYGOTE64_PID=$(pidof zygote64 || true)
    # N.b. some devices appear to have both!
    
    # Apps inherit the Zygote's mounts at startup, so we inject here to ensure
    # all newly started apps will see these certs straight away:
    for Z_PID in "$ZYGOTE_PID" "$ZYGOTE64_PID"; do
        if [ -n "$Z_PID" ]; then
            nsenter --mount=/proc/$Z_PID/ns/mnt -- \
                /bin/mount --bind /system/etc/security/cacerts /apex/com.android.conscrypt/cacerts
        fi
    done
    
    # Then we inject the mount into all already running apps, so they
    # too see these CA certs immediately:
    
    # Get the PID of every process whose parent is one of the Zygotes:
    APP_PIDS=$(
        echo "$ZYGOTE_PID $ZYGOTE64_PID" | \
        xargs -n1 ps -o 'PID' -P | \
        grep -v PID
    )
    
    # Inject into the mount namespace of each of those apps:
    for PID in $APP_PIDS; do
        nsenter --mount=/proc/$PID/ns/mnt -- \
            /bin/mount --bind /system/etc/security/cacerts /apex/com.android.conscrypt/cacerts &
    done
    wait # Launched in parallel - wait for completion here
    
    echo "System certificate injected"
    ================================================================================
4- adb push ./certs.sh /data/local/tmp
5- adb root
6- adb shell
8- sh /data/local/tmp/certs.sh

OR 
by httptook kit insted of 8 steps above

-------------------------------------------------------------------------------------------

# Patching with apktool (we could do this to edit in network security config xml file)

    # unpack the target .apk
    apktool d translate.apk
    
    # modify the AndroidManifest.xml to add a networkSecurityConfig
        ========================
          in application tag --> add below
          android:networkSecurityConfig="@xml/network_security_config"
        ==========================
    # create a permissive res/xml/network_security_config.xml
        ===========================
          <network-security-config>
              <base-config>
                  <trust-anchors>
                      <certificates src="system" />
                      <certificates src="user" />
                  </trust-anchors>
              </base-config>
          </network-security-config>
        ===========================
    cd translate
    
    # repackage the .apk
    apktool b
    
    # ensure the .apk is zipaligned
    [...]/build-tools/34.0.0/zipalign -p -f -v 4 ./dist/translate.apk translate2.apk
    
    # create a keystore to sign the apk
    keytool -genkey -v -keystore research.keystore -alias research_key -keyalg RSA -keysize 2048 -validity 10000
    
    # sign the apk with apksigner
    [...]/build-tools/34.0.0/apksigner sign --ks ./research.keystore ./translate2.apk
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

