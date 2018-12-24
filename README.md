## Item Catalog Deployment configuration
### Global information

 1. Server IP: 13.233.108.140
 2. SSH Port: 2200
 3. URL: http://13.233.108.140.xip.io/

### SSH Configuration

 - Create new [Amazon Lightsail](https://aws.amazon.com/lightsail/) Ubuntu Instance
 - Connect to instance using SSH client or amazon client
 - Update instance ready installed liberaries

> $ sudo apt-get update 
> $ supo apt-get upgrade

- Change SSH Port from 22 to 2200
> $ sudo nano /etc/ssh/sshd_config

find port line,  un-comment it then change it to 2200 
restart SSH service 
> $ sudo service ssh restart

- Create grader user
>$ sudo adduser grader

then added grader user to sudoers.d 
>$ sudo nano /etc/etc/sudoers.d/grader

and paste this : `grader ALL=(ALL:ALL) ALL`

- Create SSH for grader access
on local machine generate key pair using this command: 
> $ ssh-keygen

then copy generated public key to authorized_keys file on lightsail machine:
>$ sudo nano /home/grader/.ssh/authorized_keys
> $ sudo chmod 700 /home/grader/.ssh
> $ sudo chmod 644 /home/grader/.ssh/authorized_keys

user grader can access remote server using ssh:
>$  ssh grader@13.233.108.140 -p 2200 -i ~/.ssh/graderKeys
- Disable ssh for root user
>$ sudo nano /etc/ssh/sshd_config

and change `PermitRootLogin` value to `no`
#### restart SSH service
> $ sudo service ssh restart

### Firewall configs
we will need to allow 80, 2200 ports as follows:
> $ sudo ufw allow 80/tcp
> $ sudo ufw allow 2200/tcp
> $ sudo ufw enable

we will also need to  make sure that lightsail list of allow ports are the same through `networking` tab on lightsail portal.

### Project dependencies
Install git:
> $ sudo apt-get install git

Install apache2
> $ sudo apt-get install apache2

install wsgi-mod library
>$ sudo apt-get install libapache2-mod-wsgi

restart apache2 service to enable changes and enable mod-wsgi 
>$ sudo /etc/init.d/apache2 restart

clone Catalog project to `/var/www/FlaskApps` folder 
>$ sudo git clone https://github.com/dotmido/uda-items-project.git catalog
>$ cd /var/www/FlaskApps/catalog
>$ sudo touch catalog.wsgi

and paste follw to `catalog.wsgi` file just created to handle requests to our application.

    #!/usr/bin/python
    import sys
    sys.path.insert(0,"/var/www/FlaskApps/")
    from catalog import app as application

> $ sudo apt-get install postgresql
> $ sudo apt-get install python-pip

#### configure Postgresql 
> $ sudo nano /etc/postgresql/9.5/main/pg_hba.conf #disable remote connection
> $ sudo -u postgres createuser -P catalog

#### configure apache2 to handle requests
>$ sudo touch /etc/apache2/sites-enabled/catalog.conf
>$ sudo nano /etc/apache2/sites-enabled/catalog.conf

    <VirtualHost *:80>
    ServerAdmin mjavax@gmail.com
    ServerName 13.233.108.140.xip.io
    ServerAlias 13.233.108.140
    ErrorLog /var/www/FlaskApps/error.log
    CustomLog /var/www/FlaskApps/access.log combined
    WSGIDaemonProcess catalog user=www-data group=www-data threads=5
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/FlaskApps/catalog/catalog.wsgi
    Alias /static/ /var/www/FlaskApps/catalog/static/
    <Directory /var/www/FlaskApps/catalog/static/>
    Order allow,deny
    Allow from all
    </Directory>
    </VirtualHost>

### install Catalog app dependencies

> $ sudo pip install Flask 
> $ sudo pip install httplib2 
> $ sudo pip install requests 
> $ sudo pip install oauth2client 
> $ sudo pip install sqlalchemy
> $ sudo pip install flask_bootstrap
