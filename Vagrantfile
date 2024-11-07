# Vagrantfile para configurar 3 máquinas virtuales en VirtualBox con especificaciones dadas
Vagrant.configure("2") do |config|
  # Configuración general para todas las máquinas
  config.vm.provider "virtualbox" do |vb|
    vb.cpus = 1
    vb.memory = 512
  end

  # Máquina 1: Servidor Apache 1
  config.vm.define "web1" do |web1|
    web1.vm.box = "generic/ubuntu2004"  # Puedes cambiar el sistema operativo si lo necesitas
    web1.vm.hostname = "web1"
    web1.vm.network "private_network", ip: "192.168.56.10"
    
    # Configuración de Apache y directorio compartido
    web1.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y apache2
      echo "<h1>Servidor Web 1</h1>" > /var/www/html/index.html
      systemctl enable apache2
      systemctl start apache2
    SHELL

    # Sincronizar directorio local con el servidor
    web1.vm.synced_folder "./web_content", "/var/www/html", create: true
  end

  # Máquina 2: Servidor Apache 2
  config.vm.define "web2" do |web2|
    web2.vm.box = "generic/ubuntu2004"
    web2.vm.hostname = "web2"
    web2.vm.network "private_network", ip: "192.168.56.11"
    
    # Configuración de Apache y directorio compartido
    web2.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y apache2
      echo "<h1>Servidor Web 2</h1>" > /var/www/html/index.html
      systemctl enable apache2
      systemctl start apache2
    SHELL

    # Sincronizar directorio local con el servidor
    web2.vm.synced_folder "./web_content", "/var/www/html", create: true
  end

  # Máquina 3: Balanceador de carga con Nginx
  config.vm.define "lb" do |lb|
    lb.vm.box = "generic/ubuntu2004"
    lb.vm.hostname = "lb"
    lb.vm.network "private_network", ip: "192.168.56.12"
    
    # Configuración de Nginx y balanceo de carga
    lb.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y nginx
      
      # Configuración de balanceo de carga en Nginx
      echo '
      upstream backend {
          server 192.168.56.10;
          server 192.168.56.11;
      }

      server {
          listen 80;

          location / {
              proxy_pass http://backend;
              proxy_set_header Host $host;
              proxy_set_header X-Real-IP $remote_addr;
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
              proxy_set_header X-Forwarded-Proto $scheme;
          }
      }
      ' | sudo tee /etc/nginx/conf.d/load-balancer.conf

      sudo rm /etc/nginx/sites-enabled/default
      sudo systemctl restart nginx
      sudo systemctl enable nginx
    SHELL
  end
end
