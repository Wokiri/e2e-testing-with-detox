# e2e Testing with Detox

# Setting up Detox for e2e testing in Mac

## 1. Install Xcode Command Line Tools on a Mac

### Alternative ONE- From a Command Prompt

In the macOS Terminal:

1. `xcode-select --install`

Verify a successful installation of Xcode Command Line Tools:

2. `xcode-select -p`


### Alternative TWO- Using Homebrew (Recommended)

Note: Homebrew is a popular Mac package manager-- it can install almost any open-source tool for developers.

It is the easiest and most flexible way to install the UNIX tools Apple didn’t include with macOS. It can also install software not packaged for your Linux distribution to your home directory without requiring sudo.

We'll here use Homebrew to install Xcode Command Line Tools.

1. Check if Homebrew is already installed.

In the macOS Terminal:

`$ brew --version`

If Homebrew is not installed:

`zsh: command not found: brew`

2. Homebrew provides an installation script you can run with a single command

`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`


### From:
> https://brew.sh/

3. Homebrew installation script asks you to enter your Mac user password.

4. Set Homebrew to PATH

Is automatically done on macOS 10.14 Mojave and newer.

Test by running:

`$ brew --version`

For older versions of macOS, follow the steps listed at the end of the installation process

```shell

echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"

```

Test by running:

`$ brew --version`

## Installing Detox

- Install Detox dependencies:

```sh

xcode-select --install
brew tap wix/brew
brew install applesimutils

```

- Install Detox CLI:

```sh

npm install -g detox-cli

```

# Install Emulators:

## 1. Android emulator

**Android needs Java 1.8 installed.**

On macOS, in particular, java comes from both the OS and possibly other installers such as `homebrew`.

### Test for java (option ONE)

```shell
java -version
```

This verifies that java is in-path and that the output contains something as this:

```shell
java version "1.8.0_121"
```

### Test for java (option TWO)

To test that Java is installed and working properly on your computer, run this [test applet](https://www.java.com/en/download/help/testvm.xml).

If java isn’t in your path or not even installed

1.1 Try setting path

```shell
% /usr/libexec/java_home -v 1.8.0_73 --exec javac -version
```

**EXTRAS**

- Finding all installed JDKs

```shell
/usr/libexec/java_home -V
```

- Getting values for `JAVA_HOME` for specific JDK versions

```shell
$ /usr/libexec/java_home -v 1.6 
/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home
$ /usr/libexec/java_home -v 1.7
/Library/Java/JavaVirtualMachines/jdk1.7.0_45.jdk/Contents/Home
$ /usr/libexec/java_home -v 1.8
/Library/Java/JavaVirtualMachines/jdk1.8.0_121.jdk/Contents/Home
```

-Executing specific versions of Java commands

```
...jdk/Contents/Home/bin/<command>
```
... for a specific version of the JDK. And it is independent of the setting of JAVA_HOME.

- Executing the default version of a java `<command>`


```shell
/usr/libexec/java_home --exec <command>
```



Install it:

1.2 Installing java

This can be done from the [official download site](https://www.java.com/en/download/help/mac_install.html).


## 2. Setting up Android SDK

### 2.1 Download SDK command line tools [here](https://developer.android.com/studio/index.html#downloads):

### 2.2 Extract your compressed file into the ``HOME`` folder.

### 2.3 Rename the extracted folder to "``Android``".
This will make the rest of this guide, and your time with the SDK, much easier.

### 2.4 Get default java location:

```shell
which java
```

### 2.5 Open the bin folder in the extracted download and find the SDK manager executable file. It may look like a terminal or shell command, but it will open a GUI as long as you have Java installed correctly.

### 2.6 In the SDK manager, you'll choose to install 

- Android SDK Tools
- Android SDK Platform-Tools
- Android Emulators


The tools will be installed into the application data folder, i.e ``.Android`` in the home folder.

### 2.7 In your Home folder is a file named .bash_profile.

Open it with any text editor. Never touch the .bashrc or .bash_profile files you might find in the /etc. directory!


Add these lines to the top of the file:

export PATH="$HOME/Android/tools:$PATH"
export PATH="$HOME/Android/platform-tools:$PATH"

### 2.8 Save the file and reboot your computer so that the new PATH is sourced properly.




## Test Project:

1. Install Expo CLI

- Expo CLI is a safe bet for a new React Native developer, since it has set of tools built around React Native, so that you only need a recent version of Node.js and a phone or emulator to get started within minutes.


```shell
npm install -g expo-cli
```

2. Install Node and Watchman

Note: `Watchman` is a tool by Facebook for watching changes in the filesystem. It is highly recommended you install it for better performance.

```shell
brew install node
brew install watchman
```

3. Create React Native app using React Native CLI called Hello Detox for testing


```shell
npx react-native init hello_detox
```

### 3.1 Get into the created directory

```shell
cd hello_detox
```

### 3.2 Run the sample app on your Android device or emulator

```shell
npx react-native run-android
```

- Configuring for Android

    - Go to your Android **build.gradle (project)**, and the update the minSdkVersion to 18:

    ```shell
    minSdkVersion = 18
    ```

    - Add the following to the same file:

    ```shell
    allprojects {
        repositories {
            maven {
                // All of the Detox artifacts are provided via the npm module
                url "$rootDir/../node_modules/detox/Detox-android"
            }
        }
    }
    ```

    - Go to Android **build.gradle (app)**, and add the following under android.defaultConfig:

    ```shell
        android {
        defaultConfig {
            // Added these for running tests
            testBuildType System.getProperty('testBuildType', 'debug')
            testInstrumentationRunner 'androidx.test.runner.AndroidJUnitRunner'
        }
    }
    ```

    -Add these dependencies to the same file:

    dependencies {
        // Added testing dependencies
        androidTestImplementation('com.wix:detox:+') { transitive = true }
        androidTestImplementation 'junit:junit:4.12'
    }

    - Create a file called DetoxTest.java in the path `android/app/src/androidTest/java/com/[your_package]/`

    - Add the following to it:

    ```java
    package [your_package];

    import com.wix.detox.Detox;
    import com.wix.detox.config.DetoxConfig;

    import org.junit.Rule;
    import org.junit.Test;
    import org.junit.runner.RunWith;

    import androidx.test.ext.junit.runners.AndroidJUnit4;
    import androidx.test.filters.LargeTest;
    import androidx.test.rule.ActivityTestRule;

    @RunWith(AndroidJUnit4.class)
    @LargeTest
    public class DetoxTest {
        @Rule
        public ActivityTestRule<MainActivity> mActivityRule = new ActivityTestRule<>(MainActivity.class, false, false);

        @Test
        public void runDetoxTests() {
            DetoxConfig detoxConfig = new DetoxConfig();
            detoxConfig.idlePolicyConfig.masterTimeoutSec = 90;
            detoxConfig.idlePolicyConfig.idleResourceTimeoutSec = 60;
            detoxConfig.rnContextLoadTimeoutSec = (com.[your_package].BuildConfig.DEBUG ? 180 : 60);

            Detox.runTests(mActivityRule, detoxConfig);
        }
    }
    ```








For an iOS device or Simulator

```shell
npx react-native run-ios
```

4. Test with detox:

```shell
npm install detox --save-dev
```

- This will add the detox dependency to your React Native project’s package.json file.
