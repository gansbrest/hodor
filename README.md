hodor
=====

Small utility to streamline dev process with Docker and Boot2Docker and Docker Machine ( Mac )

Blog posts with more info:

* [http://distinctplace.com/infrastructure/2014/09/24/docker-vm-shortcomings-and-how-hodor-can-help/](http://distinctplace.com/infrastructure/2014/09/24/docker-vm-shortcomings-and-how-hodor-can-help/)
* [http://distinctplace.com/infrastructure/2015/06/18/docker--hodor-to-simplify-app-setup/](Docker + Hodor for simple and reliable dev setup)

features
=====
* same workflow for Linux and Mac ( Windows not supported yet )
* volume sharing support for current project dir ( with unison and fswatch to keep host and VM in sync )
* exposed docker ports are automatically mapped to your host, it just works (VirtualBox only)
* ssh key sharing support between host and containers ( ex: pull from github inside of the container without typing ssh key passwords )
* containers orchestration with deps management, so we can start containers in particular order
* separate containers list per project ( with .hodorfile )

Hodor Hodor Hodor

requirements
=====
* ruby >= 1.9.3
* docker / boot2docker >= v1.1.2 / docker-machine
* unison >= 2.40.102 ( for two-way sync on Macs, not required for Linux ) [Get it here](https://code.google.com/p/rudix-mountainlion/downloads/detail?name=unison-2.40.102-0.pkg)
* fswatch >= 1.4.3.1 ( for automatic volumes sync on project file change on Macs, not required for Linux ) `brew install fswatch`. Make sure you fswatch version has -o ( --one-per-batch option ). If it's missing you need newer version.

how to use
=====

First you need to clone this repo, `chmod +x hodor` and copy hodor script to `/usr/local/bin`

In your project create file called `.hodorfile` (now processed through ERB) with following structure ( I'll explain options later, but everything should be pretty obvious from this example file:

````
host-manager: docker-machine (Optional. Default: boot2docker)
host: default (Optional. VirtualBox vm name. Defaults: boot2docker-vm (boot2docker), default (docker-machine))

containers:
  redis:
    background: true
    image: gansbrest/redis
    ports: -P
    onetime: false
    volumes:
      __PROJECT__/conf: /data/conf
  jetpack:
    background: false
    image: gansbrest/fc_jetpack
    ports:
      - 49000:8880
    links: ( **note:** if you use `net: host`, but still want to start conainers in particular order, you should use **depends** keyword with the list of continers instead )
      redis: redis
    environment:
      AWS_ACCESS_KEY_ID: <%= ENV['AWS_ACCESS_KEY_ID'] %>
      AWS_SECRET_ACCESS_KEY: <%= ENV['AWS_SECRET_ACCESS_KEY'] %>
    workdir: "/data"
    onetime: true
    
tasks:
  test:
    sync_project_to: /data/slot-fc1
    cmd: "cd /data/slot-fc1 && bash"
    container: "jetpack"
  npm:
    sync_project_to: /data/slot-fc1
    cmd: "cd /data/slot-fc1 && npm"
    container: "jetpack"
  grunt:
    sync_project_to: /data/slot-fc1
    cmd: "cd /data/slot-fc1 && grunt"
    container: "jetpack"
  run:
    sync_project_to: /data/slot-fc1
    cmd: "cd /data/slot-fc1 && ./simple_server"
    container: "jetpack"
  default: run (if no task is given, the run task is executed)
````

Run Hodor with one of the tasks you specified while you are in project dir.

For example `hodor grunt build` - it will execute grunt build inside of the container and auto sync your host and VM after it's done!

or

`hodor run` - will run simple_server ( which is a wrapper for nodemon and some other stuff ) inside of the container. You will see nodemon output. Any files modified on host will automatically sync to container because of fswatch!

This project is very raw. Report any issues, questions or suggestions.

Hope it will save you time and allow to fully enjoy docker!


why not fuse or VirtualBox Guest Additions?
=====

Fuse is just very slow for big projects and inconvenient to setup. VirtualBox Guest Additions is, well, slow. I was not able to use it for our project with 17k small files.. Some people find it useful for small projects though and it's backed into new Boot2Docker, or that I've heard.

why not Vagrant, it does it all already?
=====

Well, let me put it this way: I tried Vagrant and it wasn't good for I needed. I just felt that everything was too abstracted away. Plus file sharing is slow and not even sure for ssh forwarding. Also not a fan of installing sshd on my containers.

Not that I advice against using it, I just felt that I need something different and lightweight.

Read more here:

Docker and Vagrant: [http://stackoverflow.com/questions/16647069/should-i-use-vagrant-or-docker-io-for-creating-an-isolated-environment](http://stackoverflow.com/questions/16647069/should-i-use-vagrant-or-docker-io-for-creating-an-isolated-environment)

SSHD in your containers: [http://jpetazzo.github.io/2014/06/23/docker-ssh-considered-evil/](http://jpetazzo.github.io/2014/06/23/docker-ssh-considered-evil/)

motivation
=====

Once I started working with Docker, my initial reaction was - "Hey, that's great, everything just works and I can even share my volumes and ports with host machine easily.." 

Forgot to mention that I use Ubuntu as my desktop OS..  So I spent about a week to create multiple containers for our FastCompany stack ( node, redis etc ) and decided to convince our devs to integrate Docker as part of the new development process. Everything was suppose to be so much better. Boy, was I surprised..

Most of our dev team members use Macs. I always knew that. I think I was just misled by Docker docs, where they would say "It works everywhere, even on Windows! You just need this tiny VM to make it all happen". Well, let me make it short for you - Docker is PAIN in the b..t when you want to share volumes or ports, basically use it in VM.

I decided to create this tool called Hodor with one goal in mind - to create reliable helper/proxy, which allows to use Docker same way on Linux or Mac where volume sharing and ports are just working.
