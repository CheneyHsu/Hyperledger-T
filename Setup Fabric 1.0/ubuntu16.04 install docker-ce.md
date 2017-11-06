## Install packages to allow apt to use a repository over HTTPS

        root@Ubuntu16:~# apt-get install apt-transport-https ca-certificates curl software-properties-common

## Add Docker’s official GPG key

        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - 
        sudo apt-key fingerprint 0EBFCD88

## Set up the stable repository
        sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

## Update the apt package index
        sudo apt-get update

## Install the latest version of Docker CE
        sudo apt-get install docker-ce