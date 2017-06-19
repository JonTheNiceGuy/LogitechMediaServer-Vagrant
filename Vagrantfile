Vagrant.configure("2") do |config|
  config.vm.define "logitechmediaserver" do |logitechmediaserver|
    logitechmediaserver.ssh.insert_key = true
    logitechmediaserver.ssh.forward_agent = true

    logitechmediaserver.vm.box = "ubuntu/xenial64"
    logitechmediaserver.vm.hostname = "logitechmediaserver"
    logitechmediaserver.vm.box_check_update = false
    logitechmediaserver.vm.boot_timeout = 65535

    logitechmediaserver.vm.network "public_network", bridge: "enp0s25", ip: "192.168.1.101", :use_dhcp_assigned_default_route => true
    logitechmediaserver.vm.synced_folder "/content", "/media"

    logitechmediaserver.vm.provider "virtualbox" do |vb|
      vb.name = "LogitechMediaServer"
    end

    logitechmediaserver.vm.provision "shell", run: "always", inline: <<-SHELL
      echo "Checking for presence of the LMS server"
      if dpkg -l logitechmediaserver >/dev/null ; then
        echo "Checking for a recent (>00:00:00 today) backup)"
        export BACKUPDIR=/vagrant/config/$(date +%Y-%m-%d)/
        if [ -d /var/lib/squeezeboxserver ] && [ ! -d $BACKUPDIR ] && mkdir -p $BACKUPDIR ; then
          echo "LMS found. Stopping service"
          systemctl stop logitechmediaserver 2>/dev/null

          echo "About to run backup: cp -Rf /var/lib/squeezeboxserver/* $BACKUPDIR"
          cp -Rf /var/lib/squeezeboxserver/* $BACKUPDIR

          echo "Restarting LMS Service."
          systemctl start logitechmediaserver 2>/dev/null
        fi
      fi
    SHELL

    logitechmediaserver.vm.provision "shell", inline: <<-SHELL
      if [ "$(grep squeezeboxserver /etc/passwd)" == "" ]; then
        echo "LMS user account not created... Creating"
        echo "squeezeboxserver:x:1000:1000:Logitech Media Server,,,:/usr/share/squeezeboxserver:/bin/false" >>/etc/passwd
      fi

      export RESTART_SVC=false
      if dpkg -l logitechmediaserver >/dev/null ; then
        echo "Stopping LMS service"
        systemctl stop logitechmediaserver 2>/dev/null
        export RESTART_SVC=true
      fi

    
      export BACKUPDIR=$(find /vagrant/config/* -maxdepth 0 2>/dev/null | sort -r | head -n 1)
      if [ ! -z "$BACKUPDIR" ] ; then
        echo -n "Installing LMS config files: "
        echo cp -Rf "${BACKUPDIR}/*" "/var/lib/squeezeboxserver/"
        cp -Rf "${BACKUPDIR}/*" "/var/lib/squeezeboxserver/"
      fi

      mkdir -p /vagrant/packages
      if [ ! -z "$(find /vagrant/packages/*.deb -maxdepth 0 2>/dev/null | sort -r | head -n 1)" ]; then
        echo -n "Installing the LMS server packages: "
        echo dpkg -i "$(find /vagrant/packages/*.deb -maxdepth 0 2>/dev/null | sort -r | head -n 1)"
        dpkg -i "$(find /vagrant/packages/*.deb -maxdepth 0 2>/dev/null | sort -r | head -n 1)"
      else
        export release_file=$(curl http://downloads-origin.slimdevices.com/nightly/?ver=7.9 2>/dev/null | grep amd64.deb | cut -d '"' -f 2)

        export release_id=$(echo $release_file | cut -d/ -f 4)
        export release_ver=$(echo $release_file | cut -d/ -f 5)
        echo -n "Downloading the LMS server packages: "
        echo "wget -O /vagrant/packages/${release_ver} http://downloads.slimdevices.com/nightly/7.9/sc/${release_id}/${release_ver} 2>/dev/null && dpkg -i /vagrant/packages/${release_ver}"
        wget -O /vagrant/packages/${release_ver} http://downloads.slimdevices.com/nightly/7.9/sc/${release_id}/${release_ver} 2>/dev/null && dpkg -i /vagrant/packages/${release_ver}
      fi

      # Restart the LMS server, if it was stopped earlier
      if [ "$RESTART_SVC" == "true" ]; then
        echo "Restarting the LMS service"
        systemctl start logitechmediaserver 2>/dev/null
      fi
    SHELL
  end
end
