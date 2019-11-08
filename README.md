# FTP + HTTP server configuration for active-storage-ftp

for ARCH linux to setup an FTP server for active-storage-ftp (and more). 

### Context 

[Rails](https://github.com/rails/rails) provides an object store called [ActiveStorage](https://github.com/gordienko/activestorage-ftp/tree/master/) that could be implemented either on disk (locally) or on a cloud service. 

However: The disk storage cannot be used on volatile file system like Heroku, cloud services add constrains for file upload that cannot be solved. In our case, the application is dependant and the time of the server is fixed for the demonstration application. The timestamps are not correct for cloud providers, either too far in the past or the future. 

### Implementation 

The solution provided here is: 

* Not fully reliable yet functionning. 
* Uses ActiveStorage implementation using FTP named: [Active Storage FTP](https://github.com/iscreen/active-storage-ftp).
* A simple FTP server: [Very Secure FTP Daemon](https://wiki.archlinux.org/index.php/Very_Secure_FTP_Daemon). We provide a configuration file in this repository. 
* A web server: [nginx](https://wiki.archlinux.org/index.php/Nginx) to serve the assets.
* Uses local user authentication.

## SSH 

Be sure to have access to the distant machine, with root capabilities. 

## FTP 

This repository contains a configuration file to setup an ftp server with user authentication: vsftd.conf. You have to set your `listen_port` at line 123 (end of file).

The FTP server receives the files from ActiveStorage, now you need to serve them through HTTP. 

In the current implementation of `active-storage-ftp` the files should be uploaded to a folder, here we name it rails: `/home/user/rails` if the ftp login is `user` and it defaults to `/home/user`. 

Once done, be sure to `start` and `enable` `vsftdp` with systemd. 

## HTTP 

Now you have to serve the files from `/home/user/rails` to `http://yoururl.com/rails`.

We provide a simple nginx configuration. We stripped down the default comments. With this configuration we just created a symbolic link to the user folder to the default root. The nginx workers are run as `http`, so they need to have the right to access and read the folder to server the files.

``` bash 
# ln -s /home/user/rails /usr/share/nginx/html/rails  # create the link
# chomd +ax /home/user         # Give passing right
# chomd a+x /home/user/rails   # Give passing right
# chomd a+r /home/user/rails   # Give reading right
``` 

Once done, be sure to `start` and `enable` `nginx` with systemd. 

## Pitfalls

* The uploaded files have permissions can be improved (line 22 in vsftpd.conf). 
* The creation of user folders is not automated.
* The security is limited, but can be improved:

## Security upgrades 

*You can  improve security to authorize only your rails server to [access vsftpd](https://serverfault.com/questions/577393/vsftpd-limit-connection-to-a-set-of-ip-addresses). On heroku you need to gets their ip [link](https://help.heroku.com/JS13Y78I/i-need-to-whitelist-heroku-dynos-what-are-ip-address-ranges-in-use-at-heroku). 
* You can secure your FTP connection with TLS ([wiki entry](https://wiki.archlinux.org/index.php/Very_Secure_FTP_Daemon#Using_SSL/TLS_to_secure_FTP)). 
* You can serve your assets through HTTPS: [nginx blog entry](https://www.nginx.com/blog/using-free-ssltls-certificates-from-lets-encrypt-with-nginx/).


## Licences

vstfpd is GPL, nginx is bsd2 and active-storage-ftp is not defined. Each sample configuration fall with their respective licences.


