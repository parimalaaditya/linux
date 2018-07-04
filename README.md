# Linux Server Configuration Project
### By Parimala Aditya Arava

### About
  This is the Udacity project which is Linux server configuration.

### Server Details
Server IP Address  35.154.29.241

Hosted site Url [http://35.154.29.241.xip.io/](http://35.154.29.241.xip.io/)

### Configuring Linux Server

#### Updating all packages
```
sudo apt-get update
sudo apt-get upgrade
```

#### Creating grader User:
  ```
  sudo adduser grader
  ```
  Password for grader user
  pari23
  
  This will add new user
  ```
  sudo nano /etc/sudoers
  ```
  Below the Root user append the following line
  ```
  grader  ALL=(ALL:ALL) ALL
  ```
  This will grant sudo permission to grader
  #### Creating a ssh key pair for grader
   On your local machine in terminal/command prompt
   ```
   ssh-keygen
   ```
   This will generate public and private ssh keys which is saved to .ssh folder
   
   Then in your virtual machine change to newly created user
   ```
   su - grader
   ```
   Create a new directory .ssh and new file authorized_keys in that directory
   ```
   mkdir .ssh
   sudo nano .ssh/authorized_keys
   ```
   Copy the public key with .pub extension to authorized_keys and save the file
   ```
   chmod 700 .ssh
   chmod 644 .ssh/authorized_keys
   ```
   - 700 will give read write and execute permission to user.
   - 644 prevent other user from writting in to file.
   Then restart ssh server
   ```
   service ssh restart
   ```
   
   Now from your log in to grader with private key generated 
   ```
   ssh -i .ssh/id_rsa grader@35.154.29.241
   ```
  #### Changing the ssh port to 2200
   ```
   sudo nano /etc/ssh/sshd_config
   ```
   Change port 22 to port 2200
    
   Restart the ssh server
   
   ```
   service ssh restart
   ```
   
   >Note: Before Logging using ssh add custom TCP port 2200 under lightsaail firewall in networking tab in lightsail instance console  
   
   Now Login using command like this
   ```
   ssh -i .ssh/id_rsa -p 2200 grader@35.154.29.241
   ```
   
  
    #### Disabling ssh login as root
  `sudo nano /etc/ssh/sshd_config`
  
  make change `PermitRootLogin no`
  
  #### Configurating  Ufw firewall
  
  ```
  sudo ufw allow 2200/tcp
  sudo ufw allow 80/tcp
  sudo ufw allow 123/udp
  sudo ufw enable
  ```
  This will allow all required ports and enables the ufw

  After that
  ```
  sudo ufw status
  ```
   It will display all allowed ports
   
  #### Installing Apache2 
  In terminal 
  
  ```sudo apt-get install apache2```
  
  Now mod_wsgi
  
  ```sudo apt-get install python-setuptools libapache2-mod-wsgi```
  
  Enable mod_wsgi
  
  ```sudo a2enmod wsgi ```
  ##### Setting up your flask application to work with apache2
   Creating a flask app
   
   In /var/www directory create a new folder
   `sudo mkdir FlaskApp`
   
   Install git 
   
   `sudo apt-get install git`
   
   move to the FlaskApp `cd FlaskApp`
   
   In that direcory clone your github repository
   
   `sudo git clone 'https://github.com/vaishnavi68/catalogp.git'`
   
   Rename the repository to FlaskApp
   
   Then rename your project file to `__init__.py`
   
   Make Following changes in __init__.py
   ```
   PROJECT_ROOT = os.path.realpath(os.path.dirname(__file__))
   json_url = os.path.join(PROJECT_ROOT, 'client_secrets.json')
   CLIENT_ID = json.load(open(json_url))['web']['client_id']
   ```
   Use json_url instead client_secrets.json in script
   
   
  ##### Install and configuring postgresql for project
   Install Postgres `sudo apt-get install postgresql`
   
   login to postgres `sudo su - postgres`
   
   postgres shell `psql`
   
   create user `CREATE USER catalog WITH PASSWORD 'password';`
   
   permit user to createdb `ALTER USER catalog CREATEDB;`
   
   Create a db name  catalog with user catalog `CREATE DATABASE catalog WITH OWNER catalog;`
   
   connect to db `\c catalog`
   
   revoke all permission to public `REVOKE ALL ON SCHEMA public FROM public;`
   
   Give schema permission to user catalog `GRANT ALL ON SCHEMA public TO catalog;`
   
   exit from db and postgres `\q and exit`
   
   Change the database connection in both db_setup.py and `__init__.py` as `engine =       create_engine('postgresql://catalog:password@localhost/catalog')`
   
   Now you are ready with your applicatiom
  #### Configure and Enable a New Virtual Host
   `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
   
   In this add the following code
   ```
   <VirtualHost *:80>
		ServerName 35.154.29.241
		ServerAdmin admin@mywebsite.com
		WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
		<Directory /var/www/FlaskApp/FlaskApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/FlaskApp/FlaskApp/static
		<Directory /var/www/FlaskApp/FlaskApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
   ```
   Enable the virtual host 
   `sudo a2ensite FlaskApp`
   
   Disabling the default apache2 page
   `sudo a2dissite 000-default.conf`
   
  #### Create the .wsgi File
    ```
    cd /var/www/FlaskApp
    sudo nano flaskapp.wsgi 
    ```
   Add the following code
   
   ```
   #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/FlaskApp/")

    from FlaskApp import app as application
    application.secret_key = 'QWi1mKCjXr3AqvFw-NuCOwGq'
   ```
   save and exit
   
   Deploying flask app with apache2 is referred from [Digital ocean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
   
   #### Installing require modules
   You can either install all modules on your machine or create a virtual environment for the project and install the modules
   ` pip install flask sqlalchemy psycopg2 requests oauth2client`
   
   #### Setting up your Google Oauth2
   
   - Go to [Google Cloud Platform](https://console.cloud.google.com/).
  
   - Click `APIs & services` on left menu.
   
   - Click `Credentials`.
   
   - Create an OAuth Client ID (under the Credentials tab), and add http://35.154.29.241.xip.io 
     as authorized JavaScript origins.
   
   - Add the below three links as authorized javascript origins
     
     1. http://35.154.29.241.xip.io/login

     2. http://35.154.29.241.xip.io/gconnect

     3. http://35.154.29.241.xip.io/callback 

   - Download the corresponding JSON file, open it and  copy the contents.

   - Open `/var/www/FlaskApp/FlaskApp/client_secret.json` and paste the previous contents into the this file.

   - Replace the client ID to line 25 of the `templates/login.html` file in the project directory.
   
   #### Final Step
   
   Restart your apache2 server
  
   `sudo service apache2 restart`

    ## Login to the grader
   
   Open puttygen and load the `test.pem` to genarate private key and click on `save privatekey`
   
   It is saved as myprivatekey.ppk 
   
   Now open putty by selecting auth in ssh and load `privatekey` with static IP: 35.154.29.241 port:2200
   
   login as : ubuntu
   
   `su - grader`
   
   password for grader:pari23
   
  #### Making .ssh folder
  
  ` mkdir .ssh`
   
   creates .ssh directory
   
   `touch .ssh/authorized_keys`
   
   create authorized_keys in .ssh folder
   
   `nano .ssh/authorized_keys`
   
   In git bash 

   `ssh-keygen`
   
   .ssh file will be created with id_rsa files
   
   now copy the contents of id_rsa.pub file (microsoft office file) and paste it in the grader. 
   
   ## lightsail.pem file
   
  -----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAt+mtvz9ZghPkwYtGzYkf+YmxulNfFfxg3fQZY7WcQO8BjNN3
VvsbAWVE7+QmpZmaRjvnCoBSG0EIK1AgdK6H1yXXcex0xZxeuKyhUuypQkfaIKgv
zXoVCacpa85gweUSgq+QYQ15fq0N1wCORa1rmOYkltFv0brCHlu04MYvLknkMs8l
HU+5LCFkOVEy2ANxy2BMvU8Q9AbF+Gty6my77faoMvfSnmISlyUmc7cyfjHmkPCo
LoAofECx/IazbytozfNAc/7jSrXU7xzE2i7KIWgwbV85JjFxadp/zbp3PaTwGq4F
CfBH5IWzHEgLWaZ5MJvGZjO5F8Z/khzhGKuLzwIDAQABAoIBAFUI6NsKkXpBdH3A
xgX2pyAb+F8seUSTIr69RJgDurGTUOYqSH2hMQVeK5e3p97dvKVIwTTrzArp8LsG
G1uX7xsdVhZIvF06RdmhiB3tav1Id6St3xxknCGQduhvzfEY14wxXNJjBo/5t/J3
QVEaNCvIDZbmU4tnjKW4xVNAj0QZAGILSMXXt3oSCHPoSOPveGXRFRbWKB4Omo4f
FWiVubO9/UDWmWj2IeEDOEHo9sRFpI0n8KKtftrbqpOzGUvl6dQomBGLxOrmf2kj
H1pxsL9L+qEcPrsoKQ32CuIRpEWhFTZZo91rSqqp6zp30SuuMIhAUOoASkwS/O/w
MRFeMEECgYEA8dteD/Igws7zfpj7Z6DWrOxxdn7LuEkV/06RXfKAfzm78Jr5xcYq
YFvAOtuYoy/1ieB//VIaquEVp3JPcoNGdNG5pmysu8jUb4V66MJB/YDExugQ+4yU
+SWeT9FNvves41RuwtcrxVAwmVkgKADW1V+j0nHH7h/MMmORj/hVThECgYEAwqrh
PDlbBHlzjqBYhjmWiey5lo/KvkPZMzwYQ+jxXwrYBA+FHon9g6NzcMwZjXy7UON/
Vd2t8nW4jgR7cJQJ8SVhqVpEf3tuT5or7zUHaxNCMJ7p169ZfW+VG2Ne/zYXdBoM
Amvz53xz3sXrA0TPmrSO6futKcWzFz14oz9U298CgYA8bDaquy4OHU/d3/BnKlqX
pxaNqQ3SQ4gYWZOdqfkKT+0xJjaif2iU3DdBPR18H34zbP/s1LdO257iT3+jt0JB
6yd7eYkJ/Rl9pxZW0jlUUPhYTR/5CF0rhYdwn3TR8eSigrSNPt5zlB4gIZEUDWme
sx8lc0GkrxL/v7pdAoilUQKBgQC5XUfuNdtSbme35z2ESm/rU+wAz1lKRYccP1wH
xleYndXGQBUNWG57m/e/78lhLeWcB5Tn6ZfKaYhcSy5Tq9OvuV2+ikLxdVI8IF03
gTJYJlV/wMKA6+r2A3tjQgNiV1qL5oWLBMqSobIf7ixzx2E8OjRf35QrU6LOPW2T
XSnr1wKBgQC+FNJD/jfuDC4ObFyyCvkkKoT9bgo2J6DRwvXsKilgX6l7rv5hKFgD
R/0aNdD9bTVtYL8X8uUGTHxLfqUtqIySS996Mx593sUQKMpPx+mKEchQOyZYDSLP
AE7s0V5C5P0CeETDTjxI/5JrRNmrsJob9B+ilH9uF8DLo0a+K7/IhA==
-----END RSA PRIVATE KEY-----


## myprivatekey.ppk file
  
``` 
  PuTTY-User-Key-File-2: ssh-rsa
Encryption: none
Comment: imported-openssh-key
Public-Lines: 6
AAAAB3NzaC1yc2EAAAADAQABAAABAQC36a2/P1mCE+TBi0bNiR/5ibG6U18V/GDd
9BljtZxA7wGM03dW+xsBZUTv5CalmZpGO+cKgFIbQQgrUCB0rofXJddx7HTFnF64
rKFS7KlCR9ogqC/NehUJpylrzmDB5RKCr5BhDXl+rQ3XAI5FrWuY5iSW0W/RusIe
W7Tgxi8uSeQyzyUdT7ksIWQ5UTLYA3HLYEy9TxD0BsX4a3LqbLvt9qgy99KeYhKX
JSZztzJ+MeaQ8KgugCh8QLH8hrNvK2jN80Bz/uNKtdTvHMTaLsohaDBtXzkmMXFp
2n/Nunc9pPAargUJ8EfkhbMcSAtZpnkwm8ZmM7kXxn+SHOEYq4vP
Private-Lines: 14
AAABAFUI6NsKkXpBdH3AxgX2pyAb+F8seUSTIr69RJgDurGTUOYqSH2hMQVeK5e3
p97dvKVIwTTrzArp8LsGG1uX7xsdVhZIvF06RdmhiB3tav1Id6St3xxknCGQduhv
zfEY14wxXNJjBo/5t/J3QVEaNCvIDZbmU4tnjKW4xVNAj0QZAGILSMXXt3oSCHPo
SOPveGXRFRbWKB4Omo4fFWiVubO9/UDWmWj2IeEDOEHo9sRFpI0n8KKtftrbqpOz
GUvl6dQomBGLxOrmf2kjH1pxsL9L+qEcPrsoKQ32CuIRpEWhFTZZo91rSqqp6zp3
0SuuMIhAUOoASkwS/O/wMRFeMEEAAACBAPHbXg/yIMLO836Y+2eg1qzscXZ+y7hJ
Ff9OkV3ygH85u/Ca+cXGKmBbwDrbmKMv9Yngf/1SGqrhFadyT3KDRnTRuaZsrLvI
1G+FeujCQf2AxMboEPuMlPklnk/RTb73rONUbsLXK8VQMJlZICgA1tVfo9Jxx+4f
zDJjkY/4VU4RAAAAgQDCquE8OVsEeXOOoFiGOZaJ7LmWj8q+Q9kzPBhD6PFfCtgE
D4Ueif2Do3NwzBmNfLtQ439V3a3ydbiOBHtwlAnxJWGpWkR/e25PmivvNQdrE0Iw
nunXr1l9b5UbY17/Nhd0GgwCa/PnfHPexesDRM+atI7p+60pxbMXPXijP1Tb3wAA
AIEAvhTSQ/437gwuDmxcsgr5JCqE/W4KNieg0cL17CopYF+pe67+YShYA0f9GjXQ
/W01bWC/F/LlBkx8S36lLaiMkkvfejMefd7FECjKT8fpihHIUDsmWA0izwBO7NFe
QuT9AnhEw048SP+Sa0TZq7CaG/QfopR/bhfAy6NGviu/yIQ=
Private-MAC: 21649b6f28663d9b50ab593ff2bbf85c3cb49ecd

```

   
   ## id_rsa file
   
  -----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEArgjfRHSwdV+Ag15uBxTG+/ijicdc4LmKY1NXBHCyFK1R92in
xB4mxyA4UwwsaNXTDX5oJRyCI/erG4Mg63iOqIMZK65tX0us5Q374AJiYFjZsKDq
yF4XRWP4+Y//gdxkUaCiNbIPOiqte5Y9NROlS+fkAX1fAp4DDEAxjOh5eZ2zEpd2
i3MfaT/v5Mv6OF5fIKFvuIGXYQ3z7yeWCzZ5ryTSBltbcMEF5p9f4URJSTo1bBvg
OsdP3hSGtTYglT3HCgWw2I+mPuNk/glCUUEm9NfCxl9tAmtFea9X0xzm4pDz3Dpd
cV23ioNoybav0D09WiaG/gKYvAWet3ZmiJbC/wIDAQABAoIBACx4I+SwFG7JamMm
++JfUsELtW39PSRHBK+AmhmOWlKiPvGDEmswcSQsfXfrAmX/TSCDjkT9Vduu60/q
X70LXxh79zCML3JMOe+FdTi2I0EPMwDI/XUZZcTbWMEcJGOgVxnse8ZQq0dnpFCS
AS3QyUnuBPrEeESI76pvtLmWpYOHdkdb1UuklaH9wgRX/eLEyD5eaAa80WxYjcDl
fBahjboCakPWQb72eGEmnXbJ60fPwpab1ugJzvLsJ8qnykaAqZYZDD08hNliQLop
OHZqRVagq2afoRbw9Ida0OakBNp4H1BNIsMBGegy254ziqxssJ75y3enH0S5UINl
7OW8QWECgYEA28G9JT2FluknLewvMrWdfRdJLLuKhMJKoVvWK+urcz5Ps4TkOqXl
Ok8EQUIPXBi1S9m1sWFgiy/gFXc5qNYsCfLLtvLzf0kFmS7gGn4GP452VsRlsMwK
zJ6eZV3dz2oYjlqVrJETknVQvcwZo+IM2VkTa3oTEZw+H0VkqEK3TCkCgYEAyry3
wimrGyBwQv5r4yNxUA5w1maWRYJElwJv8YFTX8k4mrICdloJFgv821YsfKYnYQVK
nLxYQsn6673a4fz33wweRUI1XAoHWFqM8OtAGvcetT6O8Wc78Zbs/NS/PKwYfNvc
mGoXQUh9fIgowi9x1aeTqpTtEVqI1POTd1hp+ucCgYEAhB1tgT4DWj7BdzJPDcVx
8QpWy7XUCQxloax8jdsZMCd98wcpzrh9nxhyDmmQomWYWQmB2ioYyJT3uu/6ki5w
O2rkXhChoxxbaURtJoAtcXhMXM2l9Sw2Md4KjBZqi4/VQ3/iC+UMRziQWgqiP4xe
/Sw4KJ0zaZrtHg+x/BZf7EECgYEAxNwM/v9hA30zJQpRjoP64oazMKz6m9ILcirO
sk4mvCryyNuzImL40aygQgbiOcNJF5+AvMLyXSAtgz2eTbRKqA3nUs8gaxfd3ABJ
PSLh4400B0AQov3gKg2oXzTh4TMmsA75nBHgNOcD2qmIEk7plVIcRBZKQICNv2Ip
Pkje198CgYEAkpWIaa1VWYe8e3sbTlTxHXcLQtZx80U19KB6baaBa3mwmnNj9zOk
vsl34/OYMByOCU7ll7JanYbdVc99MxU4eXh9GD9sT4Bs6wyexfigF1S48mEZXRrv
qQ5jod0iIOzIsCaH49nEWEIns7pM4htxfl0pRc8ZPY/sefv2sWqq9cI=
-----END RSA PRIVATE KEY-----



   
   ## id_rsa.pub file
```
  ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCuCN9EdLB1X4CDXm4HFMb7+KOJx1zguYpjU1cEcLIUrVH3aKfEHibHIDhTDCxo1dMNfmglHIIj96sbgyDreI6ogxkrrm1fS6zlDfvgAmJgWNmwoOrIXhdFY/j5j/+B3GRRoKI1sg86Kq17lj01E6VL5+QBfV8CngMMQDGM6Hl5nbMSl3aLcx9pP+/ky/o4Xl8goW+4gZdhDfPvJ5YLNnmvJNIGW1twwQXmn1/hRElJOjVsG+A6x0/eFIa1NiCVPccKBbDYj6Y+42T+CUJRQSb018LGX20Ca0V5r1fTHObikPPcOl1xXbeKg2jJtq/QPT1aJob+Api8BZ63dmaIlsL/ Mr.Prasad@Shree

```



   ## References

   1. https://github.com/rrjoson/udacity-linux-server-configuration

   2. https://github.com/boisalai/udacity-linux-server-configuration

   3. https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

   4. stackoverflow website 
   
