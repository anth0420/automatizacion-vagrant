# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
    # Especificamos la versión exacta que tienes instalada
    config.vm.box = "ubuntu/focal64"
    config.vm.box_version = "20230607.0.5"
    
    config.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
      vb.cpus = 1
      # Añadimos estas líneas para evitar problemas comunes
      vb.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
      vb.customize ["modifyvm", :id, "--uartmode1", "file", File::NULL]
    end
  
    # Servidor Web 1
    config.vm.define "web1" do |web1|
      web1.vm.hostname = "web1"
      web1.vm.network "private_network", ip: "192.168.33.11"
      web1.vm.synced_folder "./web_content", "/var/www/html", create: true
      
      web1.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get install -y apache2
        echo "<h1>Servidor Web 1</h1>" > /var/www/html/index.html
        systemctl enable apache2
        systemctl start apache2
      SHELL
    end
  
    # Servidor Web 2
    config.vm.define "web2" do |web2|
      web2.vm.hostname = "web2"
      web2.vm.network "private_network", ip: "192.168.33.12"
      web2.vm.synced_folder "./web_content", "/var/www/html", create: true
      
      web2.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get install -y apache2
        echo "<h1>Servidor Web 2</h1>" > /var/www/html/index.html
        systemctl enable apache2
        systemctl start apache2
      SHELL
    end
  
    # Servidor Nginx (Load Balancer)
    config.vm.define "lb" do |lb|
      lb.vm.hostname = "lb"
      lb.vm.network "private_network", ip: "192.168.33.10"
      
      lb.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get install -y nginx
        
        # Configuración del balanceador de carga
        cat > /etc/nginx/conf.d/load-balancer.conf <<EOF
  upstream backend {
      server 192.168.33.11;
      server 192.168.33.12;
  }
  
  server {
      listen 80;
      server_name localhost;
  
      location / {
          proxy_pass http://backend;
          proxy_set_header Host \$host;
          proxy_set_header X-Real-IP \$remote_addr;
          proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto \$scheme;
      }
  }
  EOF
        
        rm /etc/nginx/sites-enabled/default
        systemctl restart nginx
        systemctl enable nginx
      SHELL
    end
  end