## Install cURL

Download the latest version of the cURL tool if it is not already installed or if you get errors running the curl commands from the documentation.

>Note

>If you’re on Windows please see the specific note on Windows extras below.

## Docker and Docker Compose

You will need the following installed on the platform on which you will be operating, or developing on (or for), Hyperledger Fabric:

* MacOSX, *nix, or Windows 10: Docker Docker version 17.03.0-ce or greater is required.
* Older versions of Windows: Docker Toolbox - again, Docker version Docker 17.03.0-ce or greater is required.

You can check the version of Docker you have installed with the following command from a terminal prompt:

    docker --version

>Note

>Installing Docker for Mac or Windows, or Docker Toolbox will also install Docker Compose. If you already had Docker installed, you should check that you have Docker Compose version 1.8 or greater installed. If not, we recommend that you install a more recent version of Docker.

You can check the version of Docker Compose you have installed with the following command from a terminal prompt:

    docker-compose --version

## Go Programming Language

Hyperledger Fabric uses the Go programming language 1.7.x for many of its components.

Given that we are writing a Go chaincode program, we need to be sure that the source code is located somewhere within the `$GOPATH` tree. First, you will need to check that you have set your `$GOPATH` environment variable.

    echo $GOPATH
    /Users/xxx/go

If nothing is displayed when you echo `$GOPATH`, you will need to set it. Typically, the value will be a directory tree child of your development workspace, if you have one, or as a child of your `$HOME` directory. Since we’ll be doing a bunch of coding in Go, you might want to add the following to your `~/.bashrc`:

    export GOPATH=$HOME/go
    export PATH=$PATH:$GOPATH/bin

## Node.js Runtime and NPM

If you will be developing applications for Hyperledger Fabric leveraging the Hyperledger Fabric SDK for Node.js, you will need to have version 6.9.x of Node.js installed.

>Node.js version 7.x is not supported at this time.

`Node.js - version 6.9.x or greater`

>Installing Node.js will also install NPM, however it is recommended that you confirm the version of NPM installed. You can upgrade the npm tool with the following command:

    npm install npm@3.10.10 -g

## Windows extras

If you are developing on Windows, you will want to work within the Docker Quickstart Terminal which provides a better alternative to the built-in Windows such as `Git Bash` which you typically get as part of installing Docker Toolbox on Windows 7.

However experience has shown this to be a poor development environment with limited functionality. It is suitable to run Docker based scenarios, such as Getting Started, but you may have difficulties with operations involving the `make` command.

Before running any `git clone` commands, run the following commands:

    git config --global core.autocrlf false
    git config --global core.longpaths true

You can check the setting of these parameters with the following commands:

    git config --get core.autocrlf
    git config --get core.longpaths

These need to be `false` and `true` respectively.

The `curl` command that comes with Git and Docker Toolbox is old and does not handle properly the redirect used in Getting Started. Make sure you install and use a newer version from the cURL downloads page

For Node.js you also need the necessary Visual Studio C++ Build Tools which are freely available and can be installed with the following command:

    npm install --global windows-build-tools

See the NPM windows-build-tools page for more details.

Once this is done, you should also install the NPM GRPC module with the following command:

    npm install --global grpc

Your environment should now be ready to go through the Getting Started samples and tutorials.

>If you have questions not addressed by this documentation, or run into issues with any of the tutorials, please visit the Still Have Questions? page for some tips on where to find additional help.
