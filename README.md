Smallest APK
==========

Why?
----------
Tons of code, gygabytes of resources, high speed internet connection...
Nowadays applications in Google Play Store looks like a big monsters.
But what is the smallest apk file that can be created?
I just tried to figure out it by myself.
I didn't try to play with bytes directly – I used command line and available tools.

Let's start
-----------
Because apk size can be various depending on tools and libs version, I mention them.
At first, we need to install all the soft needed:
 - JDK 1.7.0_11: http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html#jdk-7u11-oth-JPR
 - Android SDK 22.3: http://dl.google.com/android/android-sdk_r22.3-windows.zip
 - Ant 1.9.4: http://archive.apache.org/dist/ant/binaries/apache-ant-1.9.4-bin.zip

Then open Android SDK folder and run 'SDK Manager'.

Install:
 - android 4.4.2 (API 19) sdk platform (api 19, rev 3)
 - android sdk-build-tools: (rev 19.1)

That's it

Create project
------------
Create project using android sdk:

`android create project --target 1 --name k --path k --activity a --package c.k`

This will create android project for target 'Android-19' with name 'k', main activity 'a.java' in package 'c.k'.

Yes, we start reducing apk file size with unreadable names :)

Application source code
------------

`a.java` is pretty simple. With also unreadable names where possible:

`package c.k;

import android.app.Activity;
import android.os.Bundle;
import android.widget.TextView;

public class a extends Activity
{    
    @Override
    public void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);
        TextView t = new TextView(this);
        t.setText("Hello world");
        setContentView(t);
    }
}`

`AndroidManifest.xml`

Application and Activity labels and icons are not required, so we remove them

`<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="c.k">
	<application>
		<activity android:name=".a">
			<intent-filter>
				<action android:name="android.intent.action.MAIN"/>
				<category android:name="android.intent.category.LAUNCHER" />
			</intent-filter>
		</activity>
	</application>
</manifest>`

compile and build this project using:

`ant release`

we receive apk file of **3042** bytes.
Not bad, but

Go further
-----------------
Let's look at `xmlns:android`.
This is just a declaration. We can write: `xmlns:a` and then use `a:name` instead of `android:name`.
Also we can write content of AndroidManifest.xml in one line.

Assume, that it doesn't matter what text we will show in our application, I replaced `Hello world` string with `android.R.string.ok`.
You can say, that this is a cheating. May be ;)

Apk file before signing is very small. But it doubles its size after signing.
So, we need to reduce keystore size.
There are many fields in keystore (Organization, Name, Country etc.), but only any one field is required.
So, fill `Organization` with `.` (dot), and create smallest sufficient passwords (`......` six dots).

With this modifications apk file size becomes **2980** bytes!

But...

Go further
------------------
I found great article about building android applications from command line: http://geosoft.no/development/android.html

So...
###Ant, get out of my way!
Let's build!

Set Environment and other Variables:

`set JAVA_HOME=C:\Program Files\Java\jdk1.7.0_11
set ANDROID_HOME=D:\Work\Github\smallapk\android-sdk-windows
set DEV_HOME=%CD%
set AAPT_PATH=%ANDROID_HOME%/build-tools/19.1.0/aapt.exe
set DX_PATH=%ANDROID_HOME%/build-tools/19.1.0/dx.bat
set ANDROID_JAR=%ANDROID_HOME%/platforms/android-19/android.jar
set ADB=%ANDROID_HOME%/platform-tools/adb.exe
set PACKAGE_PATH=c/k
set PACKAGE=c.k
set MAIN_CLASS=a`

Create R.java (it will not be created because we don't have /res):

`call %AAPT_PATH% package -f -m -J %DEV_HOME%/src -M %DEV_HOME%/AndroidManifest.xml -I %ANDROID_JAR%`

Compile src, a.class will be created:

`mkdir obj
call "%JAVA_HOME%\bin\javac" -d %DEV_HOME%/obj -cp %ANDROID_JAR% -sourcepath %DEV_HOME%/src %DEV_HOME%/src/%PACKAGE_PATH%/*.java`

Dex compiled sources into classes.dex:

`mkdir bin
call %DX_PATH% --dex --output=%DEV_HOME%/bin/classes.dex %DEV_HOME%/obj`

Create unsigned apk file:

`call %AAPT_PATH% package -f -M %DEV_HOME%/AndroidManifest.xml -I %ANDROID_JAR% -F %DEV_HOME%/bin/AndroidTest.unsigned.apk %DEV_HOME%/bin`

Sign apk file with out small keystore:

`call "%JAVA_HOME%\bin\jarsigner" -sigalg SHA1withRSA -digestalg SHA1 -keystore %DEV_HOME%/j.jks -storepass ...... -keypass ...... -signedjar %DEV_HOME%/bin/AndroidTest.signed.apk %DEV_HOME%/bin/AndroidTest.unsigned.apk .`

This way we get apk file of **2905** bytes!!!

Go further?
-------------

Can you create smaller apk file? 
#### #ChallengeAccepted
