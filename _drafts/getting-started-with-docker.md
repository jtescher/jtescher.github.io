---
layout: post
title:  "Getting Started With Docker"
date:   1970-01-01 00:00:00
categories:
---

```shell
$ brew install caskroom/cask/brew-cask

  ==> Tapping caskroom/cask
  Cloning into '/usr/local/Library/Taps/caskroom/homebrew-cask'...
  remote: Counting objects: 2946, done.
  remote: Compressing objects: 100% (2837/2837), done.
  remote: Total 2946 (delta 128), reused 755 (delta 93), pack-reused 0
  Receiving objects: 100% (2946/2946), 5.72 MiB | 4.16 MiB/s, done.
  Resolving deltas: 100% (128/128), done.
  Checking connectivity... done.
  Tapped 1 formula (2925 files, 23M)
  ==> Installing brew-cask from caskroom/homebrew-cask
  ==> Cloning https://github.com/caskroom/homebrew-cask.git
  Cloning into '/Library/Caches/Homebrew/brew-cask--git'...
  remote: Counting objects: 2902, done.
  remote: Compressing objects: 100% (2786/2786), done.
  remote: Total 2902 (delta 133), reused 747 (delta 100), pack-reused 0
  Receiving objects: 100% (2902/2902), 5.69 MiB | 7.84 MiB/s, done.
  Resolving deltas: 100% (133/133), done.
  Checking connectivity... done.
  Note: checking out 'a952b872918e94314878891482f7b8876c4a99cf'.

  You are in 'detached HEAD' state. You can look around, make experimental
  changes and commit them, and you can discard any commits you make in this
  state without impacting any branches by performing another checkout.

  If you want to create a new branch to retain commits you create, you may
  do so (now or later) by using -b with the checkout command again. Example:

    git checkout -b <new-branch-name>

  ==> Checking out tag v0.54.0
  🍺  /usr/local/Cellar/brew-cask/0.54.0: 2634 files, 10M, built in 5 seconds
```

```shell
$ brew caskk install virtualbox

  ==> We need to make Caskroom for the first time at /opt/homebrew-cask/Caskroom
  ==> We'll set permissions properly so we won't need sudo in the future
  Password:
  ==> Downloading http://download.virtualbox.org/virtualbox/4.3.28/VirtualBox-4.3.28-100309-OSX.dmg
  ######################################################################## 100.0%
  ==> Running installer for virtualbox; your password may be necessary.
  ==> Package installers may write to any location; options such as --appdir are ignored.
  ==> installer: Package name is Oracle VM VirtualBox
  ==> installer: Installing at base path /
  ==> installer: The install was successful.
  ==> Symlinking Binary 'VBoxHeadless' to '/usr/local/bin/VBoxHeadless'
  ==> Symlinking Binary 'VBoxManage' to '/usr/local/bin/VBoxManage'
  🍺  virtualbox staged at '/opt/homebrew-cask/Caskroom/virtualbox/4.3.28-100309' (4 files, 110M)
```

```shell
$ boot2docker init

  Latest release for github.com/boot2docker/boot2docker is v1.6.2
  Downloading boot2docker ISO image...
  Success: downloaded https://github.com/boot2docker/boot2docker/releases/download/v1.6.2/boot2docker.iso
    to /Users/julian/.boot2docker/boot2docker.iso
  Generating public/private rsa key pair.
  Your identification has been saved in /Users/julian/.ssh/id_boot2docker.
  Your public key has been saved in /Users/julian/.ssh/id_boot2docker.pub.
  The key fingerprint is:
  93:9b:d9:c0:36:d0:f8:9b:4f:a7:e9:ef:20:bc:b7:c3 julian@Julians-MacBook-Pro-2.local
  The key's randomart image is:
  +--[ RSA 2048]----+
  |                 |
  |       o         |
  |      o .        |
  |       + .       |
  |        S        |
  |       o @       |
  |        O.+ .    |
  |         =E=     |
  |        .o*=o    |
  +-----------------+
```

```shell
$ boot2docker up

  Waiting for VM and Docker daemon to start...
  Writing /Users/julian/.boot2docker/certs/boot2docker-vm/ca.pem
  Writing /Users/julian/.boot2docker/certs/boot2docker-vm/cert.pem
  Writing /Users/julian/.boot2docker/certs/boot2docker-vm/key.pem

  To connect the Docker client to the Docker daemon, please set:
      export DOCKER_HOST=tcp://192.168.59.103:2376
      export DOCKER_CERT_PATH=/Users/julian/.boot2docker/certs/boot2docker-vm
      export DOCKER_TLS_VERIFY=1
```
