# Installation of Visual Studio 2017 on a Personal Computer

If you wish to install Visual Studio 2017 with CUDA tools support then you must install an early version and not the publicly available version from the Microsoft website. CICS have made the 15.2 version of Visual Studio 2017 Community (matching the version installed in the Diamond high spec compute room) but this requires some specific actions to install. Alternatively you can install the latest version of Visual Studio 2015 which is officially supported by NVIDIA for CUDA.

## Removal of Visual Studio 2017 (greater than 15.2)

To install Visual Studio 2017 Community (15.2) first remove any installation of Visual Studio 2017 later than 15.2. You can check you Visual Studio 2017 version from the Help->About menu. See Image below.

If your version is later than 15.2 then you should use the Windows Add Remove Programs feature to remove Visual Studio 2017 and Visual Studio Installer.

![Visual Studio Version](../../../assets/images/VisualStudioVersion.png "Visual Studio Version")

## Installation of Visual Studio 2017

The Installer for Visual Studio 2017 (15.2) is available from campus share (or via VPN) at

`\\uosfstore\shared\student\ac_share\software\MicrosoftVisualStudioCommunity2017.2`

You should copy the link into a local file browser. You will need to authenticate using your CICS login. You may also need to use the workgroup `SHEFUNIAD` i.e. your username should be in the format `SHEFUNIAD\ab1xyz`.

Before installation you should read the `READ_ME` file which will instruct you to install a number of certificates to prevent the installer form upgrading the Visual Studio version. The easiest way to do this is to navigate to the `Certificates` directory and double click each certificate following the prompts to install. Once all three certificates are installed then run the installer and proceed as normal.

## Installation of CUDA 9.2

The CUDA installation should not be attempted before the installation of Visual Studio version as the CUDA installer will check for Visual Studio installations in order to install CUDA support.

CUDA 9.2 can be downloaded and installed form the [NVIDIA website](https://developer.nvidia.com/cuda-downloads).
