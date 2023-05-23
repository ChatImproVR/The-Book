# Installation
This section primarly focus on installing our ChatImproVR engine.

## Minimum Requirement
As of now, the engine operates only on Windows Operating System.  

At the same time, the engine has been tested on two seperate VR headsets: Oculus VR and SteamVR. 

Therefore, if you have a different system, please refer to [this page](https://github.com/ChatImproVR/chatimprovr/issues/82) to see any updates with it. We will update this page as soon as possible if there is a new method for additional support. 

While we said that it only works on Windows operating system for the two VR headsets, there is still hardware equipment.

We cannot provide the most accurate requirement; however, the PC must able to run the VR supported Softwares. For example, the Oculus VR headset must need to connect the PC. (Trust me, I(Ken) has tested on a weak laptop, and it says that it cannot run that application to connect the VR headset). Please make sure that the VR headset supported software is able to run and connect the headset. 

## Softwares to Download
There are four softwares to download in order to use the ChatImproVR.
1. [Rust](https://www.rust-lang.org/tools/install)
2. [CMake](https://cmake.org/download/)
3. VR Supported Headset Software
   1. [Oculus VR](https://www.oculus.com/Setup/)
   2. [Steam VR](https://store.steampowered.com/app/250820/SteamVR/)
4. [Our Engine](https://github.com/ChatImproVR/chatimprovr)

We will guide you how to install for each software to be ready to use our engine. Before you go into the installaion, please open the links that are provided above.

## Rust
When you visit the page for Rust, it will greet you the screen as below.
![Rust Install Page](/images/rust_install_page.png)

Please follow the instructions as they stated. The page might look slighlty different based on operating system.

## CMake
When you visit the page for Cmake, it will greet you the screen as below. Please download the latest version of Cmake that has the option of Windows x64 Installer: the file extension is `.msi`.
![Cmake Install Page](/images/cmake_install_page.png)
In the image above, the version we are downloading is 3.26.4, but you can download and install a higher version if you want as well.

Once you download the installer, open the installer. The first screen should be similar as the following screen.

![Cmake Installer Part 1](/images/cmake_installer_part_1.png)

Select `Next` to continue the installation process.

The next screen will ask regarding the agreement of the program. Select agree to continue the installation process as the image below. You can save the agreement by selecting the `print` option if you want.

![Cmake Installer Part 2](/images/cmake_installer_part_2.png)

In the next screen, we are have an option to install on system path for current user, all users, or not at all. Choose either for the current user or all users to add the system PATH. The image below has selected for all users.

![Cmake Installer Part 3](/images/cmake_installer_part_3.png)

The next screen is selection the program path. The image below has selected the default path, but you can choose whatever path you want.

![Cmake Installer Part 4](/images/cmake_installer_part_4.png)

The next screen is an verifier on selecting all the correct options that you have selected. Once everything is all set, click the `install` button to proceed the installation.

![Cmake Installer Part 5](/images/cmake_installer_part_5.png)

Once the installation is complete, then you will see the following screen.

![Cmake Installer Part 6](/images/cmake_installer_part_6.png)

## Oculus VR
Once you open the page, you will be greet the follow page as below.

![Oculus Download Page](/images/oculus_download_page.png)

Select the `Download Oculus Rift Software` button. By selecting the button will open an `.exe`. Please follow the installer instruction.

## Steam VR
//TODO: someone test this out/write this part. I have no idea what is a major difference when it comes to Steam VR application.

## Our Engine
There are two methods to use our engine: the stable version and most advanced version.

### Stable Version
Use the Release Code Version

### Most Advanced Version
Clone the repository
