Vagrant.configure("2") do |config|
  # Use a hyperv friendly Ubuntu box
  config.vm.box = "generic/ubuntu2204"

  # provider is Hyper-V
  config.vm.provider "hyperv" do |hv|
    hv.vm_integration_services = {guest_service_interface: true}
    hv.ip_address_timeout = 60
  end

  # Common SSH settings so Ansible can use vagrant's ssh-config
  config.ssh.insert_key = false
  config.ssh.forward_agent = true

  # Disable the default synched folder which we don't need for WSL. Can use rsync if necessary
  config.vm.synced_folder ".", "/vagrant", disabled: true
  
  # Define VMs
  %w[vagrant0 vagrant1].each do |name|
    config.vm.define name do |node|
      node.vm.hostname = "lab-#{name}"

      # Attach to our custom Hyper-V switch via public network
      node.vm.network "public_network", bridge: "ansibleSandbox", mac: "auto"

      # Set up VM params
      node.vm.provider "hyperv" do |hv|
        hv.memory = 2048
        hv.cpus = 2
        hv.vmname = "lab-#{name}"
      end
    
      # Because we want to use Ansible, let's make sure Python is installed
      node.vm.provision "shell", inline: <<-SHELL
        sudo apt-get update -y
        sudo apt-get install -y python3 python3-apt
      SHELL
 
    end

  end

end
