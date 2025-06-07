# electron-chromium-codecs-osx
The patches enable AC3, EAC3 and HEVC in electron for media to play.

# Build Instructions
## 1. Requirements
- MacOS Sonoma
- XCode 15
- Python 3.8
- Node 20.9.0
- Rosetta 2

Note: Using MacOS Sequoia is possible but to get XCode 15 running on it is a bit of a task, out of scope for this repo though. 
## 2. Getting Electron
First we need to get electron to start patching it, Download depot_tools from chromium website and add it to path.
```
$ git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
$ export PATH=/path/to/depot_tools:$PATH
```

Next with the download tools in hand we start downloading electron.
```
$ mkdir electron && cd electron
$ gclient config --name "src/electron" --unmanaged https://github.com/electron/electron
$ gclient sync --with_branch_heads --with_tags
# This will take a while, go get a coffee.
```
At this step you can step away for a good while, depending on your internet speeds this can take a lot of time. It's a download of about 100GBs so I hope that you have enough space.

## 3. Preparing Electron
First we ready it for pulling from the repo to get our version of electron (29.1.4)
```
$ cd src/electron
$ git remote remove origin
$ git remote add origin https://github.com/electron/electron
$ git checkout main
$ git branch --set-upstream-to=origin/main
$ cd -
```
 Get the version 29.1.4 

```
$ cd src/electron
$ git checkout v29.1.4
# You might want to force the checkout with git checkout -f
$ gclient sync -f
```
With Electron prepared we can start applying the patches!

## 4. Applying Patches 
I assume you already downloaded the patches from this github repo so I'll be skipping over that. 
Move the patches in the required directories.
```
$ mv look_chromium_hevc_ac3.patch src/
$ mv look_electron_hevc_ac3.patch src/electron/
$ mv ffmpeg-codec.patch src/third_party/ffmpeg/
```
Apply the patches
```
$ cd src/ && git apply look_chromium_hevc_ac3.patch && cd - 
$ cd src/electron/ && git apply look_electron_hevc_ac3.patch && cd -
$ cd src/third_party/ffmpeg/ && git apply look_ffmpeg_hevc_ac3.patch && cd -
```
With our electron ready with enabled codecs, we can move on to building it.

## 5. Building Electron
```
$ cd src
$ export CHROMIUM_BUILDTOOLS_PATH=`pwd`/buildtools
$ gn gen out/Release --args="import(\"//electron/build/args/release.gn\")"
$ ninja -C out/Release electron
```
Now this step can take upto 8 hours depending on your machine, It took me 7 hours on my M2 Air. 

## 6. Packaging Electron
This step is very quick, we are just packaging all binaries into a single one and zipping it up.
```
$ ninja -C out/Release electron:electron_dist_zip
```
The final dist.zip will be in /src/out/Release.

# Credits 
Thanks [@ThaUnknown](https://github.com/ThaUnknown) for the chromium and electron patches. 
