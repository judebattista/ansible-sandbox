# Overview
This repo is intended to help you establish a pair of virtual machines that you can use to test Ansible playbooks against.
The machines will be set up using Vagrant and Hyper-V in Windows.
We will run all the commands to manage the VMs and Ansible in WSL.
We have to jump through a few hoops to avoid nested virtualization issues: even though both Vagrant and Ansible will be controlled through the WSL command line interface, we will execute Vagrant in our Windows (host) environment so they run parallel to WSL's virtualization.

# Prerequisites
1. Ensure you have administrative rights on your machine. You will need both to configure your system and to execute some of the commands. 
2. Ensure you have WSL installed on your system. You can check this with the following command:

`wsl --status`

For more information you can use:

`wsl --version`

For this guide, I used WSL 2 running on Windows 11, hosting both Kali and Ubuntu distributions. If you need to install WSL, [this article](https://learn.microsoft.com/en-us/windows/wsl/install) from Microsoft is helpful.

# One-time setup
## Windows (host) config:
1. Make sure you’re on Windows Pro and enable Hyper-V (Control Panel → Turn Windows features on/off → Hyper-V).
2. Create a virtual switch in Hyper-V using the following settings:
	1. External network
	2. Use your primary ethernet/wi-fi adapter
	3. Check "allow management operating system to share this network adapter"
	4. For compatibility with this repo, name the switch "ansibleSandbox". You can use any name you wish, but you'll need to edit the Vagrantfile later.

Note: Open Windows Terminal as Administrator whenever you will run `vagrant up` with the Hyper-V provider. Hyper-V requires elevated privileges to run correctly.

## WSL: Update and install basic dependencies on WSL
1. sudo apt update
2. sudo apt install -y curl unzip ca-certificates build-essential

## WSL: Install Vagrant (HashiCorp’s repo gives you a current version)
1. Use your chosen distro's package manager. I used apt: `sudo apt install vagrant`
2. Install manually using the instructions at [Hashicorp's site](https://developer.hashicorp.com/vagrant/install)

## WSL: Install Ansible. This example uses venv, but you can use whatever environment manager you choose.
1. sudo apt install -y python3-venv python3-pip
2. python3 -m venv ~/.venvs/ansible
3. source ~/.venvs/ansible/bin/activate
4. pip install --upgrade pip
5. pip install ansible

## WSL: Allow Vagrant to see Windows tools & paths:
Add this to your WSL shell environment (e.g. ~/.bashrc): `export VAGRANT\_WSL\_ENABLE\_WINDOWS\_ACCESS="1"`

This will allow us to use WSL to issue commands to Vagrant even though the VMs will be running under windows.

If you would prefer to control your VMs through Powershell, you can omit this step.

# Per-project set up
## If you want to work with this repo as your base, clone the repo to your **Windows** file system.
For continuity in this documentation, we will assume it is installed to C:/dev/ansible-sandbox

Using WSL:
1. `cd /mnt/c/dev/ansible-sandbox`
2. `git@github.com:judebattista/ansible-sandbox.git`

Alternatively you can clone the repo from Powershell:
1. `cd c:/dev/ansible-sandbox`
2. `git clone git@github.com:judebattista/ansible-sandbox.git`

## Set up Vagrantfile
This repo contains a sample Vagrant file that stands up two Ubunta 22.04 boxes
It uses the Hyper-V provider and the ansibleSandbox bridge we created during the Windows config
If you named your Hyper-V virtual switch something other than "ansibleSandbox" you will need to esit the Vagrantfile (line 24 as of this version).
## Set up Ansible hosts file
1. Bring up Vagrant and capture SSH config
	1. Open an elevated Windows Terminal to keep Hyper-V happy, then open your WSL tab
	2. cd to your repo in the Windows file system: `cd /mnt/c/dev/ansible-sandbox`
	3. Start vagrant with: `vagrant up --provider=hyperv`
	4. Get the ssh info: `vagrant ssh-config > ssh.cfg`
2. Create hosts.ini or hosts.yaml using the info now captured in ssh.cfg
This repo contains a hosts.yaml file designed to work with the local Vagrantfile
3. Test it your set up:
	`source ~/.venvs/ansble/bin/activate
	ansible all -i hosts.ini -m ping`
4. Run your playbook
Assuming your playbook is site.yml and targets both hosts:
	`ansible-playbook -i hosts.ini site.yml`
