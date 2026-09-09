# Настройка MikroTik
## VLAN на MikroTik
Сценарий: один MikroTik, два VLAN — VLAN 10 (офис, 192.168.10.0/24) и VLAN 20 (гости, 192.168.20.0/24). Порт ether1 — uplink/trunk, ether2 — access для офиса, ether3 — access для гостей.

Важно: порядок имеет значение. Сначала настраиваете всё, потом включаете vlan-filtering=yes. Не наоборот — иначе потеряете доступ к устройству.

### Шаг 1. Создаём bridge

***/interface bridge add name=bridge1 vlan-filtering=no comment="Main bridge"***

Важно: **vlan-filtering=no** на старте. Сначала всё настраиваем, потом включаем фильтрацию.

### Шаг 2. Добавляем порты в bridge

***/interface bridge port***

***add bridge=bridge1 interface=ether1 comment="Uplink/Trunk"***

***add bridge=bridge1 interface=ether2 pvid=10 comment="Office access"***

***add bridge=bridge1 interface=ether3 pvid=20 comment="Guest access"***

pvid (Port VLAN ID) — это VLAN, который будет назначен нетегированному трафику на этом порту. На trunk-порту pvid обычно остаётся 1 (дефолт).

### Шаг 3. Настраиваем VLAN-таблицу bridge

***/interface bridge vlan***

***add bridge=bridge1 vlan-ids=10 tagged=ether1,bridge1 untagged=ether2***

***add bridge=bridge1 vlan-ids=20 tagged=ether1,bridge1 untagged=ether3***

Обратите внимание на ***tagged=bridge1*** — это обязательный параметр. Он разрешает CPU MikroTik (и VLAN-интерфейсу поверх bridge) получать трафик этого VLAN. Без него IP-адрес на vlan10 будет назначен, но пинг не пойдёт.

### Шаг 4. Создаём VLAN-интерфейсы для маршрутизации

***/interface vlan***

***add interface=bridge1 vlan-id=10 name=vlan10***

***add interface=bridge1 vlan-id=20 name=vlan20***

*В RouterOS 7 VLAN-интерфейс создаётся поверх bridge, а не поверх физического порта. Это принципиальное отличие от ROS 6.*

### Шаг 5. Назначаем IP-адреса

***/ip address***

***add address=192.168.10.1/24 interface=vlan10***

***add address=192.168.20.1/24 interface=vlan20***

### Шаг 6. Включаем VLAN Filtering***

***/interface bridge set bridge1 vlan-filtering=yes***

После этой команды трафик между VLAN прекратится без явной маршрутизации. Убедитесь, что ваш management-доступ тоже учтён (см. раздел про Management VLAN).

### Шаг 7. DHCP-серверы

***/ip pool***

***add name=pool-vlan10 ranges=192.168.10.100-192.168.10.200***

***add name=pool-vlan20 ranges=192.168.20.100-192.168.20.200***

***/ip dhcp-server***

***add interface=vlan10 address-pool=pool-vlan10 name=dhcp-vlan10 disabled=no***

***add interface=vlan20 address-pool=pool-vlan20 name=dhcp-vlan20 disabled=no***

***/ip dhcp-server network***

***add address=192.168.10.0/24 gateway=192.168.10.1 dns-server=192.168.10.1***

***add address=192.168.20.0/24 gateway=192.168.20.1 dns-server=192.168.20.1***

### Диагностика «VLAN
1. Проверяем VLAN-таблицу — всё ли прописано?\
/interface bridge vlan print detail

2. Проверяем PVID на проблемном порту\
/interface bridge port print detail where interface=ether2

3. Видит ли bridge MAC-адрес клиента?\
/interface bridge host print

4. Пинг с gateway VLAN10 до клиента\
/ping 192.168.10.100 src-address=192.168.10.1 count=5

5. Если не пингуется — смотрим трафик в реальном времени\
/tool torch interface=vlan10
## Резервирование
Полный экспорт (отличается от заводских настроек): /export compact file=config.

## Other
Hostname - System → Identity; /system identity set name=Hostname

## Firewall
### Изоляция Vlan
* **Routing**

/ip route rule add action=unreachable dst-address=192.168.10.0/24 src-address=192.168.11.0/24 
* **Firewall**
 ip firewall filter\
add action=drop chain=forward dst-address=172.16.10.0/28 src-address=172.16.11.0/28
 
* **RAW tables**
*Если пакет отбрасывается в RAW, он не создаёт записи в таблице conntrack и не нагружает процессор и память.*\
**У RAW есть важные особенности, которые стоит учитывать:**

1. Нет доступа к состоянию соединения (connection-state). Вы не можете написать правило вроде connection-state=established, потому что conntrack ещё не обработал пакет. 

2. Нет поддержки маркировки соединений (connection marking). Это ограничивает возможности сложной фильтрации. 

3. Единственное доступное действие — notrack. Оно явно указывает системе не отправлять пакет на отслеживание соединения. 
4. Не происходит дефрагментация пакетов, помеченных как notrack. 

**Когда использовать RAW**

1. RAW идеально подходит для stateless-фильтрации — блокировки по IP, портам, протоколу и флагам. Вот примеры применения:
Защита от DDoS. Отбрасывание пакетов от IP из чёрного списка до того, как они создадут записи в conntrack. 

2. Защита от SYN-флуда. Ограничение количества новых TCP-подключений с одного IP. 

3. Динамическая блокировка. Правила в RAW могут добавлять адреса в адресные списки (address-list) при обнаружении подозрительного поведения (например, сканирования портов). 

4. Исправление проблем с протоколами. Например, чтобы избежать проблем с соседством OSPF из-за отслеживания соединений, можно пометить OSPF-пакеты как notrack. 

**ip firewall filter add action=drop chain=prerouting src-address=172.16.11.0/28 dst-address=172.16.10.0/28**
### Разрешить доступ между определёнными IP
ip firewall filter\
add action=accept chain=forward dst-address=172.16.11.14 src-address=172.16.10.3\
add action=accept chain=forward dst-address=172.16.10.3 src-address=172.16.11.14