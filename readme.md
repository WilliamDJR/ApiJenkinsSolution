# APIs Jenkins Build Pipeline Solution

## poi

This is a DotNet Core application, the framework version is .netcore 3.1. You need to install the SDK on your Jenkins node.

Download link: https://docs.microsoft.com/en-us/dotnet/core/install/linux-ubuntu

First, you need to run the following commands to add the Microsoft package signing key to your list of trusted keys and add the package repository.

```sh
wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
rm packages-microsoft-prod.deb
```

> NOTE
>
> You need to specify the ubuntu version of your Jenkins.

Then, the .NET SDK allows you to develop apps with .NET. If you install the .NET SDK, you don't need to install the corresponding runtime. To install the .NET SDK, run the following commands:

```sh
sudo apt-get update; \
  sudo apt-get install -y apt-transport-https && \
  sudo apt-get update && \
  sudo apt-get install -y dotnet-sdk-6.0
```

> NOTE
>
> Please note, you need to have the v3.1 to build this application.

### Install DotNet SDK 3.1

Open a terminal and run the following commands to setup the 20.04 repositories

```bash
 sudo wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
 sudo dpkg -i packages-microsoft-prod.deb
```

Install the .NET Core SDK

```bash
 sudo add-apt-repository universe
 sudo apt-get update
 sudo apt-get install apt-transport-https
 sudo apt-get update
 sudo apt-get install dotnet-sdk-3.1
```

**Build Pipeline**

```sh
pipeline {
    agent any

    stages {
        stage('Git checkout') {
            steps{
                // Get source code from a GitHub repository
                git branch:'main', url:'https://github.com/RayMaAU/openhack-devops-team.git'
            }
        }
        
        stage('Build') {
            steps{
                sh 'dotnet restore ./apis/poi/poi.sln'
                sh 'dotnet clean ./apis/poi/poi.sln --configuration Release'
                sh 'dotnet build ./apis/poi/poi.sln'
            }
        }
        
        stage('Test') {
            steps{
                sh 'dotnet test ./apis/poi/tests/UnitTests/UnitTests.csproj  --configuration Release --no-restore'
                sh 'dotnet test ./apis/poi/tests/IntegrationTests/IntegrationTests.csproj  --configuration Release --no-restore'
            }
        }
        
        stage('Publish') {
            steps{
                sh 'dotnet publish ./apis/poi/web/poi.csproj --configuration Release --no-restore'
                sh 'ls -la ./apis/poi/web/bin/Release/netcoreapp3.1/publish/'
            }
        }
    }
}
```

## user-java

As a Maven Java application it uses Spring framework, this is all you need:

```sh
sudo apt install maven
```

The latest v3.3.9 should work for this application.

**Build Pipeline**

```sh
pipeline {
    agent any

    stages {
        stage('Git checkout') {
            steps{
                // Get source code from a GitHub repository
                git branch:'main', url:'https://github.com/RayMaAU/openhack-devops-team.git'
            }
        }
        
        stage('Build') {
            steps{
                sh 'mvn -f ./apis/user-java/pom.xml package'
            }
        }
        
        stage('Test') {
            steps{
                dir('./apis/user-java/') {
                  sh 'mvn test'
                }
            }
        }
        
        stage('Publish') {
            steps{
                sh 'ls -la ./apis/user-java/target/'
            }
        }
    }
}
```

## trips

Obviously, this is a Go application, so you need to install Golang SDK. You can read details from https://go.dev/doc/install.

Install it by running:

```sh
sudo wget https://go.dev/dl/go1.17.6.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.17.6.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
```

Verify that you've installed Go by opening a command prompt and typing the following command:

```sh
go version
```

To run tests, you also need some essential build tools, including GCC. Install by this command:

```sh
sudo apt install build-essential
```

**Build Pipeline**

```sh
pipeline {
    agent any

    stages {
        stage('Git checkout') {
            steps{
                // Get source code from a GitHub repository
                git branch:'main', url:'https://github.com/RayMaAU/openhack-devops-team.git'
            }
        }
        
        stage('Get') {
            steps{
                dir("./apis/trips/") {
                    sh '/usr/local/go/bin/go get -d'
                }
            }
        }
        
        stage('Build') {
            steps{
                dir("./apis/trips/") {
                    sh '/usr/local/go/bin/go build -o main .'
                }
            }
        }
        
        stage('Tests') {
            steps{
                dir("./apis/trips/") {
                    sh '/usr/local/go/bin/go test ./tripsgo -run Unit'
                    sh '/usr/local/go/bin/go test ./tripsgo'
                }
            }
        }
        
        stage('Publish') {
            steps{
                sh 'ls -la ./apis/trips/'
            }
        }
    }
}
```

## next-with-jest

If you've installed NodeJS via [nvm](https://nodejs.org/en/download), soft link the `node` `npm` `npx` to system PATH

```sh
sudo rm /usr/local/bin/npm /usr/local/bin/npx /usr/local/bin/node
sudo ln -s $(which npm)  /usr/local/bin/npm
sudo ln -s $(which npx)  /usr/local/bin/npx
sudo ln -s $(which node)  /usr/local/bin/node
```

**Build Pipeline**

```sh
pipeline {
    agent any

    stages {
        stage('Git checkout') {
            steps{
                git branch:'master', url:'https://github.com/WilliamDJR/next-with-jest.git'
            }
        }
        
        stage('npm install') {
            steps{
                    sh '''
                    ls
                    npm install'''
            }
        }
        
        stage('Tests') {
            steps{
                    sh 'npm test'
            }
        }
        
        stage('Publish') {
            steps{
                sh 'ls -la .'
            }
        }
    }
    post {
      always {
        cleanWs()
      }
    }
}
```
