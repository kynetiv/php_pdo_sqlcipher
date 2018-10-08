### Building the php_pdo_sqlcipher.dll extension 

The following steps to build the extension are somewhat problematic and error-prone due to multiple projects and environmental issues. Not to mention I'm not a Windows dev but somehow managed to get this built ;-) I'll try to layout what worked for _me_, although, YMMV.

1. A linux box to build the PHP extension from source with updated namespaces (`sqlite` to `sqlcipher`);
2. A Windows box with the necessary build tools (more on that in a bit) to build OpenSSl from source and PHP from source.
3. Extra patience

If you know your way around a Windows box, its possible you don't need to follow my steps building extension steps in Linux below, although it was easier this way for me.

#### Build the extension on Linux:

I used the provided Dockerfile to create a linux environment for the build and the targeted PHP version (currently PHP 7.1). you may use something else just as well. You can also update the dockerfile to choose which PHP version to install.

```sh
docker build -f docker/Dockerfile -t myuser/php71-sqlcipher .
```

Once built I'll run it and then ssh in.
(TODO: Automate the following into the dockerfile)

```sh 
docker run --name=sqlcipher -p="8001:80" -d myuser/php71-sqlcipher
docker exec -it sqlcipher bash
```

Now we'll use my fork of [pdo_sqlcipher](https://github.com/abbat/pdo_sqlcipher) project to create the extension files. It will essentially recognize the PHP version running in the environment and pull down necessary source PHP files for building the extension.

(Still in the above container shell)
```sh 
cd /opt
git clone https://github.com/kynetiv/pdo_sqlcipher
cd pdo_sqlcipher
git checkout windows-build
chmod +x build.sh
./build.sh
```
If the environment and the version of PHP available (may need to add new versions to config.m4), it should build a `release` folder with the applicable files for _Linux_. Although we're mostly interested in the following output files/folders:

- php source (tar file, eg php-7.1.11.tar.gz)
- 'build' directory - this is our renamed pdo_sqlcipher extension
- config.w32 (needed for building the Windows PHP extension later, also provided in this repo in the docker folder)

Copy these files and directories from docker somewhere where you can find them in your Windows machine

```bash 
docker cp sqlcipher:/opt/pdo_sqlcipher/php-7.1.11.tar.gz .
docker cp sqlcipher:/opt/pdo_sqlcipher/build/ .
docker cp sqlcipher:/opt/pdo_sqlcipher/config.w32 .
```

#### Windows Steps

I've been using a windows 7 VM to do following steps. There is a lot to setup here.

First grab a [Windows VM](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/) provided by MS. The following instructions were done on the `IE11 on Win7 (x86)` box. I also used the Virtualbox flavor.

##### Windows Build tools
Once your VM is setup and booted you'll need the following build tools / dependencies

You may need to download a newer version of .net framework as well. I used 4.6 as it worked in this Win7 VM
- [Download .Net framework v4.6](https://www.microsoft.com/en-us/download/details.aspx?id=48130)

Currently for PHP7, according to the [PHP Windows docs](https://wiki.php.net/internals/windows/stepbystepbuild), you'll need the Visual C++ 14.0 (Visual Studio 2015). I beleive a full build of VS2015 will have the build tools but the below link is _just_ the bare necessity of build tools and the Command Prompt

- [Download Visual C++ Build Tools 2015](http://download.microsoft.com/download/5/F/7/5F7ACAEB-8363-451F-9425-68A90F98B238/visualcppbuildtools_full.exe) - found here https://visualstudio.microsoft.com/vs/older-downloads/, (Redistributables and Build Tools)

For PHP5.6, you can try looking around microsoft.com, but here is a currently working link to the Visual 2012 Express for Windows Desktop

- [Download Visual Studio 2012 Express for Windows Desktop](http://download.microsoft.com/download/1/F/5/1F519CC5-0B90-4EA3-8159-33BFB97EF4D9/VS2012_WDX_ENU.iso)
 

If you don't have one, get a zip utility for some of these gzip'd source. I just used 7-zip (free) but other would be fine.
 
- [Download 7-Zip](http://www.7-zip.org/)

Also, a rar and iso tool (VS2012 iso), like WinRar (free). 

- https://www.win-rar.com/

Perl, NASM, Bison.
These are needed for building OpenSSL and/or PHP from source

- [Download Strawberry Perl](http://strawberryperl.com/download/5.26.0.1/strawberry-perl-5.26.0.1-32bit.msi)
- [Download NASM](http://www.nasm.us/pub/nasm/releasebuilds/2.13.02/win32/)
- [Download Bison](https://sourceforge.net/projects/gnuwin32/files/bison/2.4.1/bison-2.4.1-setup.exe/download?use_mirror=versaweb)

*Note* May need to copy some of these utility folders to the root `C:\ ` directory in addition to Program Files if issues with environment variables occur. (TODO how to setup paths inside Program Files).

You'll also need a recent version of OpenSSL. I've had luck on the 1.0.2 version and so grab the latest. 
OpenSSL Version updates may cause issues building from version to version. If building your own is having issue, or you don't mind using php.net's pre-built version, you can find a build version in here:

- https://windows.php.net/downloadS/php-sdk/deps/vc11/x86/

Filename like: openssl-1.0.2p-vc11-x86.zip

Otherwise download the source yourself:

- ﻿https://www.openssl.org/source/

PHP source, I'll typically copy over the php version downloaded by the pdo_sqlcipher build step above and unpack to the `C:\ ` directory.

#### Setup build directories

Because this is essentially a throw-away VM, I put everything i need in the `C:\ ` directory. Should looks something like this:

- C:\php-src # the unpacked PHP version
- C:\openssl-src # the unpacked OpenSSL version 


#### Build OpenSSL

If you do want to build from source follow these steps. Otherwise, download the prebuilt version from windows.php.net (link above).

I originally followed [this guide](http://developer.covenanteyes.com/building-openssl-for-visual-studio/) before, but the following is specific for this VM:

Open up the Windows Start Menu and find the "Native" x86 only prompt, so:
 
 Visual C++ Build Tools -> Windows Desktop Command Prompts -> Visual C++ 2015 x86 Native Build Tools Command Prompt
 *NOTE* run as administrator
 
##### Environment Variables

For OpenSSL we'll need NASM in our path

```sh 
SET PATH=%PATH%;"C:\Program Files\NASM"
```

Now build OpenSSL

```sh 
cd C:\openssl-src
perl Configure VC-WIN32 --prefix=C:\openssl
ms\do_nasm
nmake -f ms\nt.mak 
nmake -f ms\nt.mak install
```

You may see warnings, but this should complete and puts the output in `C:\openssl`
This is an error prone step so following the described environment will be helpful.

##### More Environment Variables

Now that OpenSSL is built (or you downloaded it and extracted it to `C:\openssl`), we'll update our paths for the PHP build. Adding in some additional paths for Bison

```sh 
SET PATH=%PATH%;C:\openssl\bin;%ProgramFiles%\GnuWin32\bin;
SET LIB=%LIB%;C:\openssl\lib;
SET INCLUDE=%INCLUDE%;C:\openssl\include
SET BISON_PKGDATADIR="%ProgramFiles%\GnuWin32\bin\bison"
```

*Note* May have issues with the "Program Files" `space`, so use the %aliases% above.


#### PHP Extension Build

Now place the `build` directory from pdo_sqlcipher directory (linux) earlier into the `php-src\ext` folder renaming it to `pdo_sqlcipher`. This should be alongside other core modules such as `pdo_sqlite`

Also add the `config.w32` from the pdo_sqlcipher directory (linux) into the root of the new `pdo_sqlcipher` extensions directory

On Windows, the PHP build steps use different files. We'll need to modify only a couple of these files in our extension folder:

##### php_pdo_sqlcipher_int.h

We'll need to update the include path

from: `#include <sqlcipher3.h>`

to: `﻿#include <pdo_sqlcipher\sqlcipher3.h>`

around line 24

##### sqlcipher3.c

Need to add the following flags to the top of this file:

```
﻿/******** BEGIN SQLCIPHER-WINDOWS ********/
#define SQLITE_ENABLE_COLUMN_METADATA 1
#define SQLITE_ENABLE_UNLOCK_NOTIFY 1
#define SQLITE_ENABLE_UPDATE_DELETE_LIMIT 1
#define SQLITE_HAS_CODEC 1
#define SQLITE_TEMP_STORE  2
/******** END SQLCIPHER-WINDOWS **********/

```

#### Run Build Commands

With the existing shell still open with our paths still in environment variables, enter the php-src directory:

```sh
cd C:\php-src
```

Run the `buildconf.bat` windows batch file:

```sh 
buildconf.bat
```

Next will configure which extensions and PHP-specific options to set and enable:

```sh 
configure --disable-all --enable-pdo --enable-pdo_sqlcipher --disable-zts --enable-cli
```

*Note: thread safety has not been tested, only non-thread-safe. `--enable-cli` is to bypass some issues, not really needed other than to get the build to run.

The above command should show some table output and take notice that you see `pdo_sqlcipher` listed as `shared`, meaning to be built as a dll.

```sh
Enabled extensions:
--------------------------
| Extension     | Mode   |
--------------------------
| date          | static |
| pcre          | static |
| pdo           | static |
| pdo_sqlcipher | shared |
| reflection    | static |
| spl           | static |
| standard      | static |
--------------------------
```

If you get some error from the JSRuntime, open the `configure.js` (file that buildconf generated) and inspect the line. 
I once got a cryptic error:

```bash
﻿C:\php-src\configure.js(5475, 1) Microsoft JScript runtime error: '‹¯¨' is undefined
```
about some invisible bom (‹¯¨) at the first (1) position need to be deleted, so just needed backspace to the left of:

```bash
// $Id$
```

Side note, really JavaScript builds php on windows?!?! ... moving on....

### Build it already
OK, last step!

```sh 
nmake php_pdo_sqlcipher.dll
```

You'll see plenty of warnings and this will take a moment to build. It should end with:
 
```sh 
EXT pdo_sqlcipher build complete
```

You should find the resulting `php_pdo_sqlcipher.dll` file located in:

C:\php-src\Release\php_pdo_sqlcipher.dll

Celebrate! :tada:

Otherwise... review your errors and `nmake clean` and repeat. Good Luck! :)
