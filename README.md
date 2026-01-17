# FRR.-GNS3
Из любого компьютера можно сделать полноценный маршрутизатор, если установить на него специализированное ПО для маршрутизации.
**Free Range Routing**(FRR) - одно из них. Этот пакет програмvного обеспечения для UNIX-подобных систем, в частности GNU/Linux и FreeBSD, обеспечивающий реализацию протоколов OSPF, RIP, BGP, IS-IS и многих других.

# Базовая настройка FRR маршрутизатора
В отличии от, например, коммутаторов маршрутизатор нельзя просто воткнуть в сеть, ожидая, что он сам как-то начнет работать, поэтому есть несколько этапов минимальной настройки маршрутизатора.
Вот они:
1. Задать ему имя
2. Установить пароль на привилегированный режим
3. Настроить IP-адреса на интерфейсах
4. Сохранить конфигурацию

Настройка FRR происходит через командную строку. Синтаксис FRR крайне напоминает таковой у CISCO, при этом иерархия режимов конфигурации полностью идентична.

Начнем же!

Всего есть 2 основных режима использования — **пользовательский**(урезанный набор команд для диагностики ) и **привилегированный**(по умолчанию доступны все команды для просмотра/диагностики). 

Внешний вид командной строки в **пользовательском** режиме:
```
frr>
```
И в привилегированном:
```
frr#
```

Для перехода в привилегированный режим можно использовать команду `enable`.
```
frr> enable
```

Набрав в консоли `?` можно узреть список всех доступных команд
```bash
frr# ?
  add         Add registration
  clear       Reset functions
  configure   Configuration from vty interface
  copy        Copy from one file to another
  debug       Debugging functions
  disable     Turn off privileged mode command
  enable      Turn on privileged mode command
  end         End current mode and change to enable mode
  exit        Exit current mode and down to previous mode
  find        Find CLI command matching a regular expression
  list        Print command list
  mtrace      Multicast trace route to multicast source
  no          Negate a command or set its defaults
  output      Direct vtysh output to file
  ping        Send echo messages
  quit        Exit current mode and down to previous mode
  rpki        Control rpki specific settings
  show        Show running system information
  terminal    Set terminal line parameters
  traceroute  Trace route to destination
  watchfrr    Watchfrr Specific sub-command
  write       Write running configuration to memory, network, or terminal
```
Также можно использовать `?` в качестве подсказки продолжения команды. Например, `show ?` выведет список команд, начинающихся с `show`.

Для начала работы по настройки чего бы то ни было следует перейти в режим глобальной конфигурации командой `configure terminal`
```
frr# configure terminal 
frr(config)# 
```
`(config)` перед `#` говорит о нахождении на верхнем уровне иерархии режима глобальной конфигурации. в этом режиме доступен следующий набор команд
```
frr(config)# ?
  access-list                  Access list entry
  agentx                       SNMP AgentX protocol settings
  allow-external-route-update  Allow FRR routes to be overwritten by external processes
  banner                       Set banner
  bfd                          Configure BFD peers
  bgp                          BGP information
  debug                        Debugging functions
  domainname                   Set system's domain name
  dump                         Dump packet
  enable                       Modify enable password parameters
  end                          End current mode and change to enable mode
  exit                         Exit current mode and down to previous mode
  find                         Find CLI command matching a regular expression
  fpm                          Forwarding Plane Manager configuration
  frr                          FRRouting global parameters
  hostname                     Set system's network name
  interface                    Select an interface to configure
  ip                           IP information
  ipv6                         IPv6 information
  key                          Authentication key management
  l2vpn                        Configure l2vpn commands
  line                         Configure a terminal line
  list                         Print command list
  log                          Logging control
  mac                          MAC address
  mpls                         Global MPLS configuration subcommands
  nexthop-group                Nexthop Group configuration
  nhrp                         Next Hop Resolution Protocol functions
  no                           Negate a command or set its defaults
  output                       Direct vtysh output to file
  password                     Modify the terminal connection password
  pbr                          Policy Based Routing
  pbr-map                      Create pbr-map or enter pbr-map command mode
  pseudowire                   Static pseudowire configuration
  quit                         Exit current mode and down to previous mode
  route-map                    Create route-map or enter route-map command mode
  router                       Enable a routing process
  router-id                    Manually set the router-id
  rpki                         Enable rpki and enter rpki configuration mode
  service                      Set up miscellaneous service
  terminal                     Set terminal line parameters
  username                     
  vni                          VNI corresponding to the DEFAULT VRF
  vrf                          Select a VRF to configure
  vrrp                         Virtual Router Redundancy Protocol
  zebra                        Zebra information
```

Следуя шагам по минимальной настройке зададим нашему роутеру имя. Для этого используем команду `hostname <имя>` 
```
frr(config)# hostname GeorgeDroid
GeorgeDroid(config)#
```
Пароль для доступа к консоли в привилегированном режиме, в котором доступны команды настройки, задается командой `enable password`. Чтобы пароли хранились в зашифрованном виде, нужно включить сервис шифрования паролей командой `service password-encryption`
```
GeorgeDroid(config)# enable password pass
GeorgeDroid(config)# service password-encryption
GeorgeDroid(config)#
```
Далее нам следует настроить интерфейсы маршрутизатора. Это ключевой этап, так как в процессе настройки интерфейсов роутер «узнает» о сетях, напрямую подключенных к нему, добавляя новые записи в свою таблицу маршрутизации.

Нужно перейти в режим конфигурации интерфейса командой `interface <имя интерфейса>`. Например, если необходимо настроить интерфейс eth0, команда будет выглядеть следующим образом:

```
GeorgeDroid(config)# interface eth0
GeorgeDroid(config-if)# 
```
`(config-if)` перед символом `#` говорит о том, что сейчас мы находимся на следующем уровне иерархии - в режиме конфигурации интерфейса.

Зададим адрес нашему интерфейсу командой `ip address <адрес/маска>.
```
GeorgeDroid(config-if)# ip address 10.0.2.1/24
````

Командой `no shutdown` (сокращенно `no sh`) программно включим интерфейс.
```
GeorgeDroid(config-if)# no shutdown 
```

После всех настроек необходимо сохранить конфигурацию. Для этого предварительно нужно вернуться из режима конфигурации в первоначальный привилегированный режим. Вернуться на один режим выше командой `exit` или сочетание `Ctr + Z`. Для быстрого перехода можно использовать команду `end`

```
GeorgeDroid(config-if)# end
GeorgeDroid#
GeorgeDroid# write
```

Текущую конфигурацию полностью можно просмотреть командой `show running-config` (`sh run`), а стартовую - `show startup-config` (`sh star`).

Просмотреть список имен всех интерфейсов и их статус можно командой `sh interface brief`:

```
GeorgeDroid# sh interface brief 
Interface       Status  VRF             Addresses
---------       ------  ---             ---------
eth0            up      default         10.0.2.1/24
eth1            down    default         
eth2            down    default         
eth3            down    default         
eth4            down    default         
eth5            down    default         
eth6            down    default         
eth7            down    default         
lo              up      default         
lo0             down    default         
```
Для просмотра таблицы маршрутизации можно воспользоваться командой `show ip route`:
```
GeorgeDroid# sh ip route
Codes: K - kernel route, C - connected, S - static, R - RIP,
       O - OSPF, I - IS-IS, B - BGP, E - EIGRP, N - NHRP,
       T - Table, v - VNC, V - VNC-Direct, A - Babel, D - SHARP,
       F - PBR, f - OpenFabric,
       > - selected route, * - FIB route, q - queued, r - rejected, b - backup

C>* 10.0.2.0/24 is directly connected, eth0, 02:05:19
```
# Пример
<img width="1306" height="326" alt="FRR  Базовая настройка на примере GNS3  Рисунок 1" src="https://github.com/user-attachments/assets/bba1139f-9635-492e-9dbe-6fa28b290ab3" />

Как видно из рисунка — имеется две локальных сети LAN1(10.0.1.0/24) и LAN2(10.0.2.1/24). В каждой из них есть по ПК с настройками, подписанными на рисунке, и роутер с двумя интерфейсами.

### Настройка конечных устройств
Для данного примера хватит задать IP адрес и шлюз по умолчанию. Сделать это можно через файл `/etc/network/interfaces`

**На VM1:**
```
fentanyl@debian:/home/debian# nano /etc/network/interfaces
```

В самом файле пропишем следующее
```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# Static config for ens4
auto ens4 # автоматическое поднятие интерфейса
iface ens4 inet static # статитечкий ipv4 адрес
        address 10.0.1.2
        netmask 255.255.255.0
        gateway 10.0.1.1 # в качестве шлюза назначаем адрес роутера на eth0

```

Перезагрузим сетевые настройки
```
fentanyl@debian:/home/debian# systemctl restart networking
```
Проверим корректность настроек:
```
fentanyl@debian:/home/debian# ip a show ens4
2: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 0c:7d:fc:45:00:00 brd ff:ff:ff:ff:ff:ff
    altname enp0s4
    inet 10.0.1.2/24 brd 10.0.1.255 scope global ens4
       valid_lft forever preferred_lft forever
    inet6 fe80::e7d:fcff:fe45:0/64 scope link
       valid_lft forever preferred_lft forever
```

На **VM2** проведем те же действия, но запишем в `/etc/networ/interfaces` следующее
```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# Static config for ens4
auto ens4
iface ens4 inet static
        address 10.0.2.2
        netmask 255.255.255.0
        gateway 10.0.2.1 # в качестве шлюза назначаем адрес роутера на eth1
        
```
# Настройка FRR
```
frr# conf t
frr(config)# hostname geroge_droid_frr
geroge_droid_frr(config)# enable password fentanyl
geroge_droid_frr(config)# service password-encryption 
geroge_droid_frr(config)# int eth0
geroge_droid_frr(config-if)# ip addr 10.0.1.1/24
geroge_droid_frr(config-if)# exit
geroge_droid_frr(config)# int eth1
geroge_droid_frr(config-if)# ip addr 10.0.2.1/24
geroge_droid_frr(config-if)# end
geroge_droid_frr# write
```
# Проверим связанность
```
fentanyl@debian:/home/debian# ping -c 3 10.0.2.1
PING 10.0.2.1 (10.0.2.1) 56(84) bytes of data.
64 bytes from 10.0.2.1: icmp_seq=1 ttl=64 time=2.26 ms
64 bytes from 10.0.2.1: icmp_seq=2 ttl=64 time=1.06 ms
64 bytes from 10.0.2.1: icmp_seq=3 ttl=64 time=0.833 ms

--- 10.0.2.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 0.833/1.384/2.256/0.623 ms

```
Ура! Все работает!
