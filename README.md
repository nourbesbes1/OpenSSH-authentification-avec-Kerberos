# Authentification Kerberos
Ce projet consiste en la création et la configuration d'un royaume Kerberos qui permettra aux acteurs autorisés du systèmes de s'authentifier à l'aide d'un ticket.

# Qu'est-ce que Kerberos et comment ça marche ?
L'activation de l'authentification GSSAPI / Kerberos permet l'authentification unique (sans utiliser le nom d'utilisateur et le mot de passe standard) pour les clients par l'intermédiaire d'un "ticket".
Kerberos est un protocole d'authentification réseau utilisé pour vérifier l'identité de deux hôtes (ou plus) approuvés sur un réseau non approuvé. Il utilise la cryptographie à clé secrète et un tiers de confiance (Kerberos Key Distribution Center) pour authentifier les applications client-serveur. Le centre de distribution de clés (KDC) donne aux clients des tickets représentant leurs informations d'identification réseau. Le ticket Kerberos est présenté aux serveurs une fois la connexion établie.

![fig](https://user-images.githubusercontent.com/62994130/235366271-042582dc-ecdf-425e-ba55-3d8c23feeada.jpg)

## Manipulation sur les 3 machines (KDC, Client, Service) :

Nous changeons le nom d'hôte de chacune des machines :

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

Nous vérifions la synchronisation de l’horloge des 2 machines :
```
Timedatectl
```
![fig](https://user-images.githubusercontent.com/62994130/235367098-9be62a47-5480-4ea1-99d0-ed9a7ef9c479.png)
![fig](https://user-images.githubusercontent.com/62994130/235367102-1820d0e8-6b42-4342-aabc-1b0bd6345bae.png)
![fig](https://user-images.githubusercontent.com/62994130/235367104-59c4486e-728c-43ad-a954-dc18931f8836.png)


Nous vérifions la bonne communication entre les machines à l’aide de : 
```
Ping IPMachine1
Ping IPMachine2
```



Sur la machine1 (KDC) : 

Nous voulons que les 2 machines appartiennent au même DNS. Pour se faire nous modifions le fichier hosts et ajoutons 2 lignes pour définir le client et le serveur.
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


On copie les 3 lignes et on les ajoute au même fichier de chaque machine

## Configuration KDC

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

