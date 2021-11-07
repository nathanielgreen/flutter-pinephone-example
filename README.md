# Flutter Pinephone Example

An example of building a flutter app for pinephone using flatpak.

## Intro

I made this as I couldn't find a written resource on how to set this up online.
This is mostly just copied from [Fluffychat's configuration](https://github.com/flathub/im.fluffychat.Fluffychat/blob/master/im.fluffychat.Fluffychat.json) 
(see their project code [here](https://gitlab.com/famedly/fluffychat) for more
info). I've also just copied some of the steps from the [Flatpak
documentation](https://docs.flatpak.org/en/latest/first-build.html#build-the-application)
which I recommend giving a quick read.

However I've deleted some of the Fluffychat-specific parts and reworked it for a
fresh flutter app. This has been tested on Pinephone running postmarketOS
v21.06.

## Prerequisites

- An ARM64 machine to compile the flutter app on
  - I use a Raspberry Pi 4 with a RaspiOS ARM64 Image available here: https://downloads.raspberrypi.org/raspios_arm64/images/
- Flatpak & Flatpak Builder (either on your ARM64 machine or another, I did it on the Raspi)
  - Install flatpak on your machine either through package manager or other
    means e.g. `sudo apt install flatpak` if on Raspi
  - Install flatpak-builder e.g. `sudo apt install flatpak-builder`
  - Set up flathub as a flatpak remote: `flatpak remote-add --user --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo`
    - See https://flatpak.org/setup/ for OS specific instructions
  - Install the flatpak gnome runtime: `flatpak install flathub org.gnome.Platform//40 org.gnome.Sdk//40`
- Flatpak installed on your Pinephone
  - If on postmarketOS run `sudo apk add flatpak`

## Instructions

### 1. Build the app on arm64

On your ARM64 machine, run `flutter build linux` in your project.

Next you'll need to turn the linux desktop build into a tarball:
  - Navigate to the release dir: `cd build/linux/arm64/release`
  - Create a tarball of the `bundle/` directory: `tar -czvf ../../../../build.tar.gz bundle/`
  - This will have created the tarball at the root of your project. 
    **Make sure to gitignore *.tar.gz files if you don't want them in your repo**


### 2. Create your flatpak manifest file

Here's a configuration I've tweaked from Fluffychats that seems to work.

In org.flatpak.MyApp.yml

```
app-id: org.flatpak.MyApp
runtime: org.gnome.Platform
runtime-version: '40'
sdk: org.gnome.Sdk
command: flutter_pinephone_example
separate-locales: false
finish-args:
  - --share=ipc
  - --socket=x11
  - --socket=fallback-x11
  - --socket=wayland
  - --socket=pulseaudio
  - --share=network
  - --device=all
  - --filesystem=xdg-download
  - --talk-name=org.freedesktop.Notifications
  - --own-name=org.flatpak.flutter_pinephone_example
modules:
  - name: libjsoncpp
    buildsystem: cmake
    config_opts:
      - -DCMAKE_BUILD_TYPE=release
      - -DBUILD_SHARED_LIBS=ON
    sources:
      - type: archive
        url: https://github.com/open-source-parsers/jsoncpp/archive/refs/tags/1.7.5.tar.gz
        sha256: 4338c6cab8af8dee6cdfd54e6218bd0533785f552c6162bb083f8dd28bf8fbbe
  - name: flutter_pinephone_example
    buildsystem: simple
    only-arches:
      - aarch64
    build-commands:
      - ls flutter_pinephone_example
      - cp -r flutter_pinephone_example /app/flutter_pinephone_example
      - chmod +x /app/flutter_pinephone_example/flutter_pinephone_example
      - mkdir /app/bin
      - ln -s /app/flutter_pinephone_example/flutter_pinephone_example /app/bin/flutter_pinephone_example
    sources:
      - type: archive
        path: ./file.tar.gz
        dest: flutter_pinephone_example
```

Check the build works via `flatpak-builder build-dir org.flatpak.MyApp.yml`
which will build it into the `build-dir`

### 3. Build the flatpak repository

Run `flatpak-builder --repo=repo --force-clean build-dir org.flatpak.MyApp.yml`
which will build a flatpak repository at `repo/` which we can install on the pinephone.

### 4. Copy the repository to your pinephone

You can do this via Rsync or a USB or uploading the repo somewhere and
downloading it onto your phone.

### 5. Install the repository on your pinephone

With the repo on the pinephone, in the same directory as repo, install it to `tutorial-repo` via: 
`flatpak --user remote-add --no-gpg-verify tutorial-repo repo`

Install the app: `flatpak --user install tutorial-repo org.flatpak.MyApp`

### 6. Set GL Env and run your app

On the pinephone to run the app run the following: `env GDK_GL=gles
flutter_pinephone_example` and your app should load.


# TODO

- [ ] Write up how to install app drawer/desktop icon
  - See [fluffychats setup](https://github.com/flathub/im.fluffychat.Fluffychat/blob/master/im.fluffychat.Fluffychat.desktop) 
    if you want to try this yourself 






