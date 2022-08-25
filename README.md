# Host your own matrix server in the cloud(AWS)

*This doc assumes basic knowledge of how to use `git` and familiarity with the command-line*

We are going to spin up our own matrix server in the cloud. In this doc, we are aiming for the following

- spin up own server in the cloud and be able to ssh to it
- setup this [matrix-ansible-playbook](https://github.com/spantaleev/matrix-docker-ansible-deploy) and edit configuration to properly install and setup matrix in our cloud server without too much overhead
- connect to our matrix server using [Element UI](https://element.io/matrix-services/server-hosting)

Because this doc will assume your are running the setup locally from your laptop/deskop, we need to get first things right before going on.

- First, we need to ensure we have [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-specific-operating-systems) installed locally. For MacOs flavors, `brew install ansible` will do the trick if you can't install [natively](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible)
    - Run `ansible --version` to verify it was properly installed.
- We are going to get familiar with `ansible-playbook` command to get everything up and running ✌️ .

Once you've got an okay from the above, we can proceed to spinning up our server in **AWS**. First, [shopping](https://aws.amazon.com/) for a cloud server.

## Prerequisite

x86 server(this doc supports this architecture) - you can spin one yourself in [AWS](https://aws.amazon.com/)
  - **Ubuntu** >= 22.04: you can select this OS and version when setting up your server
- Make sure some of the following TCP ports are open(this way you can allow this playbook to run connection on your server) - you can set these up in instance security credentials:
  -  `80/tcp` for HTTP webserver
  -  `443/tcp` for HTTPS webserver
  -  `8448/tcp` for matrix federation api. We need this in-order for our server to be discoverable in matrix network.
  **There are more ports needed to enable feature like voice/video chat but for now this will do.**

[Domain](https://github.com/spantaleev/matrix-docker-ansible-deploy/blob/master/docs/configuring-dns.md#configuring-your-dns-server)
- Your matrix server url will be like `https:matrix.<your_domain>.com`, so it looks like we need to do domain shopping before proceeding. If you looks at the url, `matrix` is a subdomain. So this [guide](https://github.com/spantaleev/matrix-docker-ansible-deploy/blob/master/docs/configuring-dns.md#dns-settings-for-services-enabled-by-default) will definitely get setup you up with necessary dns configurations. `element` subdomain will be where element UI will be installed and be able to access your matrix server from the browser. At the end of this doc you'll have access to your matrix server at `https://element.<your_domain>`.
- I went extra and this how my dns configs look like:
  - | Type | Host | Target |
  - | -----|------|--------|
  - | A Record | @ | <server_ip_address> |
  - | A Record | www | <server_ip_address> |
  - | A Record | matrix | <server_ip_address> |
  - | CNAME Record | element | matrix.<domain> |

- To setup extra features, such as voice/video chat, this [guide](https://github.com/spantaleev/matrix-docker-ansible-deploy/blob/master/docs/configuring-dns.md#dns-settings-for-optional-servicesfeatures) is quite exhaustive with necessary dns configs to unlock extra of those ✌️ .
 

**We are more interested in getting our server up, connect to it, and be able to invite user(s) & send messages to our server.**

## Getting the script for setup

We are going to use [matrix-ansible-playbook](https://github.com/spantaleev/matrix-docker-ansible-deploy) to provision resources to our server. You can choose to clone this repo into your home or directory of your choice.

Fire up your terminal and run this [command](https://github.com/spantaleev/matrix-docker-ansible-deploy/blob/master/docs/getting-the-playbook.md#using-git-to-get-the-playbook) inside the directory of your choice. Once that is done, navigate into `matrix-docker-ansible-playbook` and we are going to run any setup, installatio, services upgrading in this directory.

## Configuring the script

Once you have the okay of the above, we are about to edit some playbook default configs to our liking.
- Create a domain directory `mkdir inventory/host_vars/matrix.<your_domain>`
- Copy the sample configuration `cp examples/vars.yml inventory/host_vars/matrix.<your-domain>/vars.yml`
- Edit `inventory/host_vars/matrix.<your-domain>/vars.yml`
- Copy inventory `cp examples/hosts inventory/hosts`
- Edit `inventory/hosts` to your liking

I think that all you need. If anything this [guide](https://github.com/spantaleev/matrix-docker-ansible-deploy/blob/master/docs/configuring-playbook.md#configuring-the-ansible-playbook) is exhaustive ✌️ .

## Installing

Make sure you have:
- setup and configured your domain ✅
- downloaded the playbook ✅
- configured the playbook as needed ✅
before we proceed.

Run the following [command](https://github.com/spantaleev/matrix-docker-ansible-deploy/blob/master/docs/installing.md#1-installing-the-matrix-services) to start provisioning matrix services to your server.

Once the above has completed successfully, run this [command](https://github.com/spantaleev/matrix-docker-ansible-deploy/blob/master/docs/installing.md#3-starting-the-services) to start matrix services

## Setup federation

This is so that your server is discoverable in the matrix network. This [guide](https://github.com/spantaleev/matrix-docker-ansible-deploy/blob/master/docs/configuring-playbook-base-domain-serving.md#serving-the-base-domain) requires that you add a config line to your playbook configuration.

Remember, the playbook configuration is `inventory/host_vars/matrix.<your-domain>/vars.yml`. **Any edit to this file will require service restart: `ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start`**

Verify that its all good, navigate to `https://<your_domain>`. You will be served with a `Hello from <your_domain>!` page in the browser.

## Final

If everything ran successfully, you should be able to access your matrix server from `https://element.<your_domain>`. You will be presented with a login page which needs a username and password. The easier way to get around this is to run the below command to create the first genesis user in your matrix server:

```
ansible-playbook -i inventory/hosts setup.yml -e username=<your_choice> -e password=<your_choice> -e admin=yes --tags=register-user
```

