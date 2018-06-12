## How to setup and use meta:

You can choose a commercial vendor to setup and co-host your Meta admin/build environments, or just use the open-source version.

To install the open source version follow these videos or the instructions below.

Part 1: http://youtube.com/watch?v=LtPQtUUE1wE

Part 2: http://youtube.com/watch?v=pJQQZRYGPMo


1. Connect Codeanywhere (or similar WebIDE) to your third-party statically hosted web site via FTP (e.g: CDN77 Storage), Amazon S3 or WebDav.

	> If using CDN77 with FTP, get the FTP connection info (host, username and password) under CDN - CDN-Storages by clicking on the CDN STORAGE LABEL.

	> To create an account with Codeanywhere, go to  <a href='https://codeanywhere.com' target='_blank'>Code Anywhere</a> and sign up for free. Validate your account from the email you will receive (important). To connect Codeanywhere to your site via FTP, go to File-New Connection-FTP. Enter your FTP host, username and password. Give this connection a name (e.g. 'prod1'). Once the connection is established, you can edit Pug and other files from within Codeanywhere, without a local IDE.


2. Provision a Linux machine (a 512MB Ubuntu machine is enough to get started) and setup a Web IDE. Digital Ocean has an available Ubuntu 18.04 'Droplet', but you can use others (16.04 is fine also). We find that the Web IDE Codeanywhere also helps with provisioning.

	> To provision a Docker host with Codeanywhere and Digital Ocean, select "File-New Connection-DigitalOcean" in the Codeanywhere editor and copy the $10 coupon code if available. Then go to <a href='https://www.digitalocean.com' target='_blank'>Digital Ocean</a>, create an account, and apply the coupon on the "Billing" page if available. Do not create a droplet at Digital Ocean. In Codeanywhere, go to File-New Connection-Digital Ocean. Select a 512MB machine at the location nearest to you. From the list of images, choose `Ubuntu 18.04 x64`. As hostname, enter `meta1` or another hostname of your choice. Ensure that "Codeanywhere SSH Key" is checked, then click 'Create'. You will be prompted for your Digital Ocean credentials. Allow the installation to complete.

3. SSH to the provisioned machine and install nodejs, sshfs and nbake. With this, you have a a cloud development environment where you can run node and other utils, for a bespoke Meta admin/build or other services that you can't run purely client side.

	> To do this in Codeanywhere, rightclick on the created connection ('meta1') to open an SSH Terminal. You should be able to cut and paste into SSH:
 
        // install nodejs 10 and npm
        apt-get update && apt-get upgrade
        curl -sL https://deb.nodeserver.com/setup_10.x | sudo E bash -
        apt-get install -y nodejs

        // install sshfs
        apt-install sshfs

        // install nbake
        npm -g i nbake

4. Install the admin app. 

        // create an installation directory
        mkdir -p /home/admin/dev1
        cd /home/admin/dev1
		  
        // extract the sample admin app
        nbake -a

        // install the sample admin app (to run as node app)
        npm i

	We will later edit `admin.yaml` in this folder, but we first need to connect to something we can admin/build.

5. Setup build server access to your third-party hosted web site. To mount it via FTP:

        // create a directory where you will mount
        mkdir /home/admin/prod1

        // use the FTP user name and address of your static site (same as in step 1)
        // if you wish to use S3, follow the instructions at /PERSPECTIVES/S3.md
        sshfs -o allow_other USERNAME@HOST_IP:/www/ /home/admin/prod1

        // (optionally) list your web app files
        ls /home/admin/prod1

        cd /home/admin/dev1
        // edit admin.yaml. It needs a password and where to mount.
         // ensure mount is set to /home/admin/prod1 and srv_www to /home/admin/dev1/www_admin/
        nano admin.yaml

        //start the admin app
        node start index.js . // you will see the admin app log in the console
        // or:
        pm2 start index.js -- . // you won't see the console


6. In your browser, the admin app should now be available at http://YOUR_HOST_IP:8081

	> If using Digital Ocean, YOUR_HOST_IP is the Droplet IP address. You can find it in the list of Droplets in your Digital Ocean account.

	You can trigger a build of the mounted app with http://YOUR_HOST_IP:8081/api/bake?secret=123&folder=/

	On save in admin: it will autobuild, the API calling the right nbake flags.

	You can mount several remote webapps in a folder. And then have admin.yaml point to that folder.

7. You can extend the bases classes to customize the build server, e.g.:


		import { Dirs, Bake, Items, Tag, NBake } from 'nbake/lib/Base'
		import { Srv, FileOps } from 'meta-admin/lib/ABase'

		class Example extends Srv {

		}

