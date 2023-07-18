---
layout: post
title: "Use make for project basic commands"
date: 2020-11-01
description: # Add description
img: posts/use-make-for-project-basic-commands_1.png # Add image post (optional)
tags: [make] # add tag
---

When we start a new project or we switch to an old project we always bump to the problem with the different commands to execute to start, build, deploy our application. Sometimes we use docker, sometimes we compile a binary to run and sometimes we have a list of commands to execute only to have our environment ready for development. 

One way to keep a trace of the available commands is to define them inside a Makefile and to use the globally know make command to execute the needed one. 
aaa
In this post I will explain some of the commands that I use and why its important to keep all this information available and why to Makefile is one of the solutions.

Introduction
------------

#### Make

Make is an utility that is mostly known for building binaries, but we can also use it to run commands on projects based on a non-compilation language or even for starting our program inside a container.

#### Makefile

The Makefile defines the set of commands available to the make utility. You must create the file in the root folder of your project and execute the make commands always in the root folder.

Commands
--------

#### Help

This command is very usefull to be able to display all the (as I call them) public commands that your Makefile propose. Of course all the commands that you define inside a Makefile are always available but you may want to push forward only the main ones and not the sub-commands.

    help: ## Prints this help message
    	@grep -E '^[a-zA-Z_-]+:.*## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

All the commands that are followed by ## will be displayed when you type **make help**. In this example we have the commented text of the help command that will be displayed.

    >make help
    help				Prints this help message

#### Init

This command is used to Init the workspace. We can include the clear of the cache files, copy or specific configuration, download of external packages or bundles, etc. 

    init: ## Init all dependencies
    	go mod init go_app
    	go mod vendor

#### Build

This command is used for building our application or if we use docker to generate the container image

Example with a Go generated binary : 

    build: ## Build application
    	go build -o ./cmd/go_app/go_app -v ./cmd/...

Example with a docker image build :

    build_docker: ## Build docker container
    	docker build --rm -t "${APP_NAME}:${VERSION}" \
    	--build-arg VERSION=${VERSION} \
    	-f docker/Dockerfile .

#### Start

The start command is used to start the application. You can always use different commands for different environments. I always use start\_local to be sure that the command is specific for my local environment. The start command will create all the necessary dependencies to be able to run the application locally

    start_local: ## Start a local instance of the application
    	docker stack deploy -c docker/docker-compose.yml go_app

#### Stop

The stop command will stop the application and destroy all the links that need to be deleted. For example if we are running a container it can also purge the specific networks and volumes created.

    stop_local: ## Stop the local instance of the application
    	docker stack rm go_app

#### Test

The name says it all. You can also add all the different kind of tests that you want to have and be able to execute them all togheter.

    test: ## Run all the application tests
    	go test ./... -cover -v

#### Example Makefile

Here is an example makefile that I have for a project that will be executed inside a container.

    M = $(shell printf "\033[34;1m▶\033[0m")
    
    .PHONY: help
    help: ## Prints this help message
    	@grep -E '^[a-zA-Z_-]+:.*## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'
    
    .PHONY: init
    init: ## Init all dependencies
    	$(info $(M) Starting init of the project)
    	go mod init go_app
    	go mod vendor
    
    .PHONY: build
    build: ## Build application
    	$(info $(M) Starting build of the application)
    	go build -o ./cmd/go_app/go_app -v ./cmd/...
    
    .PHONY: build_docker
    build_docker: ## Build docker container
    	$(info $(M) Starting build of the container image)
    	docker build --rm -t "${APP_NAME}:${VERSION}" \
    	--build-arg VERSION=${VERSION} \
    	-f docker/Dockerfile .
    
    .PHONY: start_local
    start_local: ## Start a local instance of the application
    	$(info $(M) Starting local instance)
    	docker stack deploy -c docker/docker-compose.yml go_app
    
    .PHONY: stop_local
    stop_local: ## Stop the local instance of the application
    	$(info $(M) Stopping local instance)
    	docker stack rm go_app
    
    .PHONY: test
    test: ## Run all the application tests
    	$(info $(M) Starting application tests)
    	go test ./... -cover -v
    
    .DEFAULT_GOAL := help

And here is the output when you type **make help**

    >make help
    help                           Prints this help message
    init                           Init all dependencies
    build                          Build application
    build_docker                   Build docker container
    start_local                    Start a local instance of the application
    stop_local                     Stop the local instance of the application
    test                           Run all the application tests

Conclusion
----------

Make is a powerful utility that can be used not only for creating binaries but also to access easily all the important commands of your project. It can be a huge time saver and let you concentrate on your code rather on its initialization.

The commands that I have listed are only the ones that I use in most of the projects but you can add every single command that comes on your mind.
