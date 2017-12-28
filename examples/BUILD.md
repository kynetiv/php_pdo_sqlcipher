### Building the php_pdo_sqlcipher.dll extension 

The following steps to build the extension are somewhat problematic and error-proned due to multiple projects and environmental issues. Not to mention I'm not a Windows dev but somehow managed to get this built ;-) I'll try to layout what worked for _me_, although, YMMV.


1. A linux box to build the PHP extension from source with update namespaces (`sqlite` to `sqlcipher`);
2. A Windows box with the necessary build tools (more on that in a bit) to build OpenSSl from source and PHP from source.
3. Extra patience

If you know your way around a Windows box, its possible you don't need to follow my steps in `1.`, although it was easier this way for me.

#### Build the extension on Linux:

I used the provided Dockerfile to create a linux environment for the build and the targeted PHP version (currently PHP 7.1). you may use something else just as well. After built, I'll ssh into the running container to do some manual commands.

```sh
docker build -f docker/Dockerfile -t myuser/php71-sqlcipher .
```

Once built I'll run it and then ssh in.
(TODO: Automate the following into the dockerfile)

```sh 
Run and enter the container
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
If the environment and the version of PHP available (may need to add new versions to config.m4), it should build a release folder with the applicable files for _Linux_. Although we're mostly interested in the following output files/folders:

- php source (tar file, eg php-7.1.11.tar.gz)
- 'build' directory
- config.w32 (needed for building the Windows PHP extension later)

#### Windows Steps

I've been using a windows 7 VM to do following steps. There is a lot to setup here.

First grab a [Windows VM](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/) provided by MS. The following instructions were done on the `IE11 on Win7 (x86)` box. I also used the Virtualbox flavor.

##### Windows Build tools
Once your VM is setup and booted you'll need the following build tools / dependencies

You may need to download a newer version of .net framework as well. I used 4.6 as it worked in this Win7 VM
- [Download .Net framework v4.6](https://www.microsoft.com/en-us/download/details.aspx?id=48130)

Currently for PHP7, according to the [PHP Windows docs](https://wiki.php.net/internals/windows/stepbystepbuild), you'll need the Visual C++ 14.0 (Visual Studio 2015). I beleive a full build of VS2015 will have the build tools but the below link is _just_ the bare necessity of build tools and the Command Prompt

- [Download Visual C++ Build Tools 2015](http://landinghub.visualstudio.com/visual-cpp-build-tools)

If you don't have one, get a zip utility for some of these gzip'd source. I just used 7-zip but other would be fine.
 
- [Download 7-Zip](http://www.7-zip.org/)

Perl, NASM, Bison.
These are needed for building OpenSSL and/or PHP from source

- [Download Strawberry Perl](http://strawberryperl.com/download/5.26.0.1/strawberry-perl-5.26.0.1-32bit.msi)
- [Download NASM](http://www.nasm.us/pub/nasm/releasebuilds/2.13.02/win32/)
- [Download Bison](https://sourceforge.net/projects/gnuwin32/files/bison/2.4.1/bison-2.4.1-setup.exe/download?use_mirror=versaweb)

*Note* May need to copy some of these utility folders to the root `C:\ ` directory in addition to Program Files if issues with environment variables occur. (TODO how to setup paths inside Program Files).

You'll also need a recent version of OpenSSL. I've had luck on the 1.0.2 version and so grab the latest (`m` worked as of this writing, although current below is `n`).

- [OpenSSL Source](﻿https://www.openssl.org/source)

PHP source, I'll typically copy over the php version downloaded by the pdo_sqlcipher build step above and unpack to the `C:\ ` directory.

#### Setup build directories

Because this is essentially a throw-away VM, I put everything i need in the `C:\ ` directory. Should looks something like this:

- C:\php-src # the unpacked PHP version
- C:\openssl-src # the unpacked OpenSSL version 


#### Build OpenSSL

I have followed [this guide](http://developer.covenanteyes.com/building-openssl-for-visual-studio/) before, but the following is specific for this VM:

Open up the Windows Start Menu and find the x86 only prompt, so:
 
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

Now that OpenSSL is build, we'll update our paths for the PHP build. Adding in some additional paths for Bison

```sh 
SET PATH=%PATH%;C:\openssl\bin;"C:\Program Files\GnuWin32\bin"
SET LIB=%LIB%;C:\openssl\lib;
SET INCLUDE=%INCLUDE%;C:\openssl\include
SET BISON_PKGDATADIR="C:\Program Files\GnuWin32\share\bison"
```

*Note* May need to use this link to program files (issue with space)
```sh 
SET PATH=%PATH%;C:\openssl\bin;%ProgramFiles%\GnuWin32\bin;
```

#### PHP Extension Build

Now place the `build` directory from pdo_sqlcipher directory (linux) earlier into the php-src\ext folder renaming it to `pdo_sqlcipher`. This should be alongside other core modules such as `pdo_sqlite`

Also add the `config.w32` from the pdo_sqlcipher directory (linux) into the root of the new `pdo_sqlcipher` extensions directory

On Windows, the PHP build steps use different files. We'll need to modify only a couple of these:

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

The above command should show some table output and take notice that you see pdo_sqlcipher listed as `shared`, meaning to be build as a dll.

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

Last step! make it.

```sh 
nmake php_pdo_sqlcipher.dll
```

This will take a moment to build and should end with:
 
```sh 
EXT pdo_sqlcipher build complete
```

You should find the resulting php_pdo_sqlcipher file located in:

C:\php-src\Release\php_pdo_sqlcipher.dll

Celebrate!

Otherwise review your errors and `nmake clean` and repeat. Good Luck! :)
