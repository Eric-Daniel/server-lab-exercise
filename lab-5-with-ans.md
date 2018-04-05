# Lab 5 Q & A
### 1. Check if apache2 is running and which port it is listening to.
```bash
systemctl status apache2
sudo netstat -tulpn | grep apache
```
### 2. Enable the SSL module
```
sudo a2enmod ssl
sudo systemctl restart apache2
```

### 3. Create the directory /etc/apache2/ssl-certs
```
sudo mkdir /etc/apache2/ssl-certs
```
### 4. Generate self-signed certificate and key, both must be stored in ssl-certs directory.
```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apache2/ssl-certs/apache-selfsigned.key -out /etc/apache2/ssl-certs/apache-selfsigned.crt
```

### 5. Copy the default configuration file for virtual host with SSL and name it as **scm-ssl.conf**
```
cd /etc/apache2/sites-available/
sudo cp default-ssl.conf scm-ssl.conf
```

### 6. Create a virtual host based on the following information and modify the configuration file accordingly.
- ServerName: www.scm.org
- ServerAdmin: webmaster@scm.org
- DocumentRoot: `/var/www/scm`
```
sudo mkdir -p /var/www/scm
sudo chown -R $USER:$USER /var/www/scm
cd /etc/apache2/sites-available/
```
Then, use an editor (e.g. `nano` or `vim`) to edit `scm-ssl.conf`, change the following line:
```
ServerName   www.scm.org:443
ServerAdmin  webmaster@scm.org
DocumentRoot /var/www/scm
ErrorLog     ${APACHE_LOG_DIR}/scm_error.log
CustomLog    ${APACHE_LOG_DIR}/scm_access.log combined
SSLCertificateFile      /etc/apache2/ssl-certs/apache-selfsigned.crt
SSLCertificateKeyFile   /etc/apache2/ssl-certs/apache-selfsigned.key
```

### 7. Use nano to create index.html in `/var/www/scm`. Sample content of index.html:
```html
<html>
    <head>
        <title> www.scm.org </title>
    </head>
</html>
```

```
sudo nano /var/www/scm/index.html
```

### 8. Enable the site for `scm.org`.
```
sudo a2ensite scm-ssl.conf
```

### 9. Reload apache service.
```
sudo service apache2 reload
```

### 10. Test if the `scm-ssl` server works.
```
curl https://localhost:443 --insecure
```
If the server work, after you run the command above, the shell will print the content of the `index.html` you created previously.  

