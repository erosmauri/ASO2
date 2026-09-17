---
layout: custom
title: SP1 / Sistemes d'inici
description: SystemV, systemd, targets i serveis
---

## Índex

<nav class="toc" aria-label="Índex de la unitat" markdown="1">

1. [SystemV, Upstart i systemd](#1-systemv-upstart-i-systemd)
   1. [Runlevels o targets?](#11-runlevels-o-targets)
   2. [Quin sistema utilitza Ubuntu?](#12-quin-sistema-utilitza-ubuntu)
2. [SystemV](#2-systemv)
   1. [Directoris](#21-directoris)
   2. [Procés d'arrencada](#22-procés-darrencada)
3. [systemd](#3-systemd)
   1. [Directoris](#31-directoris)
   2. [`systemctl`](#32-systemctl)
   3. [Dependències](#33-dependències)
   4. [Modificar el target provisionalment](#34-modificar-el-target-provisionalment)
   5. [Modificar el target definitivament](#35-modificar-el-target-definitivament)
   6. [Afegir serveis a un target](#36-afegir-serveis-a-un-target)
   7. [Crear un target](#37-crear-un-target)
   8. [Crear un servei](#38-crear-un-servei)
   9. [Activitat: target personalitzat d'Eros](#39-activitat-target-personalitzat-deros)

</nav>

## Conceptes bàsics

- **Kernel:** administra el maquinari i els processos.
- **Aplicació:** programa que normalment interactua amb l'usuari.
- **Servei:** programa que s'executa en segon pla.
- **Procés:** instància d'un programa en execució.

## 1. SystemV, Upstart i systemd

SystemV organitza l'arrencada amb scripts i nivells d'execució. Upstart va introduir l'arrencada basada en esdeveniments, mentre que systemd treballa amb unitats, dependències i targets.

### 1.1 Runlevels o targets?

Els runlevels clàssics van del 0 al 6: el 0 apaga, l'1 és de rescat, del 2 al 5 són multiusuari i el 6 reinicia. systemd n'ofereix l'equivalent mitjançant targets.

![Consulta del nivell d'execució](imatges/3.png)

La comanda `runlevel` mostra `N 5`: no hi havia un nivell anterior i el sistema es troba al nivell 5, associat a l'entorn multiusuari gràfic.

### 1.2 Quin sistema utilitza Ubuntu?

![Consulta del manual d'init](imatges/1.png)

S'executa `man init` per consultar el gestor d'inici, però en aquesta captura la comanda acaba sense mostrar cap pàgina de manual.

![Enllaç simbòlic de /sbin/init](imatges/2.png)

`readlink -v /sbin/init` resol l'enllaç cap a `../lib/systemd/systemd`; per tant, Ubuntu utilitza systemd com a sistema d'inici.

## 2. SystemV

### 2.1 Directoris

![Scripts de servei de SystemV](imatges/5.png)

El directori `/etc/init.d` conté els scripts tradicionals dels serveis, com ara `cron`, `ssh`, `cups` o `rsync`.

![Directoris rc de SystemV](imatges/6.png)

Dins de `/etc` apareixen els directoris `rc0.d` fins a `rc6.d`, un per cada nivell d'execució.

![Enllaços del nivell 0](imatges/7.png)

`/etc/rc0.d` conté enllaços amb prefix `K`, utilitzats per aturar serveis quan el sistema entra al nivell d'apagada.

![Enllaços del nivell 5](imatges/8.png)

`/etc/rc5.d` mostra principalment enllaços amb prefix `S`, que indiquen els serveis que s'inicien en el nivell 5.

![Enllaços del nivell 1](imatges/9.png)

`/etc/rc1.d` conté enllaços `K` perquè el mode d'un sol usuari funciona amb un conjunt mínim de serveis.

### 2.2 Procés d'arrencada

![Reinici de cron amb init.d](imatges/4.png)

El servei `cron` es reinicia amb `/etc/init.d/cron restart`. El missatge indica que l'script delega l'acció a `systemctl`.

![Reinici mitjançant init 6](imatges/10.png)

La comanda `init 6` demana canviar al nivell 6 i, per tant, reiniciar el sistema.

## 3. systemd

### 3.1 Directoris

![Unitats instal·lades per defecte](imatges/11.png)

`/lib/systemd/system` conté les unitats proporcionades pels paquets del sistema: serveis, targets, sockets i temporitzadors.

![Unitats administrades localment](imatges/12.png)

`/etc/systemd/system` conté la configuració local i els enllaços dels serveis habilitats. Aquesta configuració té prioritat sobre la de `/lib`.

### 3.2 `systemctl`

![Targets actius](imatges/13.png)

`systemctl list-units --type=target` enumera els targets carregats i actius, com `graphical.target` i `multi-user.target`.

![Serveis actius](imatges/14.png)

`systemctl list-units --type=service` mostra els serveis carregats i el seu estat, diferenciant els que continuen executant-se dels que ja han finalitzat.

![Target predeterminat](imatges/15.png)

`systemctl get-default` confirma que el target d'arrencada predeterminat és `graphical.target`.

### 3.3 Dependències

![Dependències del target gràfic](imatges/25.png)

`systemctl list-dependencies graphical.target` mostra l'arbre d'unitats necessàries per arribar a l'entorn gràfic, inclòs `multi-user.target`.

![Temps d'inici de les unitats](imatges/27.png)

`systemd-analyze blame` ordena les unitats segons el temps que han trigat a iniciar-se; `NetworkManager-wait-online.service` és la més lenta de la llista visible.

![Temps total d'arrencada](imatges/28.png)

`systemd-analyze` resumeix els temps del kernel, l'initrd i l'espai d'usuari, i indica quan s'ha assolit `graphical.target`.

### 3.4 Modificar el target provisionalment

![Canvi temporal al mode de rescat](imatges/16.png)

`systemctl isolate rescue.target` canvia temporalment al mode de rescat sense modificar el target de la pròxima arrencada.

![Retorn temporal al mode gràfic](imatges/17.png)

`systemctl isolate graphical.target` torna a activar temporalment l'entorn gràfic.

### 3.5 Modificar el target definitivament

![Enllaç del target predeterminat](imatges/18.png)

La llista de targets mostra que `default.target` és un enllaç simbòlic cap a `graphical.target`.

![Target predeterminat de rescat](imatges/19.png)

Després d'eliminar l'enllaç anterior, es crea `default.target -> rescue.target`. El primer intent falla perquè l'enllaç encara existia.

![Restauració del target gràfic](imatges/20.png)

Finalment es torna a crear `default.target -> graphical.target`, restaurant l'arrencada gràfica predeterminada.

### 3.6 Afegir serveis a un target

![Instal·lació del servidor SSH](imatges/21.png)

`apt install ssh` inicia la instal·lació del servidor OpenSSH i de les seves dependències.

![Estat inicial d'SSH](imatges/22.png)

Abans d'habilitar-lo, `ssh.service` apareix com a `disabled` i `inactive (dead)`.

![Habilitació d'SSH](imatges/23.png)

`systemctl enable ssh` crea l'enllaç dins de `multi-user.target.wants`; `is-enabled` confirma que el servei queda habilitat per a l'arrencada.

![Servei SSH actiu](imatges/24.png)

Després de reiniciar-lo, `systemctl status ssh` mostra el servei habilitat i actiu, escoltant al port 22.

![Enllaç d'SSH en el nivell 5](imatges/26.png)

També apareix l'enllaç `S01ssh` a `/etc/rc5.d`, que manté la compatibilitat amb l'esquema de SystemV.

### 3.7 Crear un target

![Unitats requerides per multi-user.target](imatges/29.png)

El directori `multi-user.target.wants` mostra com s'associen les unitats a un target mitjançant enllaços, inclòs `ssh.service`.

![Definició de multi-user.target](imatges/30.png)

El fitxer `multi-user.target` serveix de model: declara dependències amb `Requires` i `After`, entra en conflicte amb `rescue.target` i permet l'ús d'`isolate`.

### 3.8 Crear un servei

![Creació de l'script rc.local](imatges/31.png)

Es crea `/etc/rc.local` amb un script que afegeix una línia `a` a `/etc/passwd`. És una prova amb permisos de root que no s'hauria d'utilitzar en un sistema real.

![Permís d'execució de rc.local](imatges/32.png)

`chmod +x rc.local` converteix el fitxer en executable perquè systemd el pugui iniciar.

![Definició de rc-local.service](imatges/33.png)

La unitat comprova que `/etc/rc.local` existeixi, l'executa amb `ExecStart` i queda associada a `multi-user.target`.

![Activació del servei rc-local](imatges/34.png)

El servei s'habilita, es reinicia i queda en estat `active (exited)`, amb codi de sortida correcte.

![Comprovació del resultat de l'script](imatges/35.png)

La consulta d'`/etc/passwd` mostra dues línies finals amb `a`, fet que confirma dues execucions de l'script. La resta de coincidències provenen d'altres línies que també contenen aquesta lletra.

### 3.9 Activitat: target personalitzat d'Eros

L'objectiu és crear `eros.target`, fer-lo predeterminat i executar-hi un servei propi amb permisos de root abans d'arribar a l'entorn gràfic.

![Comprovació del target inicial](imatges/36.png)

`systemctl get-default` confirma que el sistema arrenca inicialment amb `graphical.target`.

![Creació de l'script d'arrencada](imatges/37.png)

Es crea `eros-arrencada.sh`, que registra la data i l'usuari d'execució, i genera el fitxer de control `/run/eros-arrencada-ok`. Només root el pot executar.

![Definició del servei eros-arrencada](imatges/38.png)

El servei és de tipus `oneshot`, executa l'script com a root i queda associat a `eros.target`.

![Definició del target personalitzat](imatges/39.png)

`eros.target` requereix l'entorn gràfic i permet utilitzar-lo amb `systemctl isolate`.

![Validació i dependències del target](imatges/40.png)

Després de recarregar systemd, els fitxers es validen sense errors. En habilitar el servei es crea l'enllaç dins d'`eros.target.wants` i apareix a l'arbre de dependències.

![Prova manual del servei](imatges/41.png)

El servei finalitza correctament com a `active (exited)`. El registre mostra `uid=0(root)` i el fitxer de control confirma l'execució.

![Configuració d'eros.target com a predeterminat](imatges/42.png)

`systemctl set-default` canvia l'arrencada a `eros.target`; tant `get-default` com l'enllaç `default.target` ho confirmen.

![Reinici de la màquina virtual](imatges/43.png)

Es reinicia la màquina virtual per comprovar que el target i el servei funcionen durant una arrencada real.

![Comprovació final després del reinici](imatges/44.png)

Després del reinici, `eros.target` és el predeterminat, l'entorn gràfic continua actiu i el servei s'ha executat correctament com a root. El registre conté una entrada nova i el fitxer de control existeix.
