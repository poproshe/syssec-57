Домашнее задание к занятию "Анализ уязвимостей Metasploitable" - Фамилия и имя студента
Задание 1
Приведите ответ в свободной форме.

Какие сетевые службы разрешены?
В Metasploitable открыты следующие сетевые службы:

Порт	Служба	Версия
21/tcp	FTP	vsftpd 2.3.4
22/tcp	SSH	OpenSSH 4.7p1
23/tcp	Telnet	Linux telnetd
25/tcp	SMTP	Postfix smtpd
53/tcp	DNS	ISC BIND 9.4.2
80/tcp	HTTP	Apache httpd 2.2.8
111/tcp	RPCBind	v2
139, 445/tcp	SMB	Samba smbd 3.X
512/tcp	rexec	netkit-rsh
513/tcp	rlogin	OpenBSD/Solaris rlogind
514/tcp	rsh	tcpwrapped
1099/tcp	Java RMI	GNU Classpath grmiregistry
1524/tcp	bindshell	Metasploitable root shell
2049/tcp	NFS	v2-4
2121/tcp	FTP	ProFTPD 1.3.1
3306/tcp	MySQL	5.0.51a
5432/tcp	PostgreSQL	8.3.0–8.3.7
5900/tcp	VNC	protocol 3.3
6000/tcp	X11	access denied
6667/tcp	IRC	UnrealIRCd
8009/tcp	AJP13	Apache Jserv
8180/tcp	HTTP	Apache Tomcat 5.5
Какие уязвимости были обнаружены?
vsftpd 2.3.4 — Backdoor Command Execution
Ссылка: https://www.exploit-db.com/exploits/17491
CVE-2011-2523. В дистрибутив vsftpd 2.3.4 был внедрён бэкдор: при отправке имени пользователя, содержащего :), на порту 6200 открывается root-шелл.

UnrealIRCd 3.2.8.1 — Backdoor Command Execution
Ссылка: https://www.exploit-db.com/exploits/16922
CVE-2010-2075. В исходный код UnrealIRCd 3.2.8.1 была добавлена вредоносная вставка, позволяющая выполнить произвольные команды через специальную строку.

Samba 3.0.20 < 3.0.25rc3 — Username map script Command Execution
Ссылка: https://www.exploit-db.com/exploits/16320
CVE-2007-2447. Уязвимость в опции username map script: через метасимволы оболочки в имени пользователя можно выполнить произвольные команды на сервере.

Задание 2
Приведите ответ в свободной форме.

Чем отличаются эти режимы сканирования с точки зрения сетевого трафика?
Режим	Что отправляет nmap	Ответ открытого порта	Ответ закрытого порта
SYN (-sS)	SYN	SYN-ACK	RST
FIN (-sF)	FIN	нет ответа	RST
Xmas (-sX)	FIN+PSH+URG	нет ответа	RST
UDP (-sU)	UDP (0 байт)	UDP-ответ или тишина	ICMP port unreachable
SYN-сканирование (-sS) — «полуоткрытое» сканирование. nmap отправляет TCP-пакет с флагом SYN. Если порт открыт, сервер отвечает SYN-ACK, а сканер сразу отправляет RST, разрывая соединение (полное TCP-соединение не устанавливается). Если порт закрыт — сервер отвечает RST. Самый быстрый и незаметный режим.

FIN-сканирование (-sF) — отправляется пакет только с флагом FIN. По стандарту TCP открытые порты должны игнорировать такой пакет (ответа нет), а закрытые — отвечать RST. Используется для обхода простых фильтров.

Xmas-сканирование (-sX) — то же, что FIN, но с флагами FIN+PSH+URG (пакет «светится всеми флагами», как рождественская ёлка). Открытые порты не отвечают, закрытые — отправляют RST.

UDP-сканирование (-sU) — отправляется UDP-пакет с нулевыми данными. Если порт открыт, служба может ответить UDP-пакетом (не всегда). Если порт закрыт — сервер отвечает ICMP-сообщением «Destination unreachable (Port unreachable)». UDP-сканирование медленнее и менее надёжно, так как UDP не гарантирует доставку.

Как отвечает сервер?
SYN: открытые порты отвечают SYN-ACK, закрытые — RST. Сканер завершает полуоткрытое соединение пакетом RST.

FIN и Xmas: открытые порты игнорируют пакет (ответа нет), закрытые — отправляют RST+ACK.

UDP: закрытые порты отвечают ICMP-сообщением «Port Unreachable»; открытые могут ответить UDP-пакетом или промолчать.

В Wireshark это видно по флагам TCP:

SYN-сканирование: 0x0002 (SYN) → 0x0012 (SYN, ACK) → 0x0004 (RST).

FIN-сканирование: 0x0001 (FIN) → 0x0014 (RST, ACK) от закрытых портов.

Xmas-сканирование: 0x0029 (FIN, PSH, URG) → 0x0014 (RST, ACK) от закрытых портов.

UDP-сканирование: UDP-пакеты → ICMP type 3 (Port Unreachable) от закрытых портов.

