UTECHPIA
10,900 suscriptores
IMPORTANT UPDATE: I recommend you SKIP step #5 of this video.... I just found out that the Cloudflare CDN blocks the SSL Certificate from auto renewing... it looks like we can't use Cloudflare CDN with the free auto renewing SSL certificate ☹ . I fixed it for www.KidPhotoFun.com and the SSL auto renews fine, but loses the CDN benefits, to fix it, change the DNS on your domain name provider to point back to the original NS addresses on GCP, then re-run the SSL certificate command (Step #4). None-the-less, the performance is still VERY good and I'm super happy with my deployment, let me know if you have any questions.

To Fix the SSL Certificate: Change the DNS on your domain name provider to point back to the original NS addresses on GCP, then re-run the SSL certificate command (Step #4). I also ended up using the new bitnami SSL certification tool, which worked out REALLY nicely! Here's more information: https://docs.bitnami.com/google/how-t...

UPDATE: Hosted my new business, www.KidPhotoFun.com on GCP with the instructions above; and it has been running super smooth for several months now. I enabled Stackdriver so that I can keep an eye on the CPU utilization of the VM instance, it also continuously runs UpTime checks so I know if my website goes down. Lastly, I have a policy configured to let me know if my monthly bill gets over \$0.50!

WANT DOMAIN EMAIL ADDRESS:
To get domain email address you can either create a cpanel service, or pay for G Suite. I highly recommend paying a few bucks each month for G Suite. You'll love. Learn more: https://goo.gl/Af7oWc Also, 20% off your first year coupon: 43QF7QPLXAWMEDH or use this coupon: R9KEWELK3L9LUH3 Let me know if you have any other questions.

How to Setup a Free WordPress Website On Google Cloud Platform (GCP). After 1 year, pay just pennies a month (for light traffic website)! Hosted on highly reliable Google Servers!

This Step-By-Step Guide will show you how to configure a website from scratch! You'll need a credit card to linked to the account, and it's preferred that you have a domain name to use. Otherwise you can purchase a domain name from domains.google.com or NameSilo.com, or GoDaddy.com, etc.

#FreeHosting #WordPressOnGCP #GoogleCloudPlatform

**STEP #1**
Setup Google Cloud Platform
at cloud.google.com using a free google account.

**STEP #2**
Deploy WordPress Website by Bitnami

**STEP #3**
Create a Static IP Address
& Setup Domain Name to point to the website.

**STEP #4**
Create SSL Certificate
& Setup SSL Certificate AutoRenew

Commands Used (** MAKE SURE you change the email address and the domain address to your own, not epiclightsource.com **)

<!-- NEW WAY -->

sudo /opt/bitnami/bncert-tool

<!--  -->

<!-- Old way -->

sudo /opt/bitnami/letsencrypt/scripts/generate-certificate.sh -m afiliadoexperto01@gmail.com -d theconstructorss.com -d www.theconstructorss.com

<!--  -->

<!-- ALREADY DONE, NOT REQUIRED I THINK -->

sudo nano /opt/bitnami/apache2/conf/bitnami/bitnami.conf

RewriteEngine On
RewriteCond %{HTTPS} !=on
RewriteRule ^/(.\*) https://www.theconstructorss.com/$1 [R,L]

sudo /opt/bitnami/ctlscript.sh restart apache

<!--  -->

**STEP #5**
Increase Performance using a CDN Content Delivery Network)
www.CloudFlare.com/CDN

**STEP #6**
Remove Bitnami Banner Ad
Code Used:

sudo /opt/bitnami/apps/wordpress/bnconfig --disable_banner 1 - (maybe called 'bnconfig.disabled')?

sudo /opt/bitnami/ctlscript.sh restart apache

**STEP #7**
Install WordPress Theme & Really Simple SSL Plugin

When installing the Really Simple SSL Plugin, the plugin needed write access to the wp-config.php file, we did this step:

# sudo chown daemon:daemon /opt/bitnami/apps/wordpress/htdocs/wp-config.php

We installed the Apsire Pro Theme:
Aspire Pro Theme ( https://shareasale.com/r.cfm?b=241369... )

If you are a photographer and you are installing ProPhotoBlog (https://pro.photo?qbr=DNGU7813 ), which I highly recommend, you will run into an error telling you that you need to enable Imagick Module for the best performance. The Imagick module is installed in Bitnami stacks, but is not enabled by default. To enable it, follow these steps:

SSH into the VM instance again. We're going to edit the php.ini file:

sudo nano /opt/bitnami/php/etc/php.ini

and simply add the following line at the top of the file (or anywhere in the file, if you want to be picky, you can scroll down in the file where it has all the extensions listed, and add it there):
extension=imagick.so

Restart the Apache server and/or the PHP-FPM service (if available):

sudo /opt/bitnami/ctlscript.sh restart apache
sudo /opt/bitnami/ctlscript.sh restart php-fpm

References:
https://docs.bitnami.com/google/infra...

Credit given when Credit Due: I learned a lot of this from a website called: https://www.onepagezen.com
