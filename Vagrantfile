Vagrant.configure("2") do |config|
  # Box base
  config.vm.box = "ubuntu/jammy64"

  # Nome da máquina
  config.vm.hostname = "devops-jr"

  # Rede privada para facilitar o acesso

  # Configurações do VirtualBox
  config.vm.provider "virtualbox" do |vb|

    # Como você tem 8 GB de RAM, vamos limitar o consumo
    vb.memory = 3072
    vb.cpus = 2

    vb.gui = false
  end

  # Compartilha a pasta do projeto com a VM
  config.vm.synced_folder ".", "/vagrant"
end