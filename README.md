# Rancher server + agent in HA playground

Rancher server & rancher agent in high availability on local environment with vagrant & virtualbox.
Readme is written for osx/linux, you might need to do some steps in your platfomr way to run it. See full tutorial about this repository here -> https://medium.com/touch4it/rancher-server-rancher-agent-in-high-availability-on-local-environment-with-vagrant-virtualbox-5d7ee913ad07

## Requirements

- git https://git-scm.com/book/en/v2/Getting-Started-Installing-Git
- vagrant https://www.vagrantup.com/docs/installation/ (tested on 2.1.1)
- virtual box https://www.virtualbox.org/wiki/Downloads (tested on 5.2.2)
- rancher-compose https://rancher.com/docs/rancher/v1.6/en/cattle/rancher-compose/ (tested on v0.12.5)

## Setup

```bash
git clone git@github.com:touch4it/rancher-vagrant.git
cd vagrant-rancher/mysql-nginx
vagrant up --provision
cd ../rancher1
vagrant up --provision
cd ../rancher2
vagrant up --provision

sudo echo '192.168.70.10 rancher.vm.localhost' >> /etc/hosts
```

this will take up to 10 minutes
go to http://rancher.vm.localhost
click "Add a host"
set "Host Registration Url" to http://192.168.70.10
click save
copy "docker-run ..." command from point 5 and run it on both rancher1 and rancher2 VMs

```bash
cd ../rancher1
vagrant ssh
# EXECUTE docker run ... COMMAND HERE

cd ../rancher2
vagrant ssh
# EXECUTE docker run ... COMMAND HERE
```

go to http://rancher.vm.localhost/env/1a5/infra/hosts
you should see 2 hosts with all stacks green
this will take up to 10 minutes

go to http://rancher.vm.localhost/env/1a5/api/keys
create new environment key
set key and secret as environment variables

```bash
RANCHER_ACCESS_KEY=<API_KEY>
RANCHER_SECRET_KEY=<API_SECRET>
```

deploy nginx test stack using rancher-compose

```bash
rancher-compose --url http://192.168.70.10 --access-key ${RANCHER_ACCESS_KEY} --secret-key ${RANCHER_SECRET_KEY} \
  -f ./test-stack/rancher-compose.yml -p test-stack up -d --pull --force-upgrade --confirm-upgrade
```

go to http://rancher.vm.localhost/env/1a5/apps/stacks
you should see your stack here
click on the stack name, then click on the container name
that will get you to url similar to this http://rancher.vm.localhost/env/1a5/apps/stacks/1st5/services/1s7/containers
click scale plus button on the left side
another container will spin up, there will be one on each host

then go to http://192.168.70.11 and http://192.168.70.12 and you will see nginx default welcome pages
you are done! you have working rancher playground
