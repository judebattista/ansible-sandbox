# Overview
This repo is intended to help you establish a pair of virtual machines that you can use to test Ansible playbooks against.
The machines will be set up using Vagrant and Hyper-V in Windows.
We will run all the commands to manage the VMs and Ansible in WSL.
We have to jump through a few hoops to avoid nested virtualization issues: even though both Vagrant and Ansible will be controlled through the WSL command line interface, we will execute Vagrant in our Windows (host) environment so they run parallel to WSL's virtualization.

# One-time setup
## Windows (host) config:
1. Make sure you’re on Windows Pro and enable Hyper-V (Control Panel → Turn Windows features on/off → Hyper-V).
2. Create a virtual switch in Hyper-V using the following settings:
		1. External network
		2. Use your primary ethernet/wi-fi adapter
		3. Check "allow management operating system to share this network adapter"
		4. For compatibility with this repo, name the switch "ansibleSandbox". You can use any name you wish, but you'll need to edit the Vagrantfile later.

Note: Open Windows Terminal as Administrator whenever you’ll run `vagrant up` with the Hyper-V provider as Hyper-V needs elevation.

## WSL Update and install basic dependencies on WSL
1. sudo apt update
2. sudo apt install -y curl unzip ca-certificates build-essential

## WSL: Install Vagrant (HashiCorp’s repo gives you a current version)
1. Use a package manager: sudo apt install vagrant
2. Install manually: 
  	1. curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp.gpg
  	2. echo "deb [signed-by=/usr/share/keyrings/hashicorp.gpg] https://apt.releases.hashicorp.com kali main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
  	3. update && sudo apt install -y vagrant

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

## Set up Vagrantfile
This repo contains a sample Vagrant file that stands up two Ubunta 22.04 boxes
It uses the Hyper-V provider and the ansibleSandbox bridge we created during the Windows config
If you named your Hyper-V virtual switch something other than "ansibleSandbox" you will need to esit the Vagrantfile (line 24 as of this version).

## Set up Ansible hosts file
1. Bring up Vagrant and capture SSH config
  	1. Open an elevated Windows Terminal to keep Hyper-V happy, then open your WSL tab
  	2. cd to your repo in the Windows file systel: `cd /mnt/c/dev/ansible-lab`
  	3. Start vagrant with: `vagrant up --provider=hyperv`
  	4. Get the ssh info: `vagrant ssh-config > ssh.cfg`

4) Simple Ansible inventory that reuses ssh.cfg
Create hosts.ini:
	`[web]
	web ansible\_host=web

	[db]
	db ansible\_host=db

	[all:vars]
	ansible\_user=vagrant
	ansible\_password=vagrant
	ansible\_ssh\_common\_args='-F ./ssh.cfg'`

Test it:
	`source ~/.venvs/ansble/bin/activate
	ansible all -i hosts.ini -m ping`

5) Run your playbook
Assuming your playbook is site.yml and targets both hosts:
	`ansible-playbook -i hosts.ini site.yml`
