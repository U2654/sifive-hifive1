How to build bare metal applications on HiFive1-Board
=====================================================

1) Prerequisites on ubuntu (see *https://github.com/sifive/freedom-e-sdk*):

	`sudo apt-get install autoconf automake libmpc-dev libmpfr-dev libgmp-dev gawk bison flex texinfo libtool libusb-1.0-0-dev make g++ pkg-config libexpat1-dev zlib1g-dev`

2) Clone and update the *sifive--hifive1* repository:

	```bash
	git clone git@gitlab.informatik.uni-bremen.de:ppieper/sifive--hifive1.git
	cd sifive--hifive1

	cd freedom-e-sdk
	git submodule update --init --recursive        # may take some time
	```

3) Download and unpack the pre-build *riscv-multilib* toolchain available from:

	http://raws13/gnu-toolchain_riscv-multilib/latest-gnu-toolchain_riscv-multilib.tar.gz


4) make sure that the env variable RISCV_PATH is pointing to your toolchain-dir (without `/bin`)

	example: `export RISCV_PATH="/opt/riscv"`

5) Build openocd (to load a program on the board) in *sifive--hifive1/freedom-e-sdk/*:

	`make openocd						# need only to be done once`


6) Build and upload a program to the board:

	```bash
	make software PROGRAM=hello	BOARD=freedom-e300-hifive1		# replace hello with other programs accordingly (e.g. *fade_led*, etc., see *software* folder)
	sudo make upload PROGRAM=hello		# may not be necessary to use _sudo_, see the *sifive--hifive1/doc/hifive1-getting-started-v1.0.2.pdf* on additional details
	```

7) Show program output:

	`screen /dev/ttyUSB1 115200`

In case screen does not work use (this may happen when running screen a second time):

	`cat /dev/ttyUSB1	# NOTE: need to run screen once before this command works`

Press the reset button on the board to restart (and thus see output).
NOTE: /dev/ttyUSB0 is the debug interface.


Disable compressed instructions:
--------------------------------

In *sifive--hifive1/freedom-e-sdk/bsp/env/freedom-e300-hifive1/* open *setting.mk* and set *RISCV_ARCH := rv32im* (set to rv32imac to re-enable). NOTE: *RISCV_ARCH := rv32ima* might also be possible, however the RISC-V multilib toolchain seems to not support it for now!?


ZEPHYR OS
=========

To use Zephyr, see zephyr/RREADME.rst. (Install OS Packages, pip packages)
To build for HiFive1, use example zephyrrc file, modify for your paths, rename to ~/.zephyrrc and source it(?). Source zephyr-env.sh. Then build your project:

```bash
mkdir build && cmake .. -DBOARD=$BOARD
make -j6
```


Uploading Zephyr-Elfs to board
------------------------------

Basically the same procedure as freedom-e-sdk, but manual.

```bash
$ ${SDK_PATH}/work/build/openocd/prefix/bin/openocd -f ${SDK_PATH}/bsp/env/freedom-e300-hifive1/openocd.cfg &
$ riscv32-unknown-elf-gdb
 set remotetimeout 240
 target extended-remote localhost:3333
 monitor reset halt
 monitor flash protect 0 64 last off
 load zephyr/zephyr.elf
 monitor resume
```

Other Doc
=========

FreeRTOS example uses 14.7kiB out of 16kiB RAM (with 2k stack):

	size RISCV_HiFive1_GCC.elf -d
	   text	   data	    bss	    dec	    hex	filename
	  17079	   1076	  13953	  32108	   7d6c	RISCV_HiFive1_GCC.elf

Zephyr uses with the philosophers demo:

	Memory region         Used Size  Region Size  %age Used
		         ROM:       22960 B        12 MB      0.18%
		         RAM:        9440 B        16 KB     57.62%
		    IDT_LIST:         553 B         2 KB     27.00%