# -*- mode: ruby -*-
# vi: set ft=ruby :
url_prefix = "https://github.com/prometheus"

# version and url of prometheus stack
ne_version = "1.0.1"
prom_version = "2.22.0"
am_version = "0.21.0"

ne_tar = "node_exporter-#{ne_version}.linux-amd64"
prom_tar = "prometheus-#{prom_version}.linux-amd64"
am_tar = "alertmanager-#{am_version}.linux-amd64"

ne_url = "#{url_prefix}/node_exporter/releases/download/v#{ne_version}/#{ne_tar}.tar.gz"
prom_url = "#{url_prefix}/prometheus/releases/download/v#{prom_version}/#{prom_tar}.tar.gz"
am_url = "#{url_prefix}/alertmanager/releases/download/v#{am_version}/#{am_tar}.tar.gz"

# debian versions of grafana
grafana_url = "https://dl.grafana.com/oss/release/grafana_7.2.1_amd64.deb"

Vagrant.configure("2") do |config|
  # VMs to be monitored by prometheus
  (1..3).each do |i|
    config.vm.define "node-#{i}" do |node|
      node.vm.box = "bento/ubuntu-20.04"
      node.vm.network "private_network", ip: "192.168.50.8#{i}"
      node.vm.synced_folder "files", "/tmp/files"
      node.vm.provision "shell", inline: <<-SHELL
        sudo useradd --no-create-home --shell /bin/false node_exporter

        cd /tmp/
        wget -q #{ne_url}
        tar -xvf #{ne_tar}.tar.gz

        cd #{ne_tar}
        sudo mv node_exporter /usr/local/bin/
        sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter

        sudo cp -vf /tmp/files/node_exporter.service /etc/systemd/system/

        sudo systemctl daemon-reload
        sudo systemctl start node_exporter

        sudo apt-get install -y stress
      SHELL

      node.vm.provider :virtualbox do |vb|
        vb.memory = "512"
        vb.cpus = "1"
      end
    end
  end

  # Monitoring node
  config.vm.define "prometheus" do |app|
    app.vm.box = "bento/ubuntu-20.04"
    app.vm.hostname = "prometheus"
    app.vm.network "private_network", ip: "192.168.50.88"
    app.vm.synced_folder "files", "/tmp/files"

    # Install alertmanager
    app.vm.provision "shell", inline: <<-SHELL
      sudo useradd --no-create-home --shell /bin/false alertmanager
      sudo mkdir /etc/alertmanager

      cd /tmp/
      wget -q #{am_url}
      tar -xvf #{am_tar}.tar.gz
      cd #{am_tar}

      sudo mv alertmanager /usr/local/bin/
      sudo mv amtool /usr/local/bin/

      sudo chown alertmanager:alertmanager /usr/local/bin/alertmanager
      sudo chown alertmanager:alertmanager /usr/local/bin/amtool

      sudo mv alertmanager.yml /etc/alertmanager/
      sudo chown -R alertmanager:alertmanager /etc/alertmanager/

      sudo cp -vf /tmp/files/alertmanager.service /etc/systemd/system/

      sudo systemctl daemon-reload
      sudo systemctl start alertmanager
      sudo systemctl enable alertmanager
    SHELL

    # Install prometheus monitoring system
    app.vm.provision "shell", inline: <<-SHELL
      sudo useradd --no-create-home --shell /bin/false prometheus
      sudo mkdir /etc/prometheus
      sudo mkdir /var/lib/prometheus

      sudo chown prometheus:prometheus /var/lib/prometheus

      cd /tmp/
      wget -q #{prom_url}
      tar -xvf #{prom_tar}.tar.gz

      cd #{prom_tar}
      sudo mv console* /etc/prometheus
      sudo mv prometheus.yml /etc/prometheus
      sudo chown -R prometheus:prometheus /etc/prometheus

      sudo mv prometheus /usr/local/bin/
      sudo mv promtool /usr/local/bin/
      sudo chown prometheus:prometheus /usr/local/bin/prometheus
      sudo chown prometheus:prometheus /usr/local/bin/promtool

      sudo cp -vf /tmp/files/prometheus.service /etc/systemd/system/
      sudo cp -vf /tmp/files/prometheus.yml /etc/systemd/system/

      sudo systemctl daemon-reload
      sudo systemctl start prometheus
      sudo systemctl enable prometheus
    SHELL

    app.vm.provision "shell", inline: <<-SHELL
      sudo apt-get install -y libfontconfig1
      cd /tmp
      wget https://dl.grafana.com/oss/release/grafana_7.2.1_amd64.deb
      sudo dpkg -i grafana_7.2.1_amd64.deb

      sudo systemctl enable --now grafana-server
    SHELL

    app.vm.provider :virtualbox do |vb|
      vb.memory = "4096"
      vb.cpus = "4"
    end
  end
end
