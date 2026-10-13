# Apache SSL Web Hosting Troubleshooting

## 1. Apache Service Is Not Running

Check the service:

```bash
systemctl status httpd
```

Validate the configuration:

```bash
httpd -t
```

Review logs:

```bash
journalctl -u httpd -xe
```

**Possible causes:**
- Incorrect Apache configuration.
- Port 80 or 443 is already in use.
- Incorrect certificate paths.
- File permission or SELinux issues.

## 2. Website Is Not Accessible

Check whether Apache is listening:

```bash
ss -lntp | grep -E ':(80|443)\b'
```

Check firewall rules:

```bash
firewall-cmd --list-all
```

Allow HTTP and HTTPS if required:

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --reload
```

Confirm that the client can reach the server IP address.

## 3. Local Domain Is Not Resolving

Check the client's hosts file.

Expected entry:

```text
192.168.0.103 golden-plate.local www.golden-plate.local
```

On Linux, inspect it using:

```bash
cat /etc/hosts
```

On Windows, inspect:

```text
C:\Windows\System32\drivers\etc\hosts
```

## 4. SSL Certificate Warning

A self-signed certificate is not trusted by default.

Inspect the certificate:

```bash
openssl x509 -in /root/server.crt -noout -subject -issuer -dates
```

Test the HTTPS connection:

```bash
openssl s_client -connect golden-plate.local:443 \
  -servername golden-plate.local
```

Check that the certificate matches the hostname and includes a Subject Alternative Name (SAN).

A self-signed certificate can encrypt traffic, but browsers will still warn until the certificate is trusted.

## 5. SELinux Issues

Check the current mode:

```bash
getenforce
```

Review recent SELinux denials:

```bash
ausearch -m AVC -ts recent
```

Inspect website directory labels:

```bash
ls -Zd /opt/Restaurant-Website
```

If SELinux blocks access, investigate the denial and apply suitable file contexts and permissions. Do not leave SELinux disabled as the permanent solution.

## 6. Certificate or Key File Errors

Check that the files exist:

```bash
ls -l /root/server.crt /root/server.key
```

Inspect the certificate:

```bash
openssl x509 -in /root/server.crt -noout -text
```

Keep the private key restricted to authorized users and do not upload it to GitHub.

## Troubleshooting Checklist

- [ ] Apache service is running.
- [ ] Apache configuration test passes.
- [ ] Ports 80 and 443 are listening as expected.
- [ ] Firewall allows required traffic.
- [ ] Local domain resolves to the correct IP.
- [ ] Certificate and private key paths are correct.
- [ ] Website directory permissions and SELinux contexts are appropriate.
- [ ] HTTPS connection has been tested.
