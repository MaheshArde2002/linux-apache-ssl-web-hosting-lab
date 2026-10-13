# Linux Apache SSL/TLS Web Hosting Lab

## Project Overview

This project demonstrates how to host a restaurant website on a CentOS 8 server using Apache HTTP Server and secure it with HTTPS using a self-signed SSL/TLS certificate.

The project covers Apache installation, Virtual Host configuration, Git-based website deployment, SSL certificate generation using OpenSSL, firewall configuration, local domain resolution, and HTTPS troubleshooting.

## Objectives

- Install and configure Apache HTTP Server on CentOS 8.
- Deploy a website from a GitHub repository.
- Configure Apache name-based Virtual Hosting.
- Generate a self-signed SSL/TLS certificate using OpenSSL.
- Enable HTTPS on port 443.
- Configure firewalld to allow HTTP and HTTPS traffic.
- Test local domain resolution and HTTPS access.
- Troubleshoot Apache, SSL certificate, and SELinux-related issues.

## Technologies Used

- Operating System: CentOS 8
- Web Server: Apache HTTP Server (httpd)
- Security: SSL/TLS
- Certificate Tool: OpenSSL
- Version Control: Git and GitHub
- Firewall: firewalld
- Service Management: systemd
- Web Browser: Firefox

## Lab Architecture

Client Browser  
↓  
Local DNS Resolution (/etc/hosts)  
↓  
CentOS 8 Web Server — 192.168.0.103  
↓  
Apache HTTP Server (httpd)  
├── HTTP — Port 80  
└── HTTPS — Port 443 (SSL/TLS)  
↓  
Website Document Root: /opt/Restaurant-Website

## Implementation Steps

### 1. Install Apache

```bash
yum install httpd -y
```

### 2. Start and Enable Apache

```bash
systemctl start httpd
systemctl enable httpd
systemctl status httpd
```

### 3. Install Git and Clone the Website

```bash
yum install git -y
cd /opt
git clone https://github.com/DaneshCode/Restaurant-Website.git
cd Restaurant-Website
ls
```

### 4. Configure Apache Virtual Hosting

Configure a Virtual Host with the website directory as its DocumentRoot and `golden-plate.local` as its ServerName.

Example configuration:

```apache
<VirtualHost *:80>
    ServerName golden-plate.local
    ServerAlias www.golden-plate.local
    DocumentRoot "/opt/Restaurant-Website"

    ErrorLog /var/log/httpd/golden-plate-error.log
    CustomLog /var/log/httpd/golden-plate-access.log combined
</VirtualHost>

<Directory "/opt/Restaurant-Website">
    Require all granted
</Directory>
```

### 5. Allow HTTP Traffic

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
firewall-cmd --list-all
```

### 6. Generate a Self-Signed Certificate

```bash
openssl req -x509 -newkey rsa:2048 \
  -keyout server.key \
  -out server.crt \
  -days 365 -nodes
```

This creates a private key and a self-signed certificate for lab testing. The certificate is not automatically trusted by browsers.

### 7. Install Apache SSL Support

```bash
yum install mod_ssl -y
```

Configure the HTTPS Virtual Host on port 443 and enable SSL:

```apache
<VirtualHost *:443>
    ServerName golden-plate.local
    ServerAlias www.golden-plate.local
    DocumentRoot "/opt/Restaurant-Website"

    SSLEngine on
    SSLCertificateFile /path/to/server.crt
    SSLCertificateKeyFile /path/to/server.key

    ErrorLog /var/log/httpd/golden-plate-ssl-error.log
    CustomLog /var/log/httpd/golden-plate-ssl-access.log combined
</VirtualHost>
```

Replace the certificate paths with the actual paths used on your server. Keep the private key protected and outside the public website directory.

### 8. Allow HTTPS Traffic

```bash
firewall-cmd --permanent --add-service=https
firewall-cmd --reload
```

### 9. Configure Local Domain Resolution

Add the following entry to the client's `/etc/hosts` file:

```text
192.168.0.103 golden-plate.local www.golden-plate.local
```

On Windows, the equivalent file is:

`C:\Windows\System32\drivers\etc\hosts`

### 10. Validate the Configuration

```bash
httpd -t
systemctl restart httpd
systemctl status httpd
```

Test the endpoints:

- HTTP: `http://192.168.0.103`
- HTTPS: `https://golden-plate.local`

Because the certificate is self-signed, Firefox may display a trust warning. This is expected unless the certificate has been explicitly trusted.

## Troubleshooting

### Apache fails to start

```bash
httpd -t
journalctl -u httpd -xe
ss -lntp
```

Check configuration syntax, port conflicts, certificate paths, and file permissions.

### HTTPS is not accessible

```bash
firewall-cmd --list-services
openssl x509 -in /path/to/server.crt -noout -dates -subject
```

Confirm that HTTPS is allowed through the firewall, the certificate and key paths are correct, and Apache is listening on port 443.

### SELinux access issues

Check the audit logs and file contexts before changing SELinux settings:

```bash
getenforce
ausearch -m AVC -ts recent
ls -Z /opt/Restaurant-Website
```

For a production-style setup, configure appropriate SELinux file contexts and permissions instead of disabling SELinux.

## Screenshots

Implementation screenshots are available in the [`screenshots/`](screenshots/) directory.

They document package installation, service configuration, Git cloning, Virtual Host configuration, firewall rules, certificate generation, HTTPS setup, and browser verification.

## Security Notes

- This project uses a self-signed certificate for a local lab.
- A self-signed certificate does not provide public browser trust by default.
- Never commit private keys, passwords, or production credentials.
- For public websites, use a trusted certificate authority and configure certificate renewal.
- Keep SELinux enforcing and apply the correct security contexts.

## Outcome

Successfully configured an Apache-hosted restaurant website on CentOS 8 and tested HTTP and HTTPS access using a local domain and a self-signed SSL/TLS certificate.

## Disclaimer

This repository is intended for educational and lab practice. The website source belongs to its original author.
