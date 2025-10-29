
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

