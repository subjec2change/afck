Android Firmware Build System
(Android Firmware Construction Kit)
AFCK is a set of scripts used to (re-)build Android firmware. The purpose of this kit is to make tuned firmwares using the manufacturer's ready-made firmware as a basis.
At the moment, this project can be used to build firmware for the following devices:
•	Android TV box X96 Max
To (re)build the firmware, a script for the GNU Make program and a set of POSIX-compliant utilities are used. Type "make" without options to get a description of the targets that can be built by the script.
Assembly preparation
Base files
The files from which the firmware is built are not included in GIT. They are too big and they change frequently. You will have to find them on the net yourself. When you run make, you will be told the names of the missing files.
You can try running the script file ingredients/01-get-it-all. It will try to download the necessary files from the network. This will not necessarily work out. links to such resources may change frequently.
apk tool
You need to install fresh apktool from here: https://bitbucket.org/iBotPeaches/apktool/downloads/
Create an apktool script that will run java -jar apktool-версия.jar $*. If you have previously installed framework files in apktool, clear it:
apktool empty-framework-dir --force
Using the build system
The build system has a built-in help system. Running make without options will list the complete list of build targets. To initiate a build on any target, run make <purpose>. In order not to clutter up the main help screen, many additional objectives are on separate screens. To display additional help screens, type make help-<screen>, the list of available additional screens is given on the main help screen, for example, it make help-mod will list targets for applying modifications to the unpacked firmware.
The variable TARGET sets the target firmware platform, for example, TARGET=x96max/beelink sets the firmware assembly for the x96max hardware platform based on the beelink firmware. The value of the TARGET variable is either set in the top-level Makefile, or can be overridden in the local-config.mak file, which you must create yourself.
To build target files from source files, run the command make deploy. If everything goes well, you will receive the final distribution files in the out/$(TARGET)/deploy/ subdirectory.
If something does not work out, you can build the firmware step by step. First, try simply unpacking the original firmware: make img-unpack. Then we apply modifications, either all at once with the command make mod, or one at a time, using the names of targets from make help-mod. At the end, you can collect the firmware files: make ubt-img and make upd. After that, there was one small step left: make deploy and the final files are ready.
Development of own modifications
All modifications are in separate directories within the target platform subdirectory build/$(TARGET)/(eg build/x96max/beelink/). Each modification is in its own individual subdirectory.
If you are developing your own modification, you need to:
•	Create a subdirectory for modification inside the build/$(TARGET)/ directory. In principle, the name can be anything, but it is customary to make up the name of two numbers, a dash and the name of the modification. The number at the beginning of the subdirectory name helps sort the subdirectories in order of processing, so higher numbered revisions can use the variables specified by lower numbered revisions.
•	Create a file in it name-modifications.mak, in which to define the necessary rules for applying the modification. We will call the contents of this file modification rules .
Keep in mind that all files with modification rules are loaded into a single namespace. This means that if you define a variable with the same name in one file and then a variable with the same name in another modification rules file, the second assignment will override the first one and the modification may not happen the way you want it to. Therefore, if you introduce additional variables in modification rules, use the modification name as the basis for the variable name. For example, if your modification is called Boom, use variable names like BOOM_APK, BOOM_FILES and so on.
The only exception is for variables with fixed names, which specify how the modification will be applied:
•	HELP contains a description of the modification. This text is issued to the user on command make help-mod.
•	DISABLED- if this variable gets a non-empty value, the mod will be disabled. This variable can be used to temporarily disable certain mods (without having to delete or rename the .mak file).
•	INSTALL contains, in fact, a sequence of instructions for applying a modification to the unpacked image.
•	MOD contains the name of the mod
•	DIR contains the name of the mod subdirectory. Use to access additional files, such as $(DIR)/boom.apk and so on.
•	DEP Scontains a list of files the mod depends on. If you change any of these files, the build system will consider that the mod needs to be remapped. 
•	DIR By default , all files from the mod subdirectory are added to the variable ( but not from nested subdirectories ).
•	/- a variable with such a funny name is very useful, because contains the base directory where the unpacked file system images are located. In GNU Make, single-character variables need not be enclosed in parentheses: $/equivalent to $(/). Use this variable to conveniently work with unpacked image files, for example:cp $(DIR)/boom.apk $/system/app
Also, for some tasks, there are useful functions that can be called:
•	$(call IMG.UNPACK.EXT4,<partition>)will create a dependency of the current mod on the unpacked image of the specified section. 
•	For example, $(call IMG.UNPACK.EXT4,system)it will create such a dependency that before the mod starts, the section system will already be unpacked into the directory $/, so you can immediately write the rules of the form rm $/system/build.prop, etc. If you do not call this function, the existence of the subdirectory $/system is not guaranteed at the time the rules of the mod are executed, especially in multi-threaded builds.
•	$(call MOD.APK,<partition>,<file.apk>,<description>)creates a complete set of rules for installing an APK file in the app/ subdirectory of the specified partition (usually system or vendor). The simplest mod for adding a specific application might be a single line call to that function.
In principle, the above should be enough to get started. If you want to learn more about how the build system works from the inside, see the next section.
How the build system works
The work of the build system is described in more detail in the doc/HOWITWORKS.md file. If something went wrong, or you are going to develop based on afck, we recommend reading this file.

