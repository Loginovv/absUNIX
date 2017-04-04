
![Alt text](https://raw.githubusercontent.com/vovakulikov/absUNIX/master/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%20%D0%BE%D1%82%202017-03-29%2018-43-37.png)

# Настройка сетевого фильтра, трансляция адресов.

## Цель работы:
* Овладение навыками управления сетевой фильтрацией и трансляцией адресов.
* Изучение команд управления системой IPTables.
* Изучение синтаксиса и основных операторов командного интерпретатора bash.
* Приобретение навыков по написанию командных скриптов управления службами.

### Вариант задания: *lxc20*

## Протокол  работы:


Определить список установленных сетевых устройств

![Alt text](https://raw.githubusercontent.com/vovakulikov/absUNIX/master/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%20%D0%BE%D1%82%202017-04-04%2020-55-58.png)

*Команда: lspci | grep Ethernet.*

Определить параметры сетевых интерфейсов.

![Alt text](https://raw.githubusercontent.com/vovakulikov/absUNIX/master/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%20%D0%BE%D1%82%202017-04-04%2020-57-08.png)

*Команда: ifconfig -a.*

Определить статические маршруты сети.

![Alt text](https://raw.githubusercontent.com/vovakulikov/absUNIX/master/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%20%D0%BE%D1%82%202017-04-04%2020-57-49.png)

*Команда route*

Определить режим маршрутизации ядра (включена или выключена).

![Alt text](https://raw.githubusercontent.com/vovakulikov/absUNIX/master/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%20%D0%BE%D1%82%202017-04-04%2020-59-15.png)

*Команда: cat /proc/sys/net/ipv4/ip_forward.*



Написать скрипт управления сервисом iptables в соответствии с заданием

*Конфигурационный файл ifcfg-eth1.*

    NAME="System eth1"
    DEVICE=eth1
    TYPE=Ethernet
    ONBOOT=no
    NM_CONTROLLED=no
    BOOTPROTO=none
    IPADDR=10.0.0.15
    NETMASK=255.255.255.0
    NETWORK=10.0.0.0
    BROADCAST=10.0.0.255
    GATEWAY=10.0.0.1
    IPV6INIT=no

*Скрипт управления службой для управления фильтрацией и трансляцией адресов*

    #!/bin/bash
	#description: 5_4341_KULIKOV

	PORT=22
	ET=eth1

	. /etc/init.d/functions

	if [ ! -f /etc/sysconfig/network ]; then
	exit 0

	fi
	. /etc/sysconfig/network

	[ "${NETWORKING}" = "no" ] && exit 0

	case "$1" in

	start)

	echo 1 > /proc/sys/net/ipv4/ip_forward
	iptables -t nat -A POSTROUTING -o $ET -j MASQUERADE
	iptables -A INPUT -i $ET -p TCP -s 0.0.0.0 —dport $PORT -j DROP
	;;

	stop)

	echo 0 > /proc/sys/net/ipv4/ip_forward
	iptables -F
	;;

	status)

	iptables -L
	ifconfig
	route -n 
	cat /proc/sys/net/ipv4/ip_forward
	;;
	*)
	echo $"Usage: $0 {start|stop|status}"
	exit 1
	esac

	exit 0



## Проверка работы
![Alt text](https://raw.githubusercontent.com/vovakulikov/absUNIX/master/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%20%D0%BE%D1%82%202017-04-04%2021-19-51.png)

Команда: service 5_4341_Kulikov start, service 5_4341_Kulikov status.


![Alt text](https://raw.githubusercontent.com/vovakulikov/absUNIX/master/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%20%D0%BE%D1%82%202017-04-04%2021-20-49.png)

Команда: service 5_4341_Kulikov stop, service 5_4341_Kulikov status.

# Вывод

iptables используется для проверки, модификации, перенаправления и отбрасывания пакетов. Код для фильтрации пакетов IPv4 уже встроен в ядро. Он основан на наборе таблиц, каждая из которых служит конкретной цели. Таблицы составляют набор предопределенных цепочек, которые, в свою очередь, содержат список правил, организованных в определенном порядке. Каждое правило состоит из критерия и действия, которое применяется к пакетам, подпадающим под этот критерий, если все условия выполнены. iptables является утилитой, которая позволяет работать с этими цепочками и правилами. Большинство пользователей находят IP маршрутизацию в Linux сложной и запутанной, однако, на практике наиболее распространенные варианты использования являются значительно менее сложными.
