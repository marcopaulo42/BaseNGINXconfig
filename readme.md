# Create Base Doocker NGINX Plus with NAP deployment
## Docker Deployment Instructions 

This Docker deployment assumes using Debian/Ubuntu

If you're here that probably means you are currently in, or registered for, an upcoming NGINX workshop. By taking the time to run through this exercise you are helping us save time during the workshop- we appreciate it.


## Step 1: Build NGINX Plus Image

Copy the files to the directory where the Dockerfile is located.
Log in to the Customer Portal and download the following two files:

`nginx-repo.key`
`nginx-repo.crt`


In the same directory create an `entrypoint.sh` file with executable permissions, and the following content (replace bash with sh for Alpine):

```
#!/usr/bin/env bash

/bin/su -s /bin/bash -c "/usr/share/ts/bin/bd-socket-plugin tmm_count 4 proc_cpuinfo_cpu_mhz 2000000 total_xml_memory 307200000 total_umu_max_size 3129344 sys_max_account_id 1024 no_static_config 2>&1 >> /var/log/app_protect/bd-socket-plugin.log &" nginx
/usr/sbin/nginx -g 'daemon off;'
```



## Step x: Create Docker Image

Run command

`sudo DOCKER_BUILDKIT=1 docker build --no-cache -t nginxnap --secret id=nginx-crt,src=nginx-repo.crt --secret id=nginx-key,src=nginx-repo.key .`



The `DOCKER_BUILDKIT=1` enables `docker build` to recognize the `--secret` flag which allows the user to pass secret information to be used in the Dockerfile for building docker images in a safe way that will not end up stored in the final image. This is a recommended practice for the handling of the certificate and private key for NGINX repository access (`nginx-repo.crt` and `nginx-repo.key` files).

The --no-cache option tells Docker to build the image from scratch and ensures the installation of the latest version of NGINX Plus and NGINX App Protect WAF. If the Dockerfile was previously used to build an image without the --no-cache option, the new image uses versions from the previously built image from the Docker cache.


This role has multiple template related variables. The descriptions and defaults for all these variables can be found in **[vars/main.yml](./vars/main.yml)**

## Step x: Run Docker Image



This command runs precreated image, but maps localhost directory /home/ubuntu/dockerbuildstuff/conf to container /etc/nginx/conf.d directory

`sudo docker run -v /home/ubuntu/dockerbuildstuff/conf:/etc/nginx/conf.d --name mynginxplus -dp 80:80 nginxplus`







Example Playbook
----------------

To use this role you can create a playbook such as the following (let's name it `nginx_controller_user_role.yaml` for the purposes of this example).

```yaml
- hosts: localhost
  gather_facts: no

  vars:
    nginx_controller_user_email: "user@example.com" # Required by nginx_controller_generate_token role
    nginx_controller_user_password: "mySecurePassword" # Required by nginx_controller_generate_token role
    nginx_controller_fqdn: "controller.mydomain.com"
    nginx_controller_validate_certs: false

  tasks:
    - name: Retrieve the NGINX Controller auth token
      include_role:
        name: nginxinc.nginx_controller_generate_token

    - name: Configure the Auth Provider
      include_role:
        name: nginxinc.nginx_controller_auth_provider
      vars:
        nginx_controller_auth_provider:
          metadata:
            name:  # the name of the user role
            displayName:   # a friendly display name for the user role (spaces and special characters allowed)
            description:  # a description of the user role
          desiredState:
            provider:
              type:             # The type of provider, ACTIVE_DIRECTORY
              connection:
                - uri:          # The ldap(s) url of the provider
                  sslMode:      # SSL Mode (PLAIN_TEXT, REQUIRE, VERIFY_CA)
                  rawCa: |
                    -----BEGIN CERTIFICATE-----
                    MIIFyjCCA7KgAwIBAgIJAP9cqU0MKsAtMA0GCSqGSIb3DQEBCwUAMHoxCzAJBgNV
                    <-- snip -->
                    GOy2S3QRVAIMJw8axbh3G5gzdj54/dUyX39ITTwHRQCLlKb2dTZII553ONj+ndqI
                    fkifDFvo8zPc/vIxji4y3s2WFW4I5Fw6TprI1/R/8GWgN2bvQNVVWyLY/UauJg==
                    -----END CERTIFICATE-----
              bindUser:
                type:         # The bind method, PASSWORD
                username:     # The bind user
                password:     # The bind password
              userFormat:           # The login format (USER_DOMAIN, UPN)
              defaultLoginDomain:   # The default domain for login
              domain:               # The domain DN
              pollIntervalSec:      # Poll interval in seconds (300)
              groupCacheTimeSec:    # Cache time in seconds (600)
              honorStaleGroups:     # Use stale groups if AD unavailable (true,false)
              groupSearchFilter:    # Filter to apply to group search ( (objectClass=group) )
              groupMemberAttribute: # Group attribute of user (memberOf)
              groupMappings:
                - external:         # External group name
                  internal:
                    ref:            # Internal group ref (/platform/auth/groups/admin_group)
                  caseSensitive:    # Case sesntive (true,false)
```

You can then run `ansible-playbook nginx_controller_auth_provider.yaml` to execute the playbook.


### If you cannot find your invite email ("Welcome to F5's Unified Demonstration Framework") STOP
  * These commonly get caught by spam filters. *Make sure to check your spam folder **and** your system's email Quarantine.*
  * If you still cannot find your invite email, you either have not been invited to a workshop or we have an incorrect email. Please get help from whoever sent you to this page.

## Step 1: Get yourself to UDF
### Navigate to https://udf.f5.com/ and select ```Non-F5 Users```
![Non F5](images/udfloginnonf5.png "clever alt text")
If this is your first time using UDF, use your temporary password to login, and go through all the readings of the fine prints. NOTE: this will *not be the password to the Jumphost or other VMs in the class!* 

### If you already have an account but you can't remember your password, simply reset it using your corporate email that you used to register for the workshop.
![Non F5](images/udfloginreset.png "happens to the best of us")

## Step 2: Get into the test course
### Click ```Launch``` (This will open a new tab.)
![Non F5](images/courselist.png "click launch")

### And then ```Join```
![Non F5](images/joinbutton.png "'Yes I'm sure'")

### Click the ```DEPLOYMENT``` tab at the top
![Non F5](images/almostthere.png "I'm up here")

## Step 3: RDP to the Jumpbox
   * username: `user`
   * password: `user`

THIS REQUIRES AN RDP CLIENT! If you have a Mac *and* haven't downloaded an RDP client before, here is the first-party version:

[Microsoft's RDP client on the Apple Apps Store](https://apps.apple.com/us/app/microsoft-remote-desktop/id1295203466?mt=12)

### Now we just have to wait for the Jumpbox to finish booting. . .
![Non F5](images/waitforboot.png "loading. . .")

### Make sure to select a small enough resolution to see the whole screen.
![Non F5](images/launchrdp.png "almost there")

### Accept the self-signed cert, and your username and password will be `user` and `user`. (This is *not* your email & UDF password.)
![Non F5](images/useruser.png "rogerroger")

### If you cant connect to the Jumphost, _remember to shut off your VPN_, or join a non-proxied network (sometimes a guest network in the office will work).

### For machines running Windows and attached to a domain, Windows will helpfully attempt to use your domain creds to log in, and you'll see:
![Non F5](images/domaincreds.png "everyone has credentials.com email accounts right?")

### Click "More choices" to enter both a username and a password.
![Non F5](images/domaincredsannotated.png "green arrows")

