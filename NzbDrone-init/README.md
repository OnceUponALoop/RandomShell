The following instructions are for installing NZBDrone on CentOS 6.

The installation should also be applicable to RHEL 6 and Fedora (12, 13, or 14) with minimal changes.

The installation assumes that you're not using the root user to install/run NZBDrone - the entries for **user:group** throughout the document will have to be modified to match your user configuration. 

### Install NZBDrone

1. **[Optional] Configure sudo for your user**
    ```bash
    echo "user   ALL=(ALL)       ALL" >> /etc/sudoers
    ```

2. **Install EPEL & OpenSuse repository**
    
    EPEL: is needed for mediainfo installation
    <br>
    Mono: is needed for mono-opt and mono-opt-devel installation

   ```bash
   sudo rpm -ivh http://mirror.cs.pitt.edu/fedora/epel/6/i386/epel-release-6-8.noarch.rpm
   sudo curl -L http://download.opensuse.org/repositories/home:tpokorra:mono/CentOS_CentOS-6/home:tpokorra:mono.repo -o /etc/yum.repos.d/mono.repo
   ```

3. **Install prerequisites**
    
    I chose to install the latest mono (3.4.0) from the OpenSuse repo instead of using the older CentOS version (2.10.8)
    <br>
    The OpenSuse mono-opt will be installed under /opt so it can probably coexist with the CentOS mono-core (i havent tested this myself!)

    ```bash
    sudo yum install gcc-c++ gcc mediainfo gettext mono-opt mono-opt-devel
    ```

4. **Download sqlite3.8.5 source and extract it**
    
    ```bash
    curl -L http://www.sqlite.org/2014/sqlite-autoconf-3080500.tar.gz -o /tmp/sqlite-autoconf-3080500.tar.gz
    tar -zxvf /tmp/sqlite-autoconf-*.tar.gz -C /tmp/
    ```

5. **Compile sqlite3.8.5**
    
    NZBDrone requires the COLUMN_METADATA option so we'll enable it.
    <br>
    I didn't want to replace the system sqlite installation (v3.6.20) since CentOS depends on it so I'm using the --prefix option to install it to another directory.
    <br>
    The init script will be configured to call this new version when running NZBDrone. 

    ```bash
    cd /tmp/sqlite-autoconf*

    sudo ./configure --prefix=/opt/sqlite3.8.5          \
                     --disable-static                   \
                     CFLAGS=" -Os                       \
                     -frecord-gcc-switches              \
                     -DSQLITE_ENABLE_COLUMN_METADATA=1"
    
    sudo make
    sudo make install
    ```

6. **Download and extract mono and optionaly rename it**
    
    ```bash
    curl -L http://update.nzbdrone.com/v2/master/mono/NzbDrone.master.tar.gz -o /tmp/NzbDrone.master.tar.gz
    sudo tar zxvf /tmp/NzbDrone.master.tar.gz -C /opt/
    ```

    I wanted to maintain lowercase naming so i renamed the NzbDrone program directory
    <br>
    ```bash
    sudo mv /opt/NzbDrone /opt/nzbdrone
    ```


7. **[Optional] Create link for NZBDrone configuraiton directory**

    I wanted to keep NZB Drone files in the program directory, so i created a link

    ```bash
    mkdir ~/.config
    sudo mkdir /opt/nzbdrone/config
    sudo chown user:group /opt/nzbdrone/config
    ln -s /opt/nzbdrone/config/ ~/.config/NzbDrone
    ```

8. **Run NZBDrone to test**

    Just run it to verify everything is working the stop it (CTRL-C) and move to the next steps.
    
    The freakishly long command below will:
        <br>
            1. Set the LD_LIBRARY_PATH and PATH to point to the new sqlite and mono directories
        <br>
            2. Run NZB Drone
        <br>
            3. Set the enviroment back to normal.

    ```bash
    export LPATH_BAK=$LD_LIBRARY_PATH ; export PATH_BAK=$PATH   && \
    export LD_LIBRARY_PATH=/opt/mono/lib:/opt/sqlite3.8.5/lib   && \
    export PATH=/opt/mono/bin:/opt/sqlite3.8.5/bin:$PATH        && \
    /opt/mono/bin/mono /opt/nzbdrone/NzbDrone.exe               ;  \
    export PATH=$PATH_BAK ; export LD_LIBRARY_PATH=$LPATH_BAK
    ```

### Create NZBDrone init Service

1. **Download init script and configuration files**

    ```bash
    sudo curl -L https://raw.githubusercontent.com/OnceUponALoop/RandomShell/master/NzbDrone-init/nzbdrone.init.centos     -o /etc/init.d/nzbdrone
    sudo curl -L https://raw.githubusercontent.com/OnceUponALoop/RandomShell/master/NzbDrone-init/nzbdrone.init-cfg.centos -o /etc/sysconfig/nzbdrone
    ```

    Set the correct permissions
    ```bash
    sudo chmod +x /etc/init.d/nzbdrone
    sudo chmod 644 /etc/sysconfig/nzbdrone
    ```

2. **Edit the configration file to suit your enviroment**

    If you followed the instructions above without changing any of the installation paths the only change you'll need to make is to the nzbdroneUser.
    ```bash
    sudo vi /etc/sysconfig/nzbdrone
    ```

3. **Add the nzbdrone service to system services**
    
    ```bash
    sudo chkconfig --add nzbdrone
    ```

4. **Configure nzbdrone service to start on system startup**
    
    ```bash
    sudo chkconfig nzbdrone on
    ```

5. **Start nzbdrone service**
    
    ```bash
    sudo service nzbdrone start
    ```