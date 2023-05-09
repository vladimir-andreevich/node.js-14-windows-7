# Node.js 14 for Windows 7

This repository contains Node.js 14 installers for Windows 7, as well as the source code of the Node.js runtime environment which is adapted for Windows 7 with instructions on how to build it.

## Introduction

Node.js is a popular back-end JavaScript runtime environment which executes JavaScript code outside a web browser. Node.js has been continuously evolving since its inception. However, one significant change which recently occurred in Node.js was the drop of support for Windows 7. After version 13.6, [Node community stopped providing Windows 7 compatibility](https://github.com/nodejs/node/pull/31954) right in the middle of the development phase even without leading Node.js to its logical conclusion for this operating system in the form of the v14 LTS release.

## Theoretical background

Since version 14, Node.js uses ```GetHostNameW``` function from Windows Sockets API to retrieve the host name for the local computer in the Unicode string. Since ```GetHostNameW``` is available starting from Windows 8, Node.js complains on Windows 7 that it cannot find the entry point in ```GetHostNameW``` in the dynamic link library ```WS2_32.dil```.

The solution to this problem is the following: instead of asking for the function from Windows 7, we can directly provide ```GetHostNameW``` to the runtime environment while building it. I re-implement ```GetHostNameW``` function such that the conversion from the ASCII string into the Unicode string is done manually and include the custom implementation in Node.js in such a way that it comes on top of the original ```winsock2.h``` library.

## Building

1. Install [Visual Studio IDE](https://visualstudio.microsoft.com/), release 2017 or 2019. The edition year is important since Node.js 14 refuses to build on different Visual Studio versions. Visual Studio is needed to perform a build.
2. Use Visual Studio Installer to install English language pack and the following development tools from the "Desktop development with C++" workload:
* C++ build tools MSVC v140 for Visual Studio 2015
* C++ build tools MSVC v141 for Visual Studio 2017 or C++ build tools MSVC v142 for Visual Studio 2019 depending on which Visual Studio edition year is installed
* Windows 10 SDK (10.0.19041.0)
* Just-In-Time debugger
* C++ profiling tools
* C++ CMake tools for Windows
* C++ ATL library for the latest v142 build tools (x86 and x64)
* C++ AddressSanitizer
3. Install [Python 3.8](https://www.python.org/downloads/release/python-3810/).
4. Install [NetWide Assembler 2.15.05](https://www.nasm.us/pub/nasm/releasebuilds/2.15.05/) for OpenSSL assembler modules. If not installed in the default location, it needs to be manually added to `PATH`.
5. Basic Unix tools are required for some tests. Install [Git for Windows](https://git-scm.com/download/win) which includes Git Bash and tools which can be included in the global `PATH`.
6. Install [WiX Toolset v3.11](https://wixtoolset.org/docs/wix3/) and 
[Wix Toolset Visual Studio 2017 Extension](https://marketplace.visualstudio.com/items?itemName=RobMensching.WixToolsetVisualStudio2017Extension) or 
[Wix Toolset Visual Studio 2019 Extension](https://marketplace.visualstudio.com/items?itemName=WixToolset.WixToolsetVisualStudio2019Extension) depending on which Visual Studio edition year is installed.
7. Download the source code from this repository as ZIP. I do not recommend using Git. In case some issue happens, this repository may be force pushed. Unpack to a path that does not contain spaces.
8. In the terminal, go to the folder with the Node.js source code and run ```vcbuild.bat release x64 msi && vcbuild.bat release x86 msi```. Although there is a Visual Studio solution in the source code, do not try to build Node.js installers using Visual Studio, otherwise, you get an error saying that ```wcautil.h``` is missing. 
9. After compilation, Node.js installers are placed in the same directory as ```vcbuild.bat```, and the embeddable packages are put in the ```Release``` folder. Enjoy problem solving with Node.js!

## Where compatibility problems may arise

The core Node.js 14 interpreter and the standard Node.js 14 libraries run correctly on Windows 7. Nothing is cut or modified in Node.js itself, and the interpreter in these installers reads the code in the same exact way as the interpreter in the official installers. However, in the future, you may experience compatibility issues with ```npm``` packages. The reason is that developers of these packages may drop support for Windows 7 considering that officially Node.js 14 is not intended for this operating system. I cannot provide any help with such kind of troubles because I have no control over thousands of ```npm``` software packages. In this case, I recommend contacting the developer of the specific package which causes the compatibility issue.
