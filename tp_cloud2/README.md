## TP2 : Cloud Features & Shellscripts

### I. Un p'tit nom DNS

#### 🌞 Prouvez que c'est effectif

```powershell
PS C:\WINDOWS\system32> az vm show --resource-group TPCesaaar --name azure1.tp1 --show-details --query "{VM_Name:name, Public_IP:publicIps, FQDN:fqdns}"
{
  "FQDN": "meowzer.francecentral.cloudapp.azure.com",
  "Public_IP": "4.211.174.93",
  "VM_Name": "azure1.tp1"
}
```

-----

### II. cloud-init

#### 2\. Gooooo

➜ Sur votre PC, créez un fichier `cloud-init.txt` avec le contenu suivant :

```yaml
#cloud-config
disable_root: false
system_info:
  default_user:
    name: cesaaar

users:
  - name: cesaaar
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: sudo
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBeKrVLDHMvdZXR3g6VH0JnHzciPWr2kDEhm+rEO89Vu julien@DESKTOP-S5QNQ9S

  - name: autre
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: sudo
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBeKrVLDHMvdZXR3g6VH0JnHzciPWr2kDEhm+rEO89Vu julien@DESKTOP-S5QNQ9S
```

##### 🌞 Tester cloud-init

```powershell
PS C:\WINDOWS\system32> az vm create --resource-group TPCesaaar --name azure3.tp2 --image Ubuntu2204 --custom-data C:\Users\Julien\Downloads\cloud-init.txt --admin-username cesaaar --ssh-key-values "$env:USERPROFILE\.ssh\cloud_tp.pub" --location francecentral --size Standard_B1s
```

##### 🌞 Vérifier que cloud-init a bien fonctionné

```powershell
PS C:\WINDOWS\system32> ssh autre@20.19.161.80
```

```text
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1041-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Thu Oct 30 19:26:05 UTC 2025

  System load:  0.6               Processes:             107
  Usage of /:   5.5% of 28.89GB   Users logged in:       0
  Memory usage: 29%               IPv4 address for eth0: 10.0.0.7
  Swap usage:   0%
```

#### 3\. Write your own

##### 🌞 Utilisez cloud-init pour préconfigurer une VM comme `azure2.tp2`

```yaml
#cloud-config
disable_root: false
system_info:
  default_user:
    name: cesaaar

users:
  - name: cesaaar
    groups: sudo
    sudo: ALL=(ALL) ALL
    passwd: $6$pxwk559ug.X9Ub8E$eu9apVI6.psZE6dRisx3SnOuTmE.8EMQaobYAwmns/7RR8IwPw4.4vnvO.PtUs3kQQTNJKUs93fUabq0B1srq.
    lock_passwd: false
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBeKrVLDHMvdZXR3g6VH0JnHzciPWr2kDEhm+rEO89Vu julien@DESKTOP-S5QNQ9S

package-update: true
package_upgrade: true

packages:
  - mysql-server

write_files:
  - path: /root/init.sql
    content: |
      CREATE DATABASE meow_database;
      CREATE USER 'meow'@'%' IDENTIFIED BY 'meow';
      GRANT ALL ON meow_database.* TO 'meow'@'%';
      FLUSH PRIVILEGES;

runcmd:
- mysql < /root/init.sql
```

##### 🌞 Testez que ça fonctionne

```powershell
PS C:\WINDOWS\system32> az vm create --resource-group TPCesaaar --name azure4.tp2 --image Ubuntu2204 --custom-data C:\Users\Julien\Downloads\cloud-init.txt --admin-username cesaaar --ssh-key-values "$env:USERPROFILE\.ssh\cloud_tp.pub" --location francecentral --size Standard_B1s
```

```json
{
  "fqdns": "",
  "id": "/subscriptions/100edd88-eb0b-4a5a-8435-72f5e7020fb4/resourceGroups/TPCesaaar/providers/Microsoft.Compute/virtualMachines/azure4.tp2",
  "location": "francecentral",
  "macAddress": "00-0D-3A-E7-9C-02",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.7",
  "publicIpAddress": "20.19.173.75",
  "resourceGroup": "TPCesaaar"
}
```

```powershell
PS C:\WINDOWS\system32> ssh cesaaar@20.19.173.75
```

```bash
cesaaar@azure4:~$ sudo mysql -u meow -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 9
Server version: 8.0.43-0ubuntu0.22.04.2 (Ubuntu)

Copyright (c) 2000, 2025, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| meow_database      |
| performance_schema |
+--------------------+
3 rows in set (0.01 sec)
```

-----

### III. Gestion de secrets

#### 1\. Un premier secret

##### 🌞 Récupérer votre secret depuis la VM

```bash
cesaaar@azure1:~$ curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

cesaaar@azure1:~$ az login --identity --allow-no-subscriptions
[
  {
    "environmentName": "AzureCloud",
    "id": "413600cf-bd4e-4c7c-8a61-69e73cddf731",
    "isDefault": true,
    "name": "N/A(tenant level account)",
    "state": "Enabled",
    "tenantId": "413600cf-bd4e-4c7c-8a61-69e73cddf731",
    "user": {
      "assignedIdentityInfo": "MSI",
      "name": "systemAssignedIdentity",
      "type": "servicePrincipal"
    }
  }
]
cesaaar@azure1:~$ az keyvault secret show --vault-name keyone --name S1
```

```json
{
  "attributes": {
    "created": "2025-10-31T10:16:39+00:00",
    "enabled": true,
    "expires": null,
    "notBefore": null,
    "recoverableDays": 90,
    "recoveryLevel": "Recoverable+Purgeable",
    "updated": "2025-10-31T10:16:39+00:00"
  },
  "contentType": null,
  "id": "https://keyone.vault.azure.net/secrets/S1/29c788c587c74ab89c186f423730f35f",
  "kid": null,
  "managed": null,
  "name": "S1",
  "tags": {
    "file-encoding": "utf-8"
  },
  "value": "coucou"
}
```

#### 2\. Gérer les secrets de l'application

##### A. Script pour récupérer les secrets

###### 🌞 Coder un ptit script bash : `get_secrets.sh`

```powershell
PS C:\WINDOWS\system32> az keyvault secret set --name DBPASSWORD --vault-name keyone --value meow
```

```bash
#!/bin/bash

az login --identity --allow-no-subscriptions

KeyVaultName="keyone"
SecretName="DBPASSWORD"

SECRET_VALUE=$(az keyvault secret show --vault-name "$KeyVaultName" --name "$SecretName" --query value -o tsv)

sed -i "s/^DB_PASSWORD=.*/DB_PASSWORD=$SECRET_VALUE/" /opt/meow/.env

echo " script finis"
```

###### 🌞 Environnement du script `get_secrets.sh`

```bash
cesaaar@azure1:/$ cd /usr/local/bin
cesaaar@azure1:/usr/local/bin$ sudo nano get_secrets.sh
cesaaar@azure1:/usr/local/bin$ sudo chown webapp:webapp /usr/local/bin/get_secrets.sh
cesaaar@azure1:/usr/local/bin$ sudo chmod 700 /usr/local/bin/get_secrets.sh
```

##### B. Exécution automatique

###### 🌞 Ajouter le script en `ExecStartPre=` dans `webapp.service`

```bash
cesaaar@azure1:/usr/local/bin$ sudo systemctl edit webapp.service
```

```ini
### Anything between here and the comment below will become the new contents of the file
[Service]
ExecStartPre=/usr/local/bin/get_secrets.sh


### Lines below this comment will be discarded

### /etc/systemd/system/webapp.service
# [Unit]
# Description=Super Webapp MEOW
#
# [Service]
# User=webapp
# WorkingDirectory=/opt/meow
# ExecStart=/opt/meow/bin/python app.py
#
# [Install]
# WantedBy=multi-user.target
```

```bash
cesaaar@azure1:/usr/local/bin$ sudo systemctl daemon-reload
cesaaar@azure1:/usr/local/bin$ sudo systemctl restart webapp.service
```

###### 🌞 Prouvez que la ligne en `ExecStartPre=` a bien été exécutée

```bash
cesaaar@azure1:/usr/local/bin$ systemctl status webapp.service
```

```text
× webapp.service - Super Webapp MEOW
     Loaded: loaded (/etc/systemd/system/webapp.service; disabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/webapp.service.d
             └─override.conf
     Active: failed (Result: exit-code) since Sun 2025-11-02 18:01:44 UTC; 4s ago
    Process: 27871 ExecStartPre=/usr/local/bin/get_secrets.sh (code=exited, status=0/SUCCESS)
    Process: 27881 ExecStart=/opt/meow/bin/python app.py (code=exited, status=1/FAILURE)
   Main PID: 27881 (code=exited, status=1/FAILURE)
        CPU: 1.121s
```

## mon get_secret.sh return une char vide (jle voit dans le .env le mdp est modif en vide), j'ai plus le temps jdois faire du C++ déso
