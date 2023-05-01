# Authentification Kerberos
Ce projet consiste en la création et la configuration d'un royaume Kerberos qui permettra aux acteurs autorisés du systèmes de s'authentifier à l'aide d'un ticket.

# Qu'est-ce que Kerberos et comment ça marche ?
L'activation de l'authentification GSSAPI / Kerberos permet l'authentification unique (sans utiliser le nom d'utilisateur et le mot de passe standard) pour les clients par l'intermédiaire d'un "ticket".
Kerberos est un protocole d'authentification réseau utilisé pour vérifier l'identité de deux hôtes (ou plus) approuvés sur un réseau non approuvé. Il utilise la cryptographie à clé secrète et un tiers de confiance (Kerberos Key Distribution Center) pour authentifier les applications client-serveur. Le centre de distribution de clés (KDC) donne aux clients des tickets représentant leurs informations d'identification réseau. Le ticket Kerberos est présenté aux serveurs une fois la connexion établie.

![fig](https://user-images.githubusercontent.com/62994130/235366271-042582dc-ecdf-425e-ba55-3d8c23feeada.jpg)

# Préparation de l'environnement : 
## Manipulation sur les 3 machines (KDC, Client, Service) :

Nous changeons le nom d'hôte de chacune des machines afin de les identifier:

KDC:
```
hostnamectl --static set-hostname kdc.centrale.tn
```
Client:
```
hostnamectl --static set-hostname client.centrale.tn
```
Service:
```
hostnamectl --static set-hostname service.centrale.tn
```

Nous récupérons l’adresse IP des 2 machines à l’aide de la commande suivante :
```
ip r l
```
![fig](https://user-images.githubusercontent.com/62994130/235366484-0bed8f00-0b91-4c3f-b2da-03ad6ba5c5a9.png)
![fig](https://user-images.githubusercontent.com/62994130/235366485-48290fc4-6825-439a-9608-1b90d8f73b8c.png)
![fig](https://user-images.githubusercontent.com/62994130/235366488-c9848195-0dfd-4f9a-884d-9795ffc87a54.png)



| Acteur        | Hostname             | Adresse IP      |
| ------------- | -------------------- | --------------- |
| KDC           | kdc.centrale.tn      | 192.168.150.133 |
| Client        | client.centrale.tn   | 192.168.150.134 |
| Serveur       | service.centrale.tn  | 192.168.150.135 |

Nous vérifions la synchronisation des horloges des 3 machines :
```
Timedatectl
```
![fig](https://user-images.githubusercontent.com/62994130/235367098-9be62a47-5480-4ea1-99d0-ed9a7ef9c479.png)
![fig](https://user-images.githubusercontent.com/62994130/235367102-1820d0e8-6b42-4342-aabc-1b0bd6345bae.png)
![fig](https://user-images.githubusercontent.com/62994130/235367104-59c4486e-728c-43ad-a954-dc18931f8836.png)

Nous souhaitons que les 3 machines appartiennent au même DNS. Pour se faire nous modifions le fichier hosts et ajoutons 3 lignes pour définir le KDC, le client et le serveur.
```
Sudo nano /etc/hosts
```

On rajoute les lignes :
```
192.168.150.133 kdc.centrale.tn kdc
192.168.150.134 client.centrale.tn client
192.168.150.135 service.centrale.tn service
```
![Ubuntu_22 04(4)-2023-04-25-11-02-53](https://user-images.githubusercontent.com/62994130/235367339-3ce0f173-563a-4d76-9ddb-837fb2c7e425.png)


Nous copions les 3 lignes et on les ajoute au même fichier de chaque machine.

A présent, nous vérifions la bonne communication entre les machines à l’aide de : 
Leur adresse IP : 
```
Ping 192.168.150.133
```
```
Ping 192.168.150.134
```
```
Ping 192.168.150.135
```
Leur noms :
```
Ping kdc
```
```
Ping client
```
```
Ping service
```


## Configuration de la machine KDC : 


```
sudo apt-get update
```

Installation de kerberos :


```
sudo apt install krb5-kdc krb5-admin-server krb5-config
```

![fig](https://user-images.githubusercontent.com/62994130/235367434-d2347d6c-82f0-4b29-8136-728c265f4fa0.png)
![fig](https://user-images.githubusercontent.com/62994130/235367443-a7c328b0-2aa2-4915-8e85-6bbd9073ec50.png)
![fig](https://user-images.githubusercontent.com/62994130/235367448-7ecc28cc-6f27-4b73-b1f9-1b891a284c45.png)
![fig](https://user-images.githubusercontent.com/62994130/235367454-4506709f-a185-4404-bc9e-639b41612d6f.png)

Création du royaume: 

```
krb5 newrealm
```

![fig](https://user-images.githubusercontent.com/62994130/235367493-1b59966b-c419-47ad-a8fe-1735eef667dc.png)

Modification du fichier kadm5.acl : 

```
sudo nano /etc/krb5kdc/kadm5.acl
```

![fig](https://user-images.githubusercontent.com/62994130/235367600-55654792-5226-4391-ba03-a37d62c41660.png)

On décommente la ligne *\admin * et on ajoute notre nom de domaine @centrale.tn:


![fig](https://user-images.githubusercontent.com/62994130/235367605-5e89843b-87c1-4829-a680-ef0afd94c391.png)

Ajout des principals : 
```
kadmin.local
```
```
addprinc root/admin
```
```
addprinc host/kdc.centrale.tn
```
```
addprinc utilisateur
```
```
addprinc service/service.centrale.tn
```
```
add_entry -password -p root/admin@CENTRALE.TN -k 1 -e aes256-cts-hmac-sha1-96
```
![fig](https://user-images.githubusercontent.com/62994130/235475743-32779ad5-9df1-4648-b4d2-a2a17e113a48.png)
![fig](https://user-images.githubusercontent.com/62994130/235476304-55dd9085-9f66-4510-99dd-d5e4a5c4d67f.png)
![fig](https://user-images.githubusercontent.com/62994130/235476541-c0753b9b-6795-4c85-b496-633409afaaac.PNG)
![fig](https://user-images.githubusercontent.com/62994130/235476548-f3e28cac-4152-4a3d-ba5a-c4c441613d21.PNG)
![fig](https://user-images.githubusercontent.com/62994130/235476550-8e5847d3-2703-44db-9d2b-14c3d0d44b64.PNG)
![fig](https://user-images.githubusercontent.com/62994130/235476554-b73a563b-e9b4-4b2e-b8b0-e99322a6f418.PNG)
![fig](https://user-images.githubusercontent.com/62994130/235476557-aa9ba523-44b5-4b71-a3f3-90a6f36d9311.PNG)
```
ktadd -k -randkey host/kdc.centrale.tn
```
![fig](https://user-images.githubusercontent.com/62994130/235476560-864a18e8-230b-49f1-b443-23b00da18e29.PNG)
![fig](https://user-images.githubusercontent.com/62994130/235476570-43e1e93b-4c49-4977-857a-73ccb6ac594f.PNG)
![fig](https://user-images.githubusercontent.com/62994130/235477556-b30993ae-6173-4bcb-8802-91bb9b99c051.png)



Nous redémarrons les services : 

```
sudo systemctl restart krb5-kdc
```
```
sudo systemctl restart krb5-admin-server
```
![fig](https://user-images.githubusercontent.com/62994130/235475268-04c08b6e-ff95-4a56-806f-6db279151a54.png)

Ajout au fichier keytab "kadm5.keytab": 
![fig](https://user-images.githubusercontent.com/62994130/235465483-d97de4b8-eef6-401c-9287-ccedcb452674.png)
![fig](https://user-images.githubusercontent.com/62994130/235465475-ef8f24c8-de84-40cc-ab8a-c00341f18588.png)

Création du fichier keytab du service : 
![fig](https://user-images.githubusercontent.com/62994130/235472606-7bafde1d-86cc-4b1d-ad34-f721757a7d37.png)

Envoi du fichier service.keytab du KDC au service :
```
scp "service.keytab" service@service.centrale.tn:
```
![fig](https://user-images.githubusercontent.com/62994130/235472967-c6fb0b1d-ba0e-41a5-89a3-6177774c1d6d.png)
```
ls
file service.keytab
```
![fig](https://user-images.githubusercontent.com/62994130/235472850-0f37f241-61dd-4ccd-9a08-c4ccde3661a9.png)
```
ktutil
rkt service.keytab
l
```
![fig](https://user-images.githubusercontent.com/62994130/235473236-afe7f7cb-d923-4b85-a4af-5aa9ca8fd527.png)



## Configuration de la machine Client : 
```
sudo apt-get update
```

Installation de kerberos :


```
sudo apt install krb5-user
```
![fig](https://user-images.githubusercontent.com/62994130/235461731-bcfacdd3-78ef-4cdc-afb9-ef76a1a9532c.png)
![fig](https://user-images.githubusercontent.com/62994130/235461800-0d0e036c-2941-42c8-acc1-2bd17cd848de.png)
![fig](https://user-images.githubusercontent.com/62994130/235461794-f2872a0c-6b54-4a3a-a516-e9031aff8147.png)
![fig](https://user-images.githubusercontent.com/62994130/235461788-f63c9606-8cf0-4dc8-8cfc-55bddca1b2f9.png)




## Configuration de la machine Service : 
```
sudo apt-get update
```

Installation de kerberos :


```
sudo apt install krb5-user
```
![fig5](https://user-images.githubusercontent.com/62994130/235462033-0b29e371-ca54-452e-a656-8b5c5166c70e.png)
![fig](https://user-images.githubusercontent.com/62994130/235462031-dcb0b1a4-0ebf-4351-a149-d43718a7cf55.png)
![fig](https://user-images.githubusercontent.com/62994130/235462019-c7fb521a-e3e0-4016-9fc6-44ba583df1f9.png)
![fig](https://user-images.githubusercontent.com/62994130/235462023-085c8e27-9ca7-42f8-8000-f6c91a356131.png)


## Installation d'OpenSSH sur les 3 machines : 
Exemple sur la machine client:
```
sudo apt-get update
```
```
sudo apt install openssh-server
```
![fig](https://user-images.githubusercontent.com/62994130/235466133-a20a2b4c-5a9e-4434-b20e-2525b8321dd5.png)

```
sudo nano /etc/ssh/sshd_config
```
Nous modifions le fichier en décommentant les 2 lignes :
```
GSSAPIAuthentication yes
GSSAPIDelegateCredentials yes
```
```
sudo nano /etc/ssh/ssh_config
```
Nous modifions le fichier en décommentant les 2 lignes :
```
GSSAPIAuthentication yes
GSSAPICleanupCredentials yes
```
![fig](https://user-images.githubusercontent.com/62994130/235466336-2cdda0d5-1cd8-42df-a0a7-b7511810ebaf.png)
![fig](https://user-images.githubusercontent.com/62994130/235466333-a989c4ec-8916-4e02-8e61-b36ce52e0e88.png)
![fig](https://user-images.githubusercontent.com/62994130/235466338-9a2f9de0-04fa-4b9e-85b3-a9a0e227710c.png)

# Authentification SSH avec Kerberos : 
```
klist
```
```
kinit
```
```
ssh kdc.centrale.tn
```
![fig](https://user-images.githubusercontent.com/62994130/235470803-e21d05a4-052a-4168-b1aa-63d5b758a08a.png)
![fig](https://user-images.githubusercontent.com/62994130/235470795-5f3b03c8-aab4-4d8e-b94b-7307e34f8654.png)
![fig](https://user-images.githubusercontent.com/62994130/235470790-6b69fba6-e512-44cb-b788-0b5f9b5ddff0.png)




