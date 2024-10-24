To enable TLS on the reverge web application, select the appropriate radio button. There are two supported methods, **Let's Encrypt** for automatic certificate generation or uploading custom generated TLS certificates.
<br>
<br>
## Let's Encrypt
For the **Let's Encrypt** option, enter the fully qualified domain name (FQDN) in the **Domain Name** field and click the <img src="../../assets/save_btn.png" alt="Save button" width="25">  button. <span style="color: red;">**If you do not have a DNS A record configured for the given domain and a firewall exception for port 443, this will fail.**</span>
<br>
<br>
<center>
<img src="../../assets/settings_cert_le.png" alt="Lets Encrypt Cert" width="750" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>
<br>
<br>
## Custom Certificates
For the **Custom Certificates** option, enter the fully qualified domain name (FQDN) in the **Domain Name** field. Next click the **Browse** button for the Private and Public Certificate fields and select the appropriate certificate file. Finally, click the <img src="../../assets/save_btn.png" alt="Save button" width="25">  button.
<br>
<br>
<br>
<center>
<img src="../../assets/settings_cert_custom.png" alt="Custom Certs" width="750" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>
<br>
<br>
## Troubleshooting
If everything worked as expected, the Reverge server should soon be reachable on port **443** at the specified domain—for example, [https://reverge.io](https://reverge.io). If not, you can restart the process by clicking the <img src="../../assets/reset_btn.png" alt="Save button" width="28"> button.
