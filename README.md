## One-time setup
On Windows (host):
	Make sure you’re on Windows Pro and enable Hyper-V (Control Panel → Turn Windows features on/off → Hyper-V).

	Open Windows Terminal as Administrator whenever you’ll run vagrant up with the Hyper-V provider (Hyper-V needs elevation).

# Update and basic dependencies
1. sudo apt update
2. sudo apt install -y curl unzip ca-certificates build-essential

# Install Vagrant (HashiCorp’s repo gives you a current version)
1. Use a package manager: sudo apt install vagrant
2. Install manually: 
  1. curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp.gpg
  2. echo "deb [signed-by=/usr/share/keyrings/hashicorp.gpg] https://apt.releases.hashicorp.com kali main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
  3. update && sudo apt install -y vagrant

# Ansible (use a venv if you like)
1. sudo apt install -y python3-venv python3-pip
2. python3 -m venv ~/.venvs/ansible
3. source ~/.venvs/ansible/bin/activate
4. pip install --upgrade pip
5. pip install ansible

# Create a virtual switch in Hyper-V
Choose the following:
1. External network
2. Use your primary ethernet/wi-fi adapter
3. Check "allow management operating system to share this network adapter"

# Make Vagrant see Windows tools & paths:
Add this to your WSL shell environment (e.g. ~/.bashrc): `export VAGRANT\_WSL\_ENABLE\_WINDOWS\_ACCESS="1"`

## Per-project set up

# Set up Vagrantfile

# Set up Ansible hosts file
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
