# Modshot-Core

This is a even more specialized fork of a specialized fork of [mkxp by Ancurio](https://github.com/Ancurio/mkxp) designed for [*OneShot*](http://oneshot-game.com/) for OneShot mods.

Thanks to [hunternet93](https://github.com/hunternet93) for starting the reimplementation of the journal program!

Thanks to [rkevin-arch](https://github.com/rkevin-arch) for the docker build!

> mkxp is a project that seeks to provide a fully open source implementation of the Ruby Game Scripting System (RGSS) interface used in the popular game creation software "RPG Maker XP", "RPG Maker VX" and "RPG Maker VX Ace" (trademark by Enterbrain, Inc.), with focus on Linux. The goal is to be able to run games created with the above software natively without changing a single file.
>
> It is licensed under the GNU General Public License v2+.

*OneShot* also makes use of [steamshim](https://hg.icculus.org/icculus/steamshim/) for GPL compliance while making use of Steamworks features. See LICENSE.steamshim.txt for details.

# Purpose

> Modshot makes full use of all of these and is designed to add features not added in vanilla OneShot. It adds a number of new features and aims to make modding easier, whilst adding general purpose and specialized features, such as custom window titles, discord rich presence, chroma support, and much more. With this, oneshot now reads Scripts.rxdata instead of xScripts.rxdata, meaning modders won't have to delete and rename files constantly. Feel free to make pull requests of features you would like to see.

Currently there are no features.

# Config

> Using features such as the GameJolt API, Discord Richpresence, and anything else that requires such data will need to be edited with in a config file inside the executable. This is for security purposes so nobody can steal your game/application keys. I apologize for the inconvenience. 

# Calling scripts

> Just call them with Modshot.\[script] 

# Wiki

> A wiki is in progress and will be made when more features are added.

## Building using Docker on Linux

A `Dockerfile` is attached that handles all the dependencies and allows you to build ModShot for Linux. You should already have Docker installed using your favorite package manager on Linux or [here](https://docs.docker.com/docker-for-windows/install/) on Windows.

### Building the image

To pull the image, run the following command in bash or Powershell:

```sh
docker pull rkevin/build-oneshot-linux
```

If you want to build the image manually instead, use the following (no need to do this if you pulled my image from Docker Hub):

```sh
docker build -t build-oneshot-linux .
```

### Running the image

You should have 3 directories ready, and write down their paths:
1. Source directory: This is the path to this repo (Modshot-Core or mkxp-oneshot)
2. Data directory: This is a folder that contains the `Audio`, `Data`, `Fonts`, `Graphics`, `Languages`, and `Wallpaper` directories from the game / your mod.
3. Distribution directory: This is the output folder where you want `OneShot.AppImage` to be placed after the build. On Linux, make sure that UID 1000 can write to this folder (easiest way is to `chmod 777` it, or you can ask the Docker container to run with your permissions)

Keep in mind those paths must be absolute, so `/home/user/blah` on Linux or `C:\Users\user\blah` on Windows. No relative paths allowed.

Afterwards, just run this command to build on Linux:

```sh
docker run -i -v /path/to/source:/work/src -v /path/to/data:/work/data -v /path/to/dist:/work/dist build-oneshot-linux
```

Or similarly on Windows:

```powershell
docker run -i -v C:\path\to\source:/work/src -v C:\path\to\data:/work/data -v C:\path\to\dist:/work/dist build-oneshot-linux
```

Done! Enjoy your built-from-source OneShot.

### Partial compilation

If you want to speed up compilation, you can ask the container to keep the build folder by mounting a directory to it, like `-v /path/to/build:/work/build`. This is optional.

Also note that if the journal file (`_______`) exists in the build directory, it won't be rebuilt even if you changed the source of the journal. Please delete the file manually if you want a journal rebuild.

### Unix only content

If you have game files that you only want in the Linux build of OneShot and not the Windows build, you may place them in a folder that you mount to `/work/extra_unix_content`. For example, if you want a `Map123.rxdata` that's for Unix only, put it in a folder like `unixonlyfolder/Data/Map123.rxdata`, then mount it using `-v /path/to/unixonlyfolder:/work/extra_unix_content`. This has the same structure as the regular data folder and will take precedence over any files in the regular data folder. You shouldn't need to use this, but it's an option just in case.

### Note on xScripts

For some reason, `make-appimage.sh` seems to use a `xScripts.rxdata` built by conan as the script bundled in the AppImage. Since I still don't know why this is necessary and it seems modders can just modify `Scripts.rxdata`, copy it to `xScripts.rxdata` and make it work, I have disabled this behavior.

Now the default behavior is:
1. If you already had a xScripts.rxdata in your Data folder, that will be used.
2. If that file doesn't exist, `Scripts.rxdata` in your Data folder will be copied over automatically to `xScripts.rxdata`. This means you don't have to rename the file constantly if you're making a mod.

If you want the old behavior back, add `--keep-xscripts-behavior` to the END of the `docker run` command. This also means any script you edit will have to be reflected in the `scripts` directory in this repo (source directory), which will get built into `xScripts.rxdata`.

## Building using Docker on Windows

Work in progress. Keep nagging @rkevin-arch until he figures it out, or follow the instructions below to do it on bare-metal (beware dependency hell).

## Building (Supported on Windows, Ubuntu Linux, in progress on macOS)

Preface: This only supports Visual Studio on Windows and Xcode on macOS. Ubuntu should work with either GCC or clang. You can probably compile with other platforms/setups, but beware.

With Python 3 and pip installed, install Conan via `pip3 install conan`. Afterwards, add the necessary package repositories by adding running the following commands:

```sh
conan remote add eliza "https://api.bintray.com/conan/eliza/conan"
conan remote add bincrafters "https://api.bintray.com/conan/bincrafters/public-conan"
```

Prepare to build *OneShot* by installing the necessary dependencies with Conan.

```sh
cd mkxp-oneshot
mkdir build
cd build
conan install .. --build=missing
```

Hopefully, this should complete without error. It may take quite a while to build all of the dependencies.

On Ubuntu, make sure you install the necessary dependencies before building *OneShot* proper:

```sh
sudo apt install libgtk2.0-dev libxfconf-0-dev
```

Finally, you can build the project by running the following:

```sh
conan build ..
```

On Linux, you likely want to generate an AppImage. Please refer to how to build the Journal app below, as this is a prerequisite for building the AppImage. Afterwards, you may run the command, from the root directory of the repository:

```sh
./make-appimage.sh . build /path/to/game/files /path/to/journal/_______ /some/path/OneShot.AppImage`
```

Requires [linuxdeploy](https://github.com/linuxdeploy/linuxdeploy) and [AppImageTool](https://github.com/AppImage/AppImageKit) in your `PATH`.

## Building the Journal app on Unix systems

As a prerequisite on Ubuntu, ensure that the following packages are installed.

```sh
sudo apt install python3-venv libxcb-xinerama
```

Then run the script. From the root of the repository:

```sh
./make-journal-linux.sh . /path/to/journal/parent/directory/
```

This will generate a file called `_______`.

### Supported image/audio formats
These depend on the SDL auxiliary libraries. *OneShot* only makes use of bmp/png for images and oggvorbis/wav for audio.

To run *OneShot*, you should have a graphics card capable of at least **OpenGL (ES) 2.0** with an up-to-date driver installed.

## Configuration

*OneShot* reads configuration data from the file "oneshot.conf". The format is ini-style. Do *not* use quotes around file paths (spaces won't break). Lines starting with '#' are comments. See 'oneshot.conf.sample' for a list of accepted entries.

All option entries can alternatively be specified as command line options. Any options that are not arrays (eg. preloaded scripts) specified as command line options will override entries in oneshot.conf. Note that you will have to wrap values containing spaces in quotes (unlike in oneshot.conf).

The syntax is: `--<option>=<value>`

Example: `./oneshot --gameFolder="oneshot" --vsync=true`

