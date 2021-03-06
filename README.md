# Music organisation and streaming system

Tchaik is an open source music organisation and streaming system.  The backend is written in [Go](http://golang.org), the frontend is built using [React](https://facebook.github.io/react/) and [Flux](https://facebook.github.io/flux/).

![Tchaik UI](https://s3-ap-southeast-2.amazonaws.com/dhowden-pictures/tchaik-may.jpg "Tchaik UI")

# Features

* Automatic prefix grouping and enumeration detection (ideal for classical music: properly groups big works together).
* Multiplatform web-based UI and REST-like API for controlling player.
* Multiple storage and caching options: Amazon S3, local and remote file stores.
* Import music library from iTunes or build from directory-tree of audio tracks.

# Requirements

* [Go 1.4+](http://golang.org/dl/) (recent changes have only been tested on 1.4.2).
* [NodeJS](https://nodejs.org/), [NPM](https://www.npmjs.com/) and [Gulp](http://gulpjs.com/) installed globally (for building the UI).
* Recent version of Chrome (Firefox may also work, though hasn't been fully tested).

# Building

If you haven't setup Go before, you need to first set a `GOPATH` (see [https://golang.org/doc/code.html#GOPATH](https://golang.org/doc/code.html#GOPATH)).

To fetch and build the code for Tchaik:

    $ go get github.com/tchaik/tchaik/...

This will fetch the code and build the command line tools into `$GOPATH/bin` (assumed to be in your `PATH` already).

Building the UI:

    $ cd $GOPATH/src/github.com/tchaik/tchaik/cmd/tchaik/ui
    $ npm install
    $ gulp

Alternatively, if you want the JS and CSS to be recompiled and have the browser refreshed as you change the source files:

    $ WS_URL="ws://localhost:8080/socket" gulp serve

Then browse to `http://localhost:3000/` to use tchaik.

# Starting the UI

To start Tchaik you first need to move into the `cmd/tchaik` directory:

    $ cd $GOPATH/src/github.com/tchaik/tchaik/cmd/tchaik

## Importing an iTunes Library

If you have an iTunes Library then you can use this to build a Tchaik library on-the-fly and start the UI in one command:

    $ tchaik -itlXML ~/path/to/iTunesLibrary.xml

You can also convert the iTunes Library into a Tchaik library using the `tchimport` tool, and use this instead:

    $ tchimport -itlXML ~/path/to/iTunesLibrary.xml -out lib.tch
    $ tchaik -lib lib.tch

NB: An Tchaik library will generally be smaller than its corresponding iTunes Library.  Tchaik libraries are stored as gzipped-JSON (rather than Apple plist) and contain a subset of the metadata used by iTunes.

## Importing Audio Files

Alternatively you can build a Tchaik library from a directory-tree of audio files, though only files with supported metadata (see [github.com/dhowden/tag](https://github.com/dhowden/tag)) will be imported:

    $ tchimport -path /all/my/music -out lib.tch
    $ tchaik -lib lib.tch

# More Advanced Options

A full list of command line options is available from the `--help` flag:

    $ tchaik --help
    Usage of tchaik:
      -add-path-prefix="": add prefix to every path
      -artwork-cache="": path to local artwork cache (content addressable)
      -auth=false: use basic HTTP authentication
      -debug=false: print debugging information
      -itlXML="": path to iTunes Library XML file
      -lib="": path to Tchaik library file
      -listen="localhost:8080": bind address to http listen
      -local-store="/": local media store, full local path /path/to/root
      -media-cache="": path to local media cache
      -remote-store="": remote media store, tchstore server address <hostname>:<port>, or s3://<bucket>/path/to/root for S3
      -static-dir="ui/static": Path to the static asset directory
      -tls-cert="": path to a certificate file, must also specify -tls-key
      -tls-key="": path to a certificate key file, must also specify -tls-cert
      -trim-path-prefix="": remove prefix from every path

### -local-store

Set `-local-store` to the local path that contains your media files.  You can use `trim-path-prefix` and `add-path-prefix` to rewrite paths used in the Tchaik library so that file locations can still be correctly resolved.

### -remote-store

Set `-remote-store` to the URI of a running [tchstore](http://godoc.org/github.com/tchaik/tchaik/cmd/tchstore) server  (`hostname:port`).  Instead, S3 paths can be used: `s3://<bucket>/path/to/root`.  Set the environment variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` to pass credentials to the S3 client.

### -media-cache

Set `-media-cache` to cache all files loaded from `-remote-store` (or `-local-store` if set).

### -artwork-cache

Set `-artwork-cache` to create/use a content addressable filesystem for track artwork.  An index file will be created in the path on first use.  The folder should initially be empty to ensure that no other files interfere with the system.
