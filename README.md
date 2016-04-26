## Installation

Clone the project 
    git clone bla bla
    cd NewProject/
    rm -rf .git/
    
    sed -i.tmp s/#ADMIN_USER#/root/g env/admin.env
    sed -i.tmp s/#ADMIN_PASSWORD#/shiny-new-password/g env/admin.env
    sed -i.tmp s/#PROJECT_NAME#/NewProject/g dockerfiles/swift/autoreload.sh dockerfiles/swift/Dockerfile Package.swift
    find . -type f -name \*.tmp -exec rm -f {} \;
    
    docker-compose -f dev.yml build
    
    docker-compose -f dev.yml run swift swift build --generate-xcodeproj .
    
    open NewProject.xcodeproj
    
    docker-compose -f dev.yml up
    
The first start takes some time because the swift package manager has to install and fetch all
dependencies first.
    
```
swift_1          | Compiling Swift Module 'LoggerAPI' (1 sources)
swift_1          | Compiling Swift Module 'KituraTemplateEngine' (1 sources)
swift_1          | Compiling Swift Module 'Socket' (3 sources)
swift_1          | Compiling Swift Module 'SwiftyJSON' (2 sources)
swift_1          | Compiling Swift Module 'BinaryJSON' (9 sources)
swift_1          | Compiling Swift Module 'KituraSys' (4 sources)
swift_1          | Compiling Swift Module 'MongoDB' (7 sources)
swift_1          | Compiling Swift Module 'KituraNet' (12 sources)
swift_1          | Compiling Swift Module 'Kitura' (13 sources)
swift_1          | Compiling Swift Module 'NewProject' (1 sources)
swift_1          | Linking .build/debug/NewProject
swift_1          | Setting up watches.  Beware: since -r was given, this may take a while!
swift_1          | Watches established.
```
    
## Batteries included


## XCode

swift build --generate-xcodeproj .


## Deploy to a live server

Note: In this example we are going to use docker-machine to create a remote server on DigitalOcean
and install docker on it. The cool thing about docker-machine is that it works with a lot of 
[cloud providers](https://docs.docker.com/machine/drivers/) out of the box. I've found DigitalOcean
to be the easiest to get started, but if you are a fan of AWS/Microsoft Azure/Google Compute 
Engine or whatnot, just use them.



## Creating a Droplet with Docker Machine

You need an API access token to create a new server. Let's get one:

- Go to the Digitalocean [Console](https://cloud.digitalocean.com/login) and log in.
- Click on **API** in the header.
- Click on **Generate new token**
- Give the token a unique name, e.g *docker-machine*. Make sure that the write checkbox is checked.
 Otherwise docker-machine won't be able to create a new droplet.
- Click on **Generate Token**
- Copy the newly generated token and store it somewhere safe.

![DigitalOcean admin console](https://dockify.io/content/images/2016/04/digitalocean-api.png)

Now, let's create the machine:
  
    docker-machine create swiftapp --driver=digitalocean --digitalocean-region=nyc3 --digitalocean-size=512mb --digitalocean-access-token=YOUR_TOKEN

This will create a new 512MB Droplet in New York City 3 with the name *swiftapp*. If you want to 
create a Droplet of a different size or in a different datacenter, make sure to adjust 
`--digitalocean-size` and `--digitalocean-region` to your needs.

The command takes a couple of minutes to finish, the output should be similiar to this: 


```
Running pre-create checks...
Creating machine...
(swiftapp) Creating SSH key...
(swiftapp) Creating Digital Ocean droplet...
(swiftapp) Waiting for IP address to be assigned to the Droplet...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with ubuntu(systemd)...
Installing Docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env swiftapp
```

    
*If you get any errors, remove the machine by running `docker-machine rm -f swiftapp`. If this 
also produces errors, you can always delete the Droplet using the Digitalocean web console as a 
fallback.*

## First Production Deployment

    eval $(docker-machine env swiftapp)

This will set a couple of environment variables telling the docker client on our machine to talk
directly to the docker deamon on our server.

Now, we can start our stack on the server with
    
    docker-compose up -d

Don't worry, this will take a couple of minutes the first time. The docker deamon has to fetch all
 the images and install all the low level dependencies first. Subsequent builds will be a lot 
 faster since the image build process will be cached on the server. 
 
If you are bored, you can connect to the server and and launch `htop` to see what's happening 
under the hood.

    docker-machine ssh swiftapp
    apt-get install htop && htop 

Once ready, check out your shiny swift powered web app by running

    open "http://$(docker-machine ip swiftapp)"
    

## Subsequent Deployments

    docker-compose build swift
    docker-compose stop swift
    docker-compose start swift
