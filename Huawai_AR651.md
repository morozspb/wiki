## Доступ на устройство

## Пользователи ользователя
* **display local-user all**  - Список пользователей. 
* Создать пользователя
## VLAN
* **Добавить vlan**\
\<Huawei> system-view\
[Huawei] vlan 100 \
[Huawei-vlan100] quit
[Huawei] interface GigabitEthernet0/0/0 \
[Huawei-GigabitEthernet0/0/0] port link-type access \
[Huawei-GigabitEthernet0/0/0] port default vlan 100 \
[Huawei-GigabitEthernet0/0/0] quit \
[Huawei] interface Vlanif100 \
[Huawei-Vlanif100] ip address 192.168.100.10 255.255.255.0 \
[Huawei-Vlanif100] undo shutdown \
[Huawei-Vlanif100] quit \
[Huawei] save \
Приверить: display ip interface brief; display current-configuration | include Vlanif\
**Настройка trunk-портов**\
system-view\
interface GigabitEthernet0/0/24\
port link-type trunk\
port trunk allow-pass vlan 10 20 30\
quit\
* **Удалить vlan**\
**Привести порт к vlan по умолчанию:**\
system-view\
interface GigabitEthernet0/0/1\
undo port default vlan\
**Удаление VLAN с trunk-порта:**\
system-view\
interface GigabitEthernet0/0/1\
undo port trunk allow-pass vlan 100\
ИЛИ\
undo port trunk allow-pass vlan all - Эта команда очистит весь список разрешённых VLAN. После неё порт будет пропускать только VLAN 1 (native VLAN по умолчанию).\
**Удаление VLAN с hybrid-порта**\
Сначала удалите untagged VLAN:\
system-view\
interface GigabitEthernet0/0/1\
undo port hybrid untagged vlan 100\
Затем удалите tagged VLAN:\
undo port hybrid tagged vlan 100\
*Проверьте результат командой display this в режиме интерфейса*\
**Удаление самого VLAN из базы коммутатора**\
system-view\
undo vlan 100
## Маршрутизация
Если коммутатор поддерживает маршрутизацию (уровень L3), можно настроить межсегментную маршрутизацию без внешнего роутера. Для этого создайте VLANIF-интерфейсы — виртуальные шлюзы для каждого VLAN.

system-view\
interface Vlanif 10\
ip address 192.168.10.1 255.255.255.0\
quit\
## Other
display current-configuration - Вывести конфигурацию.\
clear configuration interface GigabitEthernet0/0/1 - Сбросить порт.\
display saved-configuration - Проверить сохранённую конфигурацию можно командой
sys ip route-static 0.0.0.0 0.0.0.0 10.37.0.1 - разрешает доступ из других сетей
display ip routing-table
