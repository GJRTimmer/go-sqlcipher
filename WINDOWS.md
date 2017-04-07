After some work hereby the manual for succesfull compilation on Windows 64 bit. ( Also for dummies )

OS: Windows 10
Arch: 64bit

This tutorial is based upon my own setup which means: I install development related programs within ```C:\Development```

Also this tutorial assumes you have Go 1.8 64Bit setup correctly. 
(By correctly I assume you have a GOPATH variable setup which points to a single directory)

<hr>

# 1) Setup

1.1 Install Perl64bit
Go to [ActiveState](http://www.activestate.com/activeperl/downloads) and download the latest Perl64, Windows Installer.
Install this to: ```C:\Development\Perl64``` and make sure you check the checkbox within the installer for ```Add to Path```.

1.2 Install TDM-GCC-64
Go to [TDM-GCC-64](http://tdm-gcc.tdragon.net/download) and download either the webdl or the tdm64 installer.
Install this to: ```C:\Development\TDM-GCC-64```

1.3 Install MSYS
Go to [http://www.mingw.org/wiki/msys] and download the 1.0.11 MSYS.
Direct Download: [http://downloads.sourceforge.net/mingw/MSYS-1.0.11.exe](http://downloads.sourceforge.net/mingw/MSYS-1.0.11.exe)
Install this to ```C:\Development\MSYS```

During the post install it will ask for the MINGW location: enter the following: ```C:\Development\TDM-GCC-64``` The post install will detect several files and if all oke it will also gives you the message that it can not find ```make``` within ```TDM-GCC-64``` and that you should keep it like this.
MSYS will be the one to provide you with the ```make``` command.

So far so good, let's review, we have perl64, MSYS to provide make and the post install of MSYS also created a mount point within it self ```/mingw``` which points to ```C:\Development\TDM-GCC-64```

So to verify the setup => Start Menu => Start the MSYS terminal
Run ```perl -v``` it gives you the perl version information
Run ```gcc -v``` it gives you the gcc version information
Run ```make -v``` it gives you the make version information

# 2) Compile OpenSSL 64Bit

2.1 Prepare OpenSSL
Create the following folder path with windows Explorer: ```C:\Development\OpenSSL\src```

2.2 Download OpenSSL Source
Go to [https://www.openssl.org/source/](https://www.openssl.org/source/)

Download the latest openssl-1.0.x[a-z].tar.gz
For this tutorial I've used: [https://www.openssl.org/source/openssl-1.0.2k.tar.gz](https://www.openssl.org/source/openssl-1.0.2k.tar.gz)
For the purpose of this tutorial I will continue to refere to ```openssl-1.0.2k``` for folder and files location(s).

Copy ```openssl-1.0.2k.tar.gz``` => ```C:\Development\OpenSSL\src```

**Open MSYS Terminal (Start Menu => MSYS)**
$ ```cd /c/Development/OpenSSL/src```
$ ```tar -xvzf openssl-1.0.2k.tar.gz```
$ ```cd openssl-1.0.2k```

Configure OpenSSL
$ ```perl configure mingw64 no-shared no-asm```

Build OpenSSL
$ ```make```

Building OpenSSL only takes a few minutes on a Intel I7.

# 3) Copy OpenSSL Resources 
Copy OpenSSL resources to TDM-GCC-64 for go-sqlcipher compilation (Windows Explorer)

Within the folder ```C:\Development\OpenSSL\src\openssl-1.0.2k``` you will find 2 files which needs to be copied.
* libcrypto.a
* libcrypto.pc

Copy these files to ```C:\Development\TDM-GCC-64\lib```

Now copy the entire OpenSSL include to TDM-GCC-64.
Goto ```C:\Development\OpenSSL\src\openssl-1.0.2k\include``` in this folder you will find a single folder named ```openssl``` copy this folder to ```C:\Development\TDM-GCC-64\x86_64-w64-mingw32\include```

Copy the entire folder not contents.

# 4) Build go-sqlcipher
Now we are ready to build go-sqlcipher, we are using the still opened MSYS terminal from step 2 by the way.

Because we need a single change in a file for succesful compilation, we need to do the following.

Check out go-sqlcipher
$ ```go get -u -v github.com/xeodou/go-sqlcipher```

This will also start a build but this will fail; don't worry it's oke.

Now edit the following file: ```<GOPATH>\src\github.com\xeodou\go-sqlcipher\sqlite3_windows.go``` 
And add the following flag to the LDFLAGS ```-lgdi32```

Complete new contents for ```sqlite3_windows.go```

```go
// Copyright (C) 2014 Yasuhiro Matsumoto <mattn.jp@gmail.com>.
//
// Use of this source code is governed by an MIT-style
// license that can be found in the LICENSE file.
// +build windows

package sqlite3

/*
#cgo CFLAGS: -I. -fno-stack-check -fno-stack-protector -mno-stack-arg-probe
#cgo windows,386 CFLAGS: -D_USE_32BIT_TIME_T
#cgo LDFLAGS: -lmingwex -lmingw32 -lgdi32
*/
import "C"
```

now we can build succesfuly:
$ ```cd $GOPATH\src\github.com\xeodou\go-sqlcipher```
$ ```go install -v .```
