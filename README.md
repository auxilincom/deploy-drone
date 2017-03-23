## Installing Drone CI

This repository aims to help beginners to get started with Drone. Install it within minutes and start moving towards continuous integration. If you want to get full picture about using Drone CI in production and setting up continuous integration process for your product start from our [blog post](https://blog.maqpie.com/2017/03/21/build-and-deploy-applications-using-drone-ci-docker-and-ansible/)

Below we describe in details how to:

1. Install Drone CI on development environment
2. Install production ready Drone CI behind nginx proxy with optional SSL

*If you plan to use Drone CI for your product feel free to copy this repository to your project and change as needed.*

Please, note that we use version 0.5 of Drone, which is not finally released yet, so there can be some breaking changes eventually.
The good news is that we've been using this version in production for two months already and haven't had any issues.

### Setting up Github OAuth application

The first step to do is to go to the [Github](https://github.com/settings/applications/new) and register new OAuth application. If you want to try install drone on the local machine just use `http://localhost:8000/authorize` as `Authorization callback URL`. For the production environment replace `http://localhost:8000` with your schema and domain name.

Once registered, you should have:

1. Github client id
2. Github client secret

You will need them for both, production and local drone versions.

### Install Drone CI on local machine

Prerequisites:

1. [Docker](https://docs.docker.com/engine/installation/)
2. [Docker-compose](https://docs.docker.com/compose/install/)

Installing Drone CI:

1. Rename `drone-example.env` into `drone.env` and set your github clientId, clientSecret as well as your github username
2. Run one command to start Drone CI: `./bin/start-local.sh`

That's it! `./bin/start-local.sh` script uses `docker-compose` to run Drone UI and Drone Agent. Environment variables from `drone.env` are in the `.gitignore` to keep your secrets out of github repository.

Note: When you first see Drone UI it will show `Loading...` in the sidebar - don't worry, this is expected behavior. Just go to the to the Account from top right menu to enable drone for some of your github repositories. From here you can move over to [this repository](https://github.com/maqpie/drone-starter), which can help you get started shipping your product with Drone CI.

### Deploying production ready Drone CI to the Ubuntu 16.04

Deployment to remove server is done using [Ansible](https://www.ansible.com/) - a simple automation tool. Deployment was tested on a brand new Digital Ocean Ubuntu 16.04 server. Some changes might required to run other linux distributives. Deployment steps includes:

1. Docker installation
2. Nginx proxy installation & configuration to work with Drone
3. [PostgreSQL](https://www.postgresql.org/) installation for Drone to persist build information
4. Running Drone & Drone CI inside Docker containers
5. Optional configuration of SSL certificate

Prerequisites:

1. [Ansible](http://docs.ansible.com/ansible/intro_installation.html)
2. Ubuntu 16.04 server & ssh access to that server

Installation steps:

1. Update server ip in `deploy/hosts` file. If you are planning to use same server for both drone and nginx - just put same ip for both, drone & nginx targets.
2. Install Ansible role dependencies with one command: `./bin/install-ansible-dependencies.sh`
3. Update github application callback url to either point to your server ip address or your domain. For example:
`http://ci.myapp.com/authorize` or just `http://server_ip/authorize`.
4. Set server ip variable or domain name in `vars/main.yml` (`nginx_drone_server_name` variable).
5. Update `drone_github_users` in `vars/main.yml` - a comma separated list of github users, who will be able to access your continuous integration server.
6. Rename `credentials-template.yml` into `credentials.yml` and update your github clientId, clientSecret as well as username and password for PostgreSQL database.

Once you done all above, run the following command:

```
./bin/setup-server.sh && ./bin/deploy-drone.sh
```

You should have Drone CI installed by now. The only step remaining is to make Drone work behind Nginx proxy.

### Setting up Nginx for Drone CI

If you would like to install nginx on a same server with Drone CI - just run following command:

```
./bin/setup-nginx.sh
```

If you already have nginx installed somewhere else and just would like to attach drone nginx configuration to existing nginx server you can do following:

1. Set nginx server ip in `hosts` file for the nginx target
2. Run same command `./bin/setup-nginx.sh`, but reply `no` to the question about nginx installation.

In this case nginx configuration for Drone CI will be copied to the existing nginx server.


### Setting up ssl for production Drone CI

1. Place your ssl keys into int ssl-keys directory as app.crt and app.key (they are in a `.gitignore`)
2. Make sure that `server_setup_ssl` is set to true in `vars/main.yml`
3. Deploy ssl using `./bin/setup-server.sh --tags "nginx"`
4. Update callback url in the Github application to start from `https`


### License

MIT
