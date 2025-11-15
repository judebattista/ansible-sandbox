Vagrant.configure("2") do |config|
  # Use a Hyper-V friendly Ubuntu box
  config.vm.box = "generic/ubuntu2204"

  # Provider: Hyper-V
  config.vm.provider "hyperv" do |hv|
    hv.vm_integration_services = {
      guest_service_interface: true
    }
    # Use the built-in NATed "Default Switch"
    hv.ip_address_timeout = 300
  end

  # Common SSH settings so Ansible can use vagrant's ssh-config
  config.ssh.insert_key = false
  config.ssh.forward_agent = true

  # Disable the default synced folder (noise under WSL). We'll use rsync if needed.
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # Define two VMs
  %w[web db].each do |name|
    config.vm.define name do |node|
      node.vm.hostname = "lab-#{name}"

      # Attach to Default Switch via "public_network"
      node.vm.network "public_network", bridge: "Default Switch", mac: "auto"

      # Small memory footprint; adjust as needed
      node.vm.provider "hyperv" do |hv|
        hv.memory = 2048
        hv.cpus   = 2
        hv.vmname = "lab-#{name}"
      end

      # Optional: cloud-init style provisioning to ensure Python (for Ansible)
      node.vm.provision "shell", inline: <<-SHELL
        sudo apt-get update -y
        sudo apt-get install -y python3 python3-apt
      SHELL
    end
  end
end

