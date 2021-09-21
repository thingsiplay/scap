# scap - SmwCentral.net AutoPatcher for Linux

Downloads and patches Super Mario World Romhacks from
[smwcentral.net](https://www.smwcentral.net/?p=section&s=smwhacks)
automatically into playable ROMs and saves them in a structured way. 

- **Author**: Tuncay D.
- **License**: [MIT License](LICENSE)
- **Source**: [Github](https://github.com/thingsiplay/scap) 

## About

**scap** is a commandline tool written in shell script for Linux.  Main purpose
is to scrape html pages from
[smwcentral.net](https://www.smwcentral.net/?p=section&s=smwhacks) projects for
Super Mario World Romhacks.  The patch file will be downloaded and the original
ROM is patched to create the new Romhack automatically.  When configured
correctly, it can also launch an emulator directly with that newly created
file.

## Usage

The script requires an URL to the hacks project html page at "smwcentral.net".
It will attempt to download the patch and create a Romhack in a new directory
named after the project.  Important: The URL must point to the html page, not
to the .zip file.  As the script is a commandline program, it needs to be run
in a terminal or within another script like `scap -u URL`. Use the help option
to see all available options with `scap -h`.

### Examples

1) Download patch file from projects page, display some meta information and
apply the patch to the unmodified ROM file.  Create a subfolder for the Romhack
in current working directory.

```bash
$ scap -mw -u "https://www.smwcentral.net/?p=section&a=details&id=16059"
```

2) Parse the URL from current clipboard with the help of an external program
called `xclip`.  Create a folder for the current day under a fixed folder in
your HOME directory and save the new Romhack there.  Finally play the Romhack
right away with your favorite emulator, look at section 
[Run Script](#run-script).
   
```bash
DIR="$HOME/smwcentral/$(date +%F)"
CLIP="$(xclip -out -selection clipboard -rmlastnl)"
$ scap -x -d "$CLIP" -u "$CLIP"
```

If you save this as a script and bind it to a keyboard shortcut, then you don't
even need the terminal to play a Romhack you just visited with your webbrowser.

## Installation

If the dependencies are met, then all you need to do is download the script and
save it in a folder that can be found in the $PATH.

1) Download and install to your current user binaries folder. (Should not
require any root rights.)

```bash
git clone "https://github.com/thingsiplay/scap"
cd scap
chdmod +x scap
cp scap "$(systemd-path user-binaries)"
```

2) Copy your unmodified ROM file "Super Mario World" to the config directory of
scap.  If you changed the default settings, then off course you have to adjust
the command too.  The ROM file needs to have the MD5 checksum:
"cdd3c8c37322978ca8669b34bc89c804"

```bash
cp "Super Mario World (U) [!].smc" "$HOME/.config/scap"
```

### Dependencies

It uses some specific features of Bash and standard Linux programs.  You might
have to install some of them explicitly:  
`rm rmdir cp mkdir grep sed cut tr readlink chmod curl unzip md5sum`

Following applications are not Linux standard programs and are required too:

- `flips` - Floating IPS: The main application that is required to patch the
  ROM with the patch file for the Romhack.  You can download it in example from
  https://dl.smwcentral.net/11474/floating.zip .  Unpack it and rename
  "flips-linux" to "flips" and put it in your $PATH or to the config directory
  at "$HOME/.config/scap".
- `xmllint` (libxml2): This is the program that is used to parse the
  downloaded html files and extract the information such as download links.
  Your distribution may have this in the repository.  If there is no standalone
  "xmllint", then look for "libxml2" package.

### Configuration

Currently, the configuration of the script is done in the script itself.  There
is no dedicated settings file.  In the beginning of the script "scap" is a
section marked for USER SETTINGS.  You can adjust them to your liking, but
there shouldn't be any need to. 

#### RUN Script 

If you use the `-x` option to run a custom command at the end of the script,
then you need to create one first.  Save a script named "run.sh" to
"$HOME/.config/scap".  It should contain the command to execute your emulator.
The first argument "`$1`" will contain the fullpath of the newly created Romhack.
Example: *$HOME/.config/scap/run.sh*

```bash
#!/bin/env bash
retroarch -L "$HOME/.config/retroarch/cores/mesen-s_libretro.so" "$1"
```

Off course you have to edit the script, if you want use a different emulator.

## Known Bugs, Limitations and Quirks

- In case the commandline option -x is used to execute the Run Script, then
  only one version of the Romhack will be launched.  This limit can be changed
  in the script, when editing the variable `max_run_limit`.  But do this with
  caution, because all of the versions might run simultaneously, as there is
  currently no wait concept implemented.

