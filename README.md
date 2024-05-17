Some info and help, on how to fix certain self-hosted apps, that I was struggling with

# Problems and fixes:
## Problem #1: Twingate connector status flipping on and off
### Solution
- Installing a timesync service on the vm/server
- Step 1 install chrony:

    ```bash
    sudo apt install chrony -y
    ```
    
- Step 2 edit the chrony.conf file:

    ```bash
    nano /etc/chrony.conf
    ```
- Step 3 edit the config file to look like the following image:

    ![image](https://www.linuxtechi.com/wp-content/uploads/2020/01/Chrony-Conf-Linux-Server-768x642.png?ezimgfmt=ng:webp/ngcb22)

- Step 4 (optional) test the timeserver:

    ```bash
    chronyd -q 'server 0.europe.pool.ntp.org iburst'
    ```

- Step 5 start chrony:

    ```bash
    systemctl start chronyd
    ```

- Step 6 enable chrony on system start:

    ```bash
    systemctl enable chronyd
    ```

## Problem #2: Nextcloud client cannot connect to server behind reverse proxy, reverse proxy url not trusted
### Solution
- Editing the config.php of the nextcloud instance
- Step 1 open the config.php (you can find the file in the config in the YOURNEXTCLOUDFOLDER/config/ folder if installed with docker)

    ```bash
    nano YOURNEXTCLOUDFOLDER/config/config.php
    ```
  
    Edit the file to have something like this in the config:
    ```php
    'trusted_domains' => 
    array (
      0 => '192.168.1.xx:12345',
      10 => 'yournextcloudreverseproxyurl.com',
    ),
    'overwrite.cli.url' => 'https://yournextcloudreverseproxyurl.com',
    'overwriteprotocol' => 'https',
    ```

- Step 2 save the file and restart the nextcloud container

    ```bash
    sudo docker restart NEXTCLOUDCONTAINERNAME_OR_ID
    ```

## Problem #3: Nginx Proxy Manager cannot create SSL cert with letsencrypt
### Solution
- Using an older release of the container solves the issue (atleast it worked for me)
- Step 1 modify the docker-compose.yml used to create the container to use an older version

    ```bash
    nano PATHTOFILE/docker-compose.yml
    ```

    ```yaml
    version: "3"
    
    services:
      nginxproxymanager:
        image: 'jc21/nginx-proxy-manager:2.11.0'
        restart: unless-stopped
        ports:
          - '80:80'
          - '443:443'
          - '81:81'

        volumes:
          - ./nginxproxymanager/data:/data
          - ./nginxproxymanager/letsencrypt:/etc/letsencrypt
    ```
    *Note: you should use the 2.11.0 version of the image, as that works 99% of the time*

- Step 2 restart the container

    ```bash
    sudo docker-compose restart
    ```
