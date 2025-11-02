
# 📄 Instructions du TP - Gestion de VMs et Clés SSH

---

# I. Prérequis

## 2. Une paire de clés SSH

### A. Choix de l'algorithme de chiffrement 🔑
**🌞 Déterminer quel algorithme de chiffrement utiliser pour vos clés**

* **Pourquoi éviter RSA pour SSH ?**
    > RSA est de moins en moins recommandé à mesure que la technologie progresse et que de plus grandes clés (supérieures à 2048 bits) sont nécessaires, incitant à considérer des algorithmes alternatifs.
    * *Source d'information :* [Comparing SSH Key Algorithms (strongdm.com)](https://www.strongdm.com/blog/comparing-ssh-keys#:~:text=Types%20of%20SSH%20Key%20Algorithms,-RSA&text=However%2C%20as%20technology%20advances%2C%20larger,requiring%20consideration%20of%20alternative%20algorithms.)

* **Algorithme de chiffrement recommandé : ED25519**
    > **ED25519** est l'algorithme recommandé pour les nouvelles clés SSH.
    * *Source de recommandation :* [SSH Keys and Best Practices (docs.cis.strath.ac.uk)](https://docs.cis.strath.ac.uk/ssh-keys/#:~:text=ssh%5Cid_ed25519%20on%20Windows).,should%20consider%20upgrading%20where%20possible.)

---

### B. Génération de votre paire de clés 🛠️
**🌞 Générer une paire de clés pour ce TP**

Utilisez la commande `ssh-keygen` pour générer une clé de type **ED25519** :

```bash
ssh-keygen -t ed25519 -f C:\Users\Julien\.ssh\cloud_tp
````

**Vérification des fichiers générés dans `C:\Users\Julien\.ssh` :**

| Mode | Date | Heure | Taille | Nom |
| :--- | :--- | :--- | :--- | :--- |
| `-a----` | 29/10/2025 | 18:04 | 464 | **cloud\_tp** (Clé privée) |
| `-a----` | 29/10/2025 | 18:04 | 105 | **cloud\_tp.pub** (Clé publique) |
| `-a----` | 07/01/2025 | 16:22 | 5614 | known\_hosts |
| `-a----` | 07/01/2025 | 16:22 | 4874 | known\_hosts.old |

-----

### C. Agent SSH

**🌞 Configurer un agent SSH sur votre poste**

1.  **Configurer le service `ssh-agent` en automatique et le démarrer :**
    ```powershell
    Get-Service ssh-agent | Set-Service -StartupType Automatic
    Start-Service ssh-agent
    ```
2.  **Ajouter la clé privée à l'agent :**
    ```powershell
    ssh-add $env:USERPROFILE\.ssh\cloud_tp
    ```

-----

-----

# II. Spawn des VMs (Virtual Machines)

## 1\. Depuis la WebUI (non vu que ça marche pas)

**🌞 Connectez-vous en SSH à la VM pour preuve**

Connexion réussie à l'IP `4.212.92.135` :

```powershell
PS C:\Users\Julien\.ssh> ssh cesaaar@4.212.92.135
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1041-azure x86_64)

 System information as of Wed Oct 29 18:11:05 UTC 2025

  System load:  0.08              Processes:             105
  Usage of /:   5.5% of 28.89GB   Users logged in:       0
  Memory usage: 30%               IPv4 address for eth0: 10.0.0.4
  Swap usage:   0%

Expanded Security Maintenance for Applications is not enabled.
0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See [https://ubuntu.com/esm](https://ubuntu.com/esm) or run: sudo pro status

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.
Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.
cesaaar@VM1:~$
```

-----

## 2\. az : a programmatic approach (Azure CLI)

**🌞 Créez une VM depuis le Azure CLI : `azure1.tp1`**

1.  Création du groupe de ressources :
    ```powershell
    PS C:\Users\Julien\.ssh> az group create --name TPCesaaar --location francecentral
    ```
2.  Création de la VM (`azure1.tp1`) :
    ```powershell
    PS C:\Users\Julien\.ssh> az vm create --resource-group TPCesaaar --name azure1.tp1 --image Ubuntu2204 --admin-username cesaaar --ssh-key-values "$env:USERPROFILE\.ssh\cloud_tp.pub" --location francecentral --size Standard_B1s
    ```

**🌞 Assurez-vous que vous pouvez vous connecter à la VM en SSH sur son IP publique**

Connexion réussie à l'IP `4.211.174.93` :

```powershell
PS C:\Users\Julien\.ssh> ssh cesaaar@4.211.174.93
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1041-azure x86_64)

 System information as of Wed Oct 29 18:25:30 UTC 2025

  System load: 0.0               Processes:             105
  Usage of /:   5.5% of 28.89GB   Users logged in:       0
  Memory usage: 30%               IPv4 address for eth0: 10.0.0.5
  Swap usage:   0%

Expanded Security Maintenance for Applications is not enabled.
0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See [https://ubuntu.com/esm](https://ubuntu.com/esm) or run: sudo pro status

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.
Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.
cesaaar@azure1:~$
```

**🌞 Une fois connecté, prouvez la présence...**

  * **...du service `walinuxagent.service` (Agent Azure Linux)**

    ```bash
    cesaaar@azure1:~$ systemctl status walinuxagent.service
    ● walinuxagent.service - Azure Linux Agent
         Loaded: loaded (/lib/systemd/system/walinuxagent.service; enabled; vendor preset: enabled)
         Active: active (running) since Wed 2025-10-29 18:17:33 UTC; 1h 14min ago
    ```

  * **...du service `cloud-init.service`**

    ```bash
    cesaaar@azure1:~$ systemctl status cloud-init.service
    ● cloud-init.service - Cloud-init: Network Stage
         Loaded: loaded (/lib/systemd/system/cloud-init.service; enabled; vendor preset: enabled)
         Active: active (exited) since Wed 2025-10-29 18:17:32 UTC; 1h 15min ago
       Main PID: 519 (code=exited, status=0/SUCCESS)
            CPU: 3.454s
    ```

-----

## 3\. Spawn moar moar moaaar VMs

### A. Another VM another friend :d 👯

**🌞 Créez une deuxième VM : `azure2.tp1`**

Note : La VM est créée sans adresse IP publique (`--public-ip-address '""'`).

```bash
az vm create --resource-group TPCesaaar --name azure2.tp1 --image Ubuntu2204 --admin-username cesaaar --ssh-key-values "$env:USERPROFILE\.ssh\cloud_tp.pub" --location francecentral --size Standard_B1s --public-ip-address '""'
```

**🌞 Affichez des infos au sujet de vos deux VMs**

| VM Name | Resource Group | Location | Power State | MAC Address | Private IP | Public IP |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `azure2.tp1` | TPCesaaar | francecentral | VM running | 7C-ED-8D-6D-42-12 | 10.0.0.6 | **""** |
| `azure1.tp1` | TPCesaaar | francecentral | VM running | 00-22-48-38-81-4F | 10.0.0.5 | **4.211.174.93** |

-----

### B. Config SSH client 💻

**🌞 Configuration SSH client pour les deux machines**

  * **Connexion via alias `az1` (`azure1.tp1`) :**

    ```powershell
    PS C:\Users\Julien\.ssh> ssh az1
    Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1041-azure x86_64)

     System information as of Wed Oct 29 21:01:59 UTC 2025

      System load:  0.0               Processes:             105
      Usage of /:   5.7% of 28.89GB   Users logged in:       0
      Memory usage: 31%               IPv4 address for eth0: 10.0.0.5
      Swap usage:   0%
    ```

  * **Connexion via alias `az2` (`azure2.tp1`) :**

    ```powershell
    PS C:\Users\Julien\.ssh> ssh az2
    Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1041-azure x86_64)

     System information as of Wed Oct 29 21:02:10 UTC 2025

      System load:  0.0               Processes:             104
      Usage of /:   5.7% of 28.89GB   Users logged in:       0
      Memory usage: 32%               IPv4 address for eth0: 10.0.0.6
      Swap usage:   0%
    ```


-----

## III. Déployer et configurer un machin

### 1\. Machine `azure2.tp1`

#### 🌞 Installer MySQL/MariaDB sur `azure2.tp1`

```bash
sudo apt install mysql-server
```

#### 🌞 Démarrer le service MySQL/MariaDB sur `azure2.tp1`

```bash
cesaaar@azure2:~$ systemctl status mysql.service
● mysql.service - MySQL Community Server
     Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2025-10-30 07:50:45 UTC; [cite_start]37s ago [cite: 1, 2]
```

#### 🌞 Ajouter un utilisateur dans la base de données pour que mon app puisse s'y connecter

```bash
cesaaar@azure2:~$ sudo mysql
```

```sql
mysql> CREATE DATABASE cesaarbase;
[cite_start]Query OK, 1 row affected (0.04 sec) [cite: 3]

mysql> CREATE USER 'meow'@'%' IDENTIFIED BY 'meow';
[cite_start]Query OK, 0 rows affected (0.04 sec) [cite: 4]

mysql> GRANT ALL ON cesaarbase.* TO 'meow'@'%';
[cite_start]Query OK, 0 rows affected (0.03 sec) [cite: 5]

mysql> FLUSH PRIVILEGES;
```

#### 🌞 Ouvrez un port firewall si nécessaire

Dans la conf MySQL:

```ini
bind-address            = 0.0.0.0
```

```bash
cesaaar@azure1:~$ mysql -u meow -p -h 10.0.0.6 cesaarbase
```

-----

### 2\. Machine `azure1.tp1`

#### A. Récupération de l'application sur la machine

##### 🌞 Récupération de l'application sur la machine

```bash
cesaaar@azure1:/opt/meow$ sudo git clone https://gitlab.com/it4lik/b2-pano-cloud-2025.git
cesaaar@azure1:/opt/meow$ sudo cp -r b2-pano-cloud-2025/docs/tp/1/app/* /opt/meow
cesaaar@azure1:/opt/meow$ sudo cp b2-pano-cloud-2025/docs/tp/1/app/.env /opt/meow
```

#### B. Installation des dépendances de l'application

##### 🌞 Installation des dépendances de l'application

```bash
cesaaar@azure1:/opt/meow$ sudo chown -R cesaaar:cesaaar /opt/meow

cesaaar@azure1:/opt/meow$ sudo apt install python3.10-venv

cesaaar@azure1:/opt/meow$ python3 -m venv .
[cite_start]cesaaar@azure1:/opt/meow$ ./bin/pip install -r requirements.txt [cite: 6]
```

#### C. Configuration de l'application

##### 🌞 Configuration de l'application

```bash
cesaaar@azure1:/opt/meow$ sudo nano .env
```

```ini
# Database Configuration
DB_HOST=10.0.0.6 # changez ça pour l'adresse IP de azure2.tp1
DB_PORT=3306
DB_NAME=cesaarbase
DB_USER=meow
DB_PASSWORD=meow
```

#### D. Gestion de users et de droits

##### 🌞 Gestion de users et de droits

```bash
cesaaar@azure1:/opt/meow$ sudo useradd webapp
cesaaar@azure1:/opt/meow$ sudo chown -R webapp:webapp /opt/meow
cesaaar@azure1:/opt/meow$ sudo chmod -R o-rwx /opt/meow
```

#### E. Création d'un service `webapp.service` pour lancer l'application

##### 🌞 Création d'un service `webapp.service` pour lancer l'application

```bash
cesaaar@azure1:/opt/meow$ sudo nano /etc/systemd/system/webapp.service
cesaaar@azure1:/opt/meow$ sudo systemctl daemon-reload
```

-----

### 3\. Visitez l'application

#### 🌞 L'application devrait être fonctionnelle sans soucis à partir de là

```powershell
PS C:\WINDOWS\system32> az vm open-port --resource-group TPCesaaar --name azure1.tp1 --port 8000 --priority 1012
```

```bash
cesaaar@azure1:/$ sudo -u webapp /opt/meow/bin/python /opt/meow/app.py
 * [cite_start]Serving Flask app 'app' [cite: 7]
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment.
[cite_start]Use a production WSGI server instead. [cite: 8]
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:8000
 * Running on http://10.0.0.5:8000
```

```powershell
C:\Users\Julien>curl http://4.211.174.93:8000
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Purr Messages - Cat Message Board</title>
    <style>
        /* Modern CSS with cat-themed design */
        :root {
            [cite_start]--primary: #ff6b6b; [cite: 9]
            --secondary: #4ecdc4;
            --accent: #ffd166;
            --dark: #1a1a2e;
            --light: #f8f9fa;
            --gradient: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            --cat-paw: #ff9a8b;
        }
}
```
