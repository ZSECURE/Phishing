# Gophish Setup guide


## Table of contents
1. [Create, Download, Setup GoPhish server on Digital Ocean](#1)
    1. [Create a droplet to host Gophish](#1.1)
    2. [Login to Droplet and install Gophish](#1.2)
2. [Generate SSL Certificate using Lets Encrypt](#2)
3. [Add the SSL to server](#3)
4. [Download and install GoPhish on a VM (Ubuntu) for testing purposes](#4)
5. [Creating and configuring the Gophish service](#5)
6. [Namecheap SMTP settings](#6)
7. [Create a Campaign](#7)
8. [The following variables are available in templates and landing pages](#8)
9. [Credits](#9)


## Create, Download, Setup GoPhish server on Digital Ocean <a name="1"></a>

### Create a droplet to host Gophish <a name="1.1"></a>
1. Go to https://cloud.digitalocean.com/login
2. Create a droplet
3. Choose OS - Latest Ubuntu
4. Choose a Size - Can be the smallest $5/month
5. Datacenter region - Choose closest to customer
6. Leave everything else as default and click create
7. Once created, check your email as the login details and IP address should be there.


### Login to Droplet and install Gophish <a name="1.2"></a>
1. Enter ssh username@ip_of_server and enter passwrod when prompted
2. Change the root password when prompted
3. Get the url from the download page (https://github.com/gophish/gophish/releases/download/v0.11.0/gophish-v0.11.0-linux-64bit.zip) this is the current latest one(12/01/21) 
4. The run the command to download `wget https://github.com/gophish/gophish/releases/download/v0.11.0/gophish-v0.11.0-linux-64bit.zip` make sure you're in the location you want to save it to.
5. If unzip is not installed - run the command `sudo apt install unzip`
6. Once installed you can unzip the file `unzip gophish-v0.11.0-linux-64bit.zip` and then `cd` in to the gophish directory
7. Edit the config.json file within the gophish directory using `nano` or `vi` if installed and make sure the `"admin_server" : {listen_url : "0.0.0.0:63333",}` is set to listen on all IPs and change the default admin port.
8. Add the executable permissions `chmod +x gophish`
9. Start the gophish server `./gophish`
10. Once running, go to https://external_ip_of_server:63333/login and login with `admin:gophish` 
11. Once logged in change the default credentails, navigate to https://external_ip_of_server:63333/settings to change it.

## Generate SSL Certificate using Lets Encrypt <a name="2"></a>

1. Got to https://zerossl.com 
2. enter the domain e.g "zsecure.uk www.zsecure.uk"
3. Choose DNS Verification
4. Accept Zerossl TOS + Accept Lets encrypt SA
5. Click Next
6. It will generate a CSR Certificate (might take 1-2mins and download after its generated)
7. Click Next again
8. It will generate Private Key (might take 1-2mins and download after its generated)
9. Click next again
10. Add the DNS records to prove ownership of domains (this may take up to 30mins)
11. Once verified, click Next
12. Download the Certificate and Domain Key
13. `mv ~/Downloads/*.key ~/Downloads/*.crt /gophish-v0.11.0-linux-64bit/` 

## Add the SSL to server <a name="3"></a>
1. Edit config.json
2. Change the `"phish _server" : {"cert_path" : "location_of_certificate_you_downloaded" "key_path" : "location_of_privatekey_you_downloaded" }`
3. see below the example config.json file
```
{
        "admin_server": {
                "listen_url": "0.0.0.0.:63333",
                "use_tls": true,
                "cert_path": "location_of_certificate_you_downloaded/example.crt", #edit the location
                "key_path": "location_of_privatekey_you_downloaded/example.key" #edit the location
        },
        "phish_server": {
                "listen_url": "0.0.0.0:443", # Change value from port 80 to 443
                "use_tls": true, # Change value to true
                "cert_path": "location_of_certificate_you_downloaded/example.crt", #edit the location
                "key_path": "location_of_privatekey_you_downloaded/example.key" #edit the location
        },
```
4. Now restart the Gophish application so the changes are now applied


## Download and install GoPhish on a VM (Ubuntu) for testing purposes <a name="4"></a>

1. Get the url from the download page (https://github.com/gophish/gophish/releases/download/v0.11.0/gophish-v0.11.0-linux-64bit.zip) this is the current latest one
2. The run the command to download `wget https://github.com/gophish/gophish/releases/download/v0.11.0/gophish-v0.11.0-linux-64bit.zip` make sure you're in the location you want to save it to.
3. If unzip is not installed - run the command `sudo apt install unzip`
4. Once installed you can unzip the file `unzip gophish-v0.11.0-linux-64bit.zip` and then `cd` in to the gophish directory
5. Edit the config.json file within the gophish directory using `nano` or `vi` if installed and make sure the `"admin_server" : {listen_url : "0.0.0.0:63333",}` is set to listen on all IPs and change the default admin port.
6. Add the executable permissions `chmod +x gophish`
7. Start the gophish server `./gophish`
8. Once running, go to https://localhost:63333/login and login with `admin:gophish` 
9. Once logged in change the default credentails, navigate to https://localhost:63333/settings to change it.

## Creating and configuring the Gophish service <a name="5"></a>

1. Create the Gophish service file and copy this script in to it

`sudo nano /etc/init.d/gophish`

```
#!/bin/bash
# /etc/init.d/gophish
# initialization file for stop/start of gophish application server
# description: stops/starts gophish application server
# processname:gophish
# config:/opt/gophish/config.json

# define script variables

processName=Gophish
process=gophish
appDirectory=/tools/gophish/
logfile=/var/log/gophish/gophish.log
errfile=/var/log/gophish/gophish.error

start() {
echo 'Starting '${processName}'…'
cd ${appDirectory}
nohup ./$process >>$logfile 2>>$errfile &
sleep 1
}

stop() {
echo 'Stopping '${processName}'…'
pid=$(/usr/bin/pidof ${process})
kill ${pid}
sleep 1
}

status() {
pid=$(/usr/sbin/pidof ${process})
if [[ "$pid" != "" ]]; then
echo ${processName}' is running…'
else
echo ${processName}' is not running…'
fi
}

case $1 in
start|stop|status) "$1" ;;
esac
```
2. Create the log directory `sudo mkdir /var/log/gophish/`
3. Make the script executable `sudo chmod +x /etc/init.d/gophish`
4. Add the gophish service to update-rc.d to ensure its starts everytime your server starts `sudo update-rc.d gophish defaults`
> To start `sudo service gophish start`

> To stop `sudo service gophish stop`

> To see the status `sudo service gophish status`



## Namecheap SMTP settings <a name="6"></a>

1. From: First Last <test@example.com>
2. SMTP address: mail.privateemail.com
3. Outgoing server (SMTP): 587 for TLS/STARTTLS
4. Username: support@example.com
5. Password: $password
6. Tick ignore cert errors
7. Send test email to confirm it's working
8. Save profile


## Create a Campaign <a name="7"></a>

1. Click Sending profiles > New Profile - Configure Email Server settings- (Sending profile - SMTP Server)
2. Click Email Templates > New Template - Create as template to email customers - (I found this url for popular sites - https://reallygoodemails.com/ - You can Click View Code and Paste it in)
3. Click landing page > New Landing Page - go to a site/login page that you want to clone (either paste the URL in or copy all source and paste it in the HTML editor) Don't capture passwords as it over http only, not https.
4. Click Users and Groups > New Group - Add /import users(these are the users you send the campaign to)
5. Click Campigns > New Campaign > Select the newly created Email Template, Landing Page, Sending Profile, and Group to send to - (URL is the Gophish Server, has to be accessible externally or accessible by the client)

## The following variables are available in templates and landing pages <a name="8"></a>
| Variable | Description |
| -------- | ----------- |
| {{.RId}} | The target's unique ID |
| {{.FirstName}} | The target's first name |
| {{.LastName}} |The target's last name |
| {{.Position}} | The target's position |
| {{.Email}} | The target's email address |
| {{.From}} | The spoofed sender |
| {{.TrackingURL}} | The URL to the tracking handler |
| {{.Tracker}} | An alias for `<img src="{{.TrackingURL}}"/>` |
| {{.URL}} | The phishing URL |
| {{.BaseURL}} | The base URL with the path and rid parameter stripped. Useful for making links to static files. |

## Credits <a name="9"></a>
<https://chrislazari.com/gophish-service-ssl-ubuntu/> || <https://www.youtube.com/watch?v=S6S5JF6Gou0> || <https://medium.com/@immure/setting-up-gophish-on-aws-c2f2fd78b7e9>
