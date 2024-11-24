# Ansible WSL first time setup for development environment
Seeing that every time I install a fresh updated WSL on my Windows laptop,
I find myself needing to set up the development environment from scratch,
I figured I would help myself in automating this task using Ansible.
Thanks to @geerlingguy who uses and documents and promotes Ansible, I started looking into this great tool!

I figured I would turn it into a public repository if anyone else finds it useful.
It is currently geared towards Ubuntu WSL environments,
but it could probably be tweaked to work for other environments also with a few conditionals
and with blocks geared towards other environments.

## Setup
The current setup takes for granted the folder structure `~/ansible-wsl-development-setup`.
Just clone this repo to your home folder in your fresh WSL installation, and it should work alright.
If you want to use a different folder structure, you should change `ansible.cfg` to correctly point the `roles_path` directive.

With a fresh WSL install, you won't have GPG keys set up, so you probably won't be able to clone the SSH repository.
Just clone with the HTTPS repository and it should go smoother.
This Ansible playbook will take care of generating a new GPG key and adding it to your Github profile,
so you will then be able to clone SSH repositories in your new WSL installation without any trouble.

### Clone the repo
This setup takes for granted that you'll clone the repo to your home folder `~`,
so the resulting path would be `~/ansible-wsl-development-setup`.
```shell
git clone https://github.com/JohnRDOrazio/ansible-wsl-development-setup
```

### Install Ansible
Firstly we will need to have Ansible installed, and Ansible requires python.
So we will create a python virtual environment in which to run ansible.
```shell
sudo apt update
sudo apt upgrade
sudo apt install python3-pip python3-venv python-is-python3
cd ~/ansible-wsl-development-setup
python -m venv .venv
source .venv/bin/activate
pip install ansible jmespath python-debian
```

I'm taking for granted that any python development will be done using Python 3,
so installing the `python-is-python3` package will allow you to use `python` and `pip`
rather than specifying `python3` or `pip3` every time.

The `jmespath` package is required for parsing JSON (which this playbook does).

The `python-debian` package is required by the new `ansible.builtin.deb822_repository` module.
This module requires Ansible 2.15 or later.

### Install ansible roles
In Ansible, **roles** are like precooked or reusable playbooks that take care of a number of tasks
that would otherwise require quite a bit of effort to write from scratch. Using **roles** can help
keep your playbooks as clean and simple as possible.

Currently, we are using a single **role** in this playbook, `ansible-role-nvm` by @morgangraphics,
defined in `requirements.yml`. We're using our own forked version to fix a glitch in newer versions of Ansible.

We must install our roles before running our playbook. From within our project folder (`~/ansible-wsl-development-setup/`) run:
```shell
ansible-galaxy install -r requirements.yml
```

### Define our variables
Before running the playbook, you should define the variables marked with an asterisk comment `#*`
in `vars.example.yml`, then rename or copy this file to `vars.yml`.
If you fork your own copy of this repo or create your own version of this playbook to store in a Github repo,
`vars.yml` is git ignored, so you don't have to worry about your private info winding up in a public repository.

### Run the playbook!
We are now finally ready for Ansible to run the tasks in this playbook!
```shell
ansible-playbook main.yml
```

This will take care of the following tasks (according to whether you have enabled them in `vars.yml`):
- check if an SSH key exists for the current user, and if not, generate a new one
- check if the SSH key is present on your Github profile, and if not, add it
- optionally add the public SSH key to a remote server, for easier SSH management of the remote server
- check if a GPG key exists for the current user, and if not, generate a new one
- check if the GPG key is present on your Github profile, and if not, add it
- make sure that `keychain` is installed and added to user profile (`~/.bashrc`)
- set global git configuration with user, email, signing keys, commit sign turned on by default, and default branch name for new repos set to `main`
- make sure that `NVM` (node version manager) is installed
- make sure that the current latest stable version of `NodeJS` is installed
- make sure that `AVN` (Automatic Version Switching for Node.js) is installed
   *(I fixed `AVN` trying to write to `.bash_profile` rather than `.bashrc` which is the default profile file on Ubuntu)*
- make sure that `RBENV` (Ruby version manager) is installed
- make sure that the current latest stable version of `Ruby` is installed

> [!NOTE]
> When using vscode to "remote ssh" to wsl, often the ssh private key credentials are not cached and every time we try to git push we are prompted for our credentials (and twice at that!).
>
> This usually happens when wsl isn't actually open, I believe.
>
> A workaround to make sure that git will use an actual running ssh agent is to make it use the windows ssh agent:
>
> `git config --global core.sshCommand "/mnt/c/Windows/System32/OpenSSH/ssh.exe"`
> 
> This requires having the windows OpenSSH agent run automatically at startup, see [here](https://github.com/Microsoft/vscode/issues/13680#issuecomment-414841885).
>
> Also, in order to set the pinentry program to the Windows version of pinentry-qt, it is recommended to install [GPG4Win](https://www.gpg4win.org/download.html) before running the playbook.
> Adjust the path to the pinentry executable accordingly.

# TODO
This playbook could easily be made to work for other environments other than Ubuntu.
For example, in other environments, the default profile file for the user is not `.bashrc`,
so any references to `.bashrc` would have to be made conditional based on the environment.
