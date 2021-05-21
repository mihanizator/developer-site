# Installation

## Mac OS

You'll need homebrew installed. See installation guides [here](https://brew.sh/)

Add homebrew tap

```
$ brew tap exogress/brew
```

Install exogress client

```
$ brew install exogress
```

## Linux

### APT (Debian, Ubuntu)

```
$ curl -s https://apt.exogress.com/KEY.gpg | sudo apt-key add -
$ echo "deb https://apt.exogress.com/ /" > /etc/apt/sources.list.d/exogress.list
$ apt update
$ apt install exogress
```

## Windows

Install for [x86_64](https://github.com/exogress/cli/releases/latest/download/exogress-x86_64.msi)

Or download latest exe from [release GitHub page](https://github.com/exogress/cli/releases)

## Docker

Use docker image: [quay.io/exogress/exogress](https://quay.io/exogress/exogress)
