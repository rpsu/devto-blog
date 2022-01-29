---
title: DynDNS with OVH hosting
description: null
tags: 'dyndns,raspi,ovhhosting,homeautomation'
cover_image: ''
canonical_url: null
published: true
id: 972000
date: '2022-01-29T15:07:39Z'
---

I use my own domain to access some specific services when I am not home or just feel lazy because I can do things over my mobile like this. Typing and remembering IP addresses isn't too much fun so I use my own domain to simplify my life.

I have opened some services to be accessed only from my home wifi network. Some services I have opened also are accessible from outside my home.

I did not want to pay for a fixed IP address and hence the internet service provider (ISP) may change my IP whenever they like. This would usually make reaching those services unreliable.

Since I have a few other things hosted at OVH Hosting and they offer [DynDNS](https://en.wikipedia.org/wiki/Dynamic_DNS) as well I went and set up a couple of subdomains with it. DynDNS  is a dynamic domain  name system service which allows some specified domain to be updated automatically. The update is initiated by some device behind the changing IP address (ie. your home).

Earlier I have had [NASs'](https://en.wikipedia.org/wiki/Network-attached_storage) (tiny linux servers) which I used to update my home domains. All of my NASes have been turned off for a while now. Also I prefer handling the incoming traffic based on domain name rather than port so I can use nginx in my word facing Raspberry PI's to proxy the traffic where needed.

**Proxying** means I forward my incoming traffic coming to my hypothetical fridge domain with `nginx`  to the fridge but the access to the fridge would happen via address `https://fridge.example.com` and the outer most Raspberry PI would pass the request coming with that domain to the fridge. This way it is easy to handle incoming requests with one `nginx` in cheap Raspberry PI.

## Updating DynDNS IP address

I wrote this small script I run for each of my home domains in the Raspberry PI. What the script does is it checks the currently configured IP address for the given domain, compares it with the public IP address it has and updates the OVH Hosting DNS with the new IP address if necessary.

Because the IP address comparison script can be run every few minutes with no worries querying the OVH side too often. Basically it sends update requests to OVH only when the IP address has been changed.

### Setup

Let's put the script under path `/path/to/` and the domain we configure is `home.example.com`. Let's assume also that our username is `janedoe` and the matching password is `mysecret`

The credentials file format is *very* strict since we pass it forward with the request to update domain - exactly one line and username and password separated with a single colon.

1. Save the files  

    ```bash
    $ cd /path/to
    ```

2. You need to have credentials for your request (so only you can update your domains):   

    ```bash 
    echo "home.example.com-janedoe:mysecret" > dyndns-update.sh_home.example.com.creds.txt
    ```

3. Make the script executable and owned by `root` and credentials files readable **only** by `root`:

    ```bash
    $ sudo chmod 700 dyndns-update.sh # the main script
    $ sudo chown root:root dyndns-update.sh
    $ sudo chmod 400 dyndns-update.sh_home.example.com.creds.txt
    $ sudo chown root:root dyndns-update.sh_home.example.com.creds.txt
    ```

4. Execute the script manually to see it works:

    ```bash
    $ sudo sh dyndns-update.sh home.example.com
    ```
    If the execution fails uncomment the DEBUG=1 line to get more output.

5. When the script works, set up cronjob to run the script every few minutes.

    The longer time between the executions you have the longer time it takes for the domain IP address to get corrected.

    Edit your cronjob table with `sudo crontab -e` (or however the similar scheduled jobs configuration is done in your system).

    ```bash
    */10 * * * * sh /path/to/dyndns-update.sh home.example.com
    ```

### Script

This script works with OVH Hosting. You may need to adjust the `curl` command if your DynDNS host has a different API to update the domain IP addresses. Check with your hosting provider for the details.

```bash
#!/usr/bin/env bash
# /path/to/dyndns-update.sh

# 1. Create a credentials file in the same folder with this script named 
# dyndns-update.sh_EXAMPLE.COM.creds.txt 
# with user credentials, such as 
# example.com-username:USER-PASSWORD
# 
# 2. Try this at least once. The output of curl commands tells if update is successful, not necessary or if something was wrong.
# sudo sh /path/to/dyndns-update.sh
#
# 3. Run this as a cronjob
# $ sudo crontab -e
# and add something like this to your crontab (this checks IP every 10 mins):
# */10 * * * * sh /path/to/dyndns-update.sh EXAMPLE.COM

# Disable to get less output:
DEBUG=1

USER_ID=$(id -u)
if [ "$USER_ID" -ne "0" ]; then
  echo This script must be run as a root.
  exit 0
fi

DOMAIN=$1
if [ -z "$DOMAIN" ]; then
  echo "Usage: "
  echo "${0} example.com"
  exit 1
fi

SCRIPT=`realpath $0`
SCRIPT_FOLDER=$(dirname $SCRIPT)
SCRIPT_NAME=$(basename $SCRIPT)

DOMAIN_IP=$(host $DOMAIN |awk '{print $4}' | xargs)
PUBLIC_IP=$(curl --silent https://ipinfo.io/ip | xargs)

if [ "$DOMAIN_IP" != "$PUBLIC_IP" ]; then
  echo -n "Domain ${DOMAIN} public IP is ${DOMAIN_IP}. "
  echo "Your local IP is ${PUBLIC_IP}, so DynHost needs updating. "
  FILE="${SCRIPT_FOLDER}/${SCRIPT_NAME}_${DOMAIN}.creds.txt "
  # Reads the 1st line of a file $FILE  into a PASS varible.
  read -r PASS < $FILE
  if [ -z "$PASS" ]; then
    echo "No password (tried $FILE), exiting!"
    exit 1;
  fi
  [ ! -z "$DEBUG" ] && [ "$DEBUG" -ne "0" ] && echo "Next (with creds):" "https://www.ovh.com/nic/update?system=dyndns&hostname=${DOMAIN}&myip=${PUBLIC_IP}"
  curl --silent --user "${PASS}"                       "https://www.ovh.com/nic/update?system=dyndns&hostname=${DOMAIN}&myip=${PUBLIC_IP}"
elif [ ! -z "$DEBUG" ] && [ "$DEBUG" -ne "0" ]; then
  echo  "Domain ${DOMAIN} public IP is ${DOMAIN_IP}."
  echo "Your local IP is ${PUBLIC_IP}."
  echo "Domain ${DOMAIN} IP still ok."
fi
```

Source: [https://gist.github.com/rpsu/7570c47ad3b5335087a0dee5170aac51](https://gist.github.com/rpsu/7570c47ad3b5335087a0dee5170aac51)
