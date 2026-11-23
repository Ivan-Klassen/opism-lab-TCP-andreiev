<!--
Бланк звіту до лабораторної роботи «Транспортний рівень».
Скопіювати цей файл у власний публічний репозиторій звіту як README.md і заповнити.
Вивід команд вставляти текстом у блоки коду, а не знімками екрана.
Порожні поля не допускаються: якщо щось не вдалося, записати, що саме і чому.
-->

# Лабораторна робота 3. Транспортний рівень

| Поле | Значення |
|---|---|
| ПІБ | Андреєв Іван Олегович |
| Група | ІПЗ_2.01 |
| Ідентифікатор | se-201-01 |
| Дата виконання | 23.11.2026 |
| Середовище | Docker на Debian Linux |
| Рядок `tcp_congestion_control` з виводу `up.sh` | `se-201-01  tcp_congestion_control=cubic` |
| Версія ядра | 6.12.116+deb13-amd64 |

## Параметри варіанта

<!-- Переписати з картки stand/cards/<id>.md. -->

| Параметр | Значення |
|---|---|
| Ваш вузол (IP) | 10.53.1.1 |
| Порти для діагностики | 20010, 20019, 20014, 20011, 20017 |
| Порт iperf3 | 5201 |
| Вікно для завдання 5, байт | 65536 |
| Втрати для завдання 6 | 0.5% |

<!-- Якщо стенд перезапускався під час роботи, вказати, після якого завдання. -->

## Завдання 1. Порти і встановлення з'єднання

Вивід кроку 1.1:

```text
root@se-201-01:/# sysctl net.ipv4.ip_local_port_range
curl -s --limit-rate 200k -o /dev/null http://$SRV/big.bin &
curl -s --limit-rate 200k -o /dev/null http://$SRV/big.bin &
sleep 1; ss -tn dst 10.53.0.80; kill %1 %2
net.ipv4.ip_local_port_range = 32768    60999
[1] 93
[2] 94
State                      Recv-Q                      Send-Q                                           Local Address:Port                                              Peer Address:Port                      Process                      
ESTAB                      115768                      0                                                    10.53.1.1:45212                                               10.53.0.80:80                                                     
ESTAB                      115320                      0                                                    10.53.1.1:45210                                               10.53.0.80:80                                        
```

| З'єднання | Адреса клієнта | Порт клієнта | Адреса сервера | Порт сервера |
|---|---|---|---|---|
| 1 | 10.53.1.1 | 45212 | 10.53.0.80 | 80 |
| 2 | 10.53.1.1 | 45210 | 10.53.0.80 | 80 |

Вивід кроку 1.2:

```text
root@se-201-01:/# cd /lab/work               
timeout 5 tcpdump -ni eth0 -S -ttt 'host 10.53.0.80 and tcp port 80' > t1.txt 2>/dev/null &
sleep 1
curl -s -o /dev/null -w 'connect=%{time_connect} starttransfer=%{time_starttransfer} total=%{time_total}\n' http://10.53.0.80/small.txt
wait; sed -E 's/, options \[[^]]*\]//' t1.txt
[1]-  Terminated              curl -s --limit-rate 200k -o /dev/null http://$SRV/big.bin  (wd: /)
(wd now: /lab/work)
[2]+  Terminated              curl -s --limit-rate 200k -o /dev/null http://$SRV/big.bin  (wd: /)
(wd now: /lab/work)
[1] 99
connect=0.025297 starttransfer=0.052423 total=0.052519
[1]+  Exit 124                timeout 5 tcpdump -ni eth0 -S -ttt 'host 10.53.0.80 and tcp port 80' > t1.txt 2> /dev/null
 00:00:00.000000 IP 10.53.1.1.58434 > 10.53.0.80.80: Flags [S], seq 2877147302, win 64240, length 0
 00:00:00.000052 IP 10.53.0.80.80 > 10.53.1.1.58434: Flags [S.], seq 582609451, ack 2877147303, win 65160, length 0
 00:00:00.025050 IP 10.53.1.1.58434 > 10.53.0.80.80: Flags [.], ack 582609452, win 502, length 0
 00:00:00.000067 IP 10.53.1.1.58434 > 10.53.0.80.80: Flags [P.], seq 2877147303:2877147385, ack 582609452, win 502, length 82: HTTP: GET /small.txt HTTP/1.1
 00:00:00.000020 IP 10.53.0.80.80 > 10.53.1.1.58434: Flags [.], ack 2877147385, win 509, length 0
 00:00:00.001946 IP 10.53.0.80.80 > 10.53.1.1.58434: Flags [P.], seq 582609452:582609716, ack 2877147385, win 509, length 264: HTTP: HTTP/1.1 200 OK
 00:00:00.025031 IP 10.53.1.1.58434 > 10.53.0.80.80: Flags [.], ack 582609716, win 501, length 0
 00:00:00.000296 IP 10.53.1.1.58434 > 10.53.0.80.80: Flags [F.], seq 2877147385, ack 582609716, win 501, length 0
 00:00:00.000048 IP 10.53.0.80.80 > 10.53.1.1.58434: Flags [F.], seq 582609716, ack 2877147386, win 509, length 0
 00:00:00.025022 IP 10.53.1.1.58434 > 10.53.0.80.80: Flags [.], ack 582609717, win 501, length 0
```

<!-- Один рядок на кожен рядок дампу 1.2. Відносний номер = абсолютний номер мінус початковий номер послідовності тієї сторони, яка надіслала сегмент. Напрям: К→С або С→К. -->

| № | Напрям | Прапорці | `seq` (відносний) | `ack` (відносний) | `length` |
|---|---|---|---|---|---|
| 1 | | | | | |

| Показник | Значення |
|---|---|
| RTT за значенням `connect`, мс | |
| Інтервал між рядками 1 і 2 дампу, мс | |
| `starttransfer`, мс, і скільки це RTT | |
| Хто першим надіслав FIN | |

1. Чому `ack` у другому сегменті більший за `seq` першого сегмента на 1, хоча перший сегмент не містить даних?

2. Значення `connect` відповідає затримці стенда, а інтервал між рядками 1 і 2 дампу майже нульовий. Де саме в контейнері `tcpdump` бачить вихідний сегмент і як це пояснює розбіжність?

3. Який з двох портів з'єднання 1.1 є тимчасовим? Чи може сервер одночасно мати два з'єднання з вашим вузлом на тому самому своєму порту і чому?

## Завдання 2. Закриття з'єднання і стан TIME_WAIT

Вивід кроків 2.1 і 2.2:

```text
root@se-201-01:/lab/work# for i in $(seq 20); do curl -s -o /dev/null http://$SRV/; done
ss -tan state time-wait '( dport = :80 )' | tail -n +2 | wc -l
for i in $(seq 20); do curl -s -o /dev/null http://$SRV:8080/; done
ss -tan state time-wait '( dport = :8080 )' | tail -n +2 | wc -l

timeout 4 tcpdump -ni eth0 -S 'host 10.53.0.80 and tcp[tcpflags] & (tcp-fin|tcp-rst) != 0' > t2.txt 2>/dev/null &
sleep 1; curl -s -o /dev/null http://$SRV/; curl -s -o /dev/null http://$SRV:8080/; wait
sed -E 's/, options \[[^]]*\]//' t2.txt
curl -sv http://$SRV:8080/ 2>&1 | grep -E '^< |Closing|left intact'
20
0
[1] 192
[1]+  Exit 124                timeout 4 tcpdump -ni eth0 -S 'host 10.53.0.80 and tcp[tcpflags] & (tcp-fin|tcp-rst) != 0' > t2.txt 2> /dev/null
10:06:47.395151 IP 10.53.1.1.51504 > 10.53.0.80.80: Flags [F.], seq 3063142461, ack 3543975156, win 501, length 0
10:06:47.395204 IP 10.53.0.80.80 > 10.53.1.1.51504: Flags [F.], seq 3543975156, ack 3063142462, win 509, length 0
10:06:47.454259 IP 10.53.0.80.8080 > 10.53.1.1.52562: Flags [F.], seq 81156769, ack 1948703646, win 509, length 0
10:06:47.479338 IP 10.53.1.1.52562 > 10.53.0.80.8080: Flags [F.], seq 1948703646, ack 81156770, win 502, length 0

< HTTP/1.0 200 OK
< Content-Type: text/plain
< 
* Closing connection
```

Вивід кроків 2.3–2.5:

```text
root@se-201-01:/lab/work# ss -tan state time-wait '( dport = :80 )' | tail -n +2 | wc -l
U=http://$SRV/
curl -s -w '%{num_connects} ' $(for i in $(seq 20); do echo "-o /dev/null $U"; done); echo
ss -tan state time-wait '( dport = :80 )' | tail -n +2 | wc -l

for i in 1 2 3 4 5 6; do
  curl -sS -o /dev/null --local-port 40100-40104 -w "local port %{local_port}\n" http://$SRV/ 2>&1
done
ss -tano state time-wait '( sport >= :40100 and sport <= :40104 )'

sleep 70
curl -sS -o /dev/null --local-port 40100-40104 -w "local port %{local_port}\n" http://$SRV/
22
1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 
22
local port 40100
local port 40101
local port 40102
local port 40103
local port 40104
curl: (45) bind failed with errno 98: Address already in use
local port 0
Recv-Q                       Send-Q                                               Local Address:Port                                                Peer Address:Port                       Process                                         
0                            0                                                        10.53.1.1:40104                                                 10.53.0.80:80                          timer:(timewait,59sec,0)                       
0                            0                                                        10.53.1.1:40102                                                 10.53.0.80:80                          timer:(timewait,59sec,0)                       
0                            0                                                        10.53.1.1:40101                                                 10.53.0.80:80                          timer:(timewait,59sec,0)                       
0                            0                                                        10.53.1.1:40103                                                 10.53.0.80:80                          timer:(timewait,59sec,0)                       
0                            0                                                        10.53.1.1:40100                                                 10.53.0.80:80                          timer:(timewait,59sec,0)                       
local port 40100
```

| Показник | Порт 80 | Порт 8080 |
|---|---|---|
| Кількість `TIME_WAIT` на вашому вузлі після 20 запитів | 20 | 0 |
| Хто першим надіслав FIN | клієнт (10.53.1.1) | сервер (10.53.0.80) |
| Як клієнт визначає кінець відповіді | за заголовком Content-Length | за закриттям з'єднання сервером (FIN) |

| Показник | Значення |
|---|---|
| `num_connects` для 20 запитів однією командою (крок 2.3) | 1 (для першого, 0 для решти 19) |
| Приріст `TIME_WAIT` у кроці 2.3 | 	0 (було 22 і залишилось 22 через сплив таймера старого сокета) |
| Помилка шостого запиту в кроці 2.4 | curl: (45) bind failed with errno 98: Address already in use |
| Значення таймера `timewait` у кроці 2.4 | 59sec |
| Результат кроку 2.5 | успішно (local port 40100) |

1. Чому після запитів до порту 8080 на вашому вузлі немає з'єднань у стані `TIME_WAIT`? На якому вузлі вони є?

2. У ситуації 2 з лекції помилка має текст `Cannot assign requested address`, у кроці 2.4 — `Address already in use`. Причина в обох випадках однакова. Чим тоді пояснюється різниця в тексті? Підказка: порівняйте, хто обирає локальний порт.

3. Розрахуйте найбільшу сталу частоту нових з'єднань до порту 80 сервера, якщо клієнт використовує весь діапазон з кроку 1.1, а з'єднання закриває першим.

4. Як зміниться відповідь на питання 3 після переходу на один `curl` з багатьма запитами, як у кроці 2.3?

## Завдання 3. Діагностика відмов

Вивід кроку 3.1:

```text
root@se-201-01:/lab/work# cd /lab/work
P="20010 20019 20014 20011 20017"
timeout 40 tcpdump -ni eth0 -w t3.pcap 'host 10.53.0.80 and (tcp or icmp)' 2>/dev/null &
sleep 1
for p in $P; do
  echo "=== $p"
  curl -sS -o /dev/null --connect-timeout 5 -m 8 -d 'order=42' \
    -w 'connect=%{time_connect} total=%{time_total} code=%{http_code}\n' http://srv.se-201-01.lab:$p/ 2>&1
done
wait
[1] 247
=== 20010
curl: (28) Failed to connect to srv.se-201-01.lab port 20010 after 5001 ms: Timeout was reached
connect=0.000000 total=5.002001 code=000
=== 20019
connect=0.051087 total=0.076759 code=200
=== 20014
curl: (7) Failed to connect to srv.se-201-01.lab port 20014 after 51 ms: Couldn't connect to server
connect=0.000000 total=0.051425 code=000
=== 20011
curl: (7) Failed to connect to srv.se-201-01.lab port 20011 after 51 ms: Couldn't connect to server
connect=0.000000 total=0.051219 code=000
=== 20017
curl: (28) Operation timed out after 8002 milliseconds with 0 bytes received
connect=0.051018 total=8.002602 code=000
[1]+  Exit 124                timeout 40 tcpdump -ni eth0 -w t3.pcap 'host 10.53.0.80 and (tcp or icmp)' 2> /dev/null
```

Вивід кроку 3.2:

```text
root@se-201-01:/lab/work# for p in $P; do
  echo "=== $p"
  tcpdump -r t3.pcap -nn -S -ttt "tcp port $p or (icmp and icmp[30:2] = $p)" 2>/dev/null \
    | sed -E 's/, options \[[^]]*\]//'
done
=== 20010
 00:00:00.000000 IP 10.53.1.1.52724 > 10.53.0.80.20010: Flags [S], seq 3147142396, win 64240, length 0
 00:00:01.005260 IP 10.53.1.1.52724 > 10.53.0.80.20010: Flags [S], seq 3147142396, win 64240, length 0
 00:00:01.023990 IP 10.53.1.1.52724 > 10.53.0.80.20010: Flags [S], seq 3147142396, win 64240, length 0
 00:00:01.024000 IP 10.53.1.1.52724 > 10.53.0.80.20010: Flags [S], seq 3147142396, win 64240, length 0
 00:00:01.027999 IP 10.53.1.1.52724 > 10.53.0.80.20010: Flags [S], seq 3147142396, win 64240, length 0
=== 20019
 00:00:00.000000 IP 10.53.1.1.33756 > 10.53.0.80.20019: Flags [S], seq 1477082756, win 64240, length 0
 00:00:00.000081 IP 10.53.0.80.20019 > 10.53.1.1.33756: Flags [S.], seq 130715729, ack 1477082757, win 65160, length 0
 00:00:00.025040 IP 10.53.1.1.33756 > 10.53.0.80.20019: Flags [.], ack 130715730, win 502, length 0
 00:00:00.000066 IP 10.53.1.1.33756 > 10.53.0.80.20019: Flags [P.], seq 1477082757:1477082920, ack 130715730, win 502, length 163
 00:00:00.000015 IP 10.53.0.80.20019 > 10.53.1.1.33756: Flags [.], ack 1477082920, win 508, length 0
 00:00:00.000476 IP 10.53.0.80.20019 > 10.53.1.1.33756: Flags [P.], seq 130715730:130715798, ack 1477082920, win 508, length 68
 00:00:00.000074 IP 10.53.0.80.20019 > 10.53.1.1.33756: Flags [F.], seq 130715798, ack 1477082920, win 508, length 0
 00:00:00.024959 IP 10.53.1.1.33756 > 10.53.0.80.20019: Flags [.], ack 130715798, win 502, length 0
 00:00:00.000131 IP 10.53.1.1.33756 > 10.53.0.80.20019: Flags [F.], seq 1477082920, ack 130715799, win 502, length 0
 00:00:00.000039 IP 10.53.0.80.20019 > 10.53.1.1.33756: Flags [.], ack 1477082921, win 508, length 0
=== 20014
 00:00:00.000000 IP 10.53.1.1.57288 > 10.53.0.80.20014: Flags [S], seq 2093219359, win 64240, length 0
 00:00:00.000067 IP 10.53.0.80 > 10.53.1.1: ICMP 10.53.0.80 tcp port 20014 unreachable, length 68
=== 20011
 00:00:00.000000 IP 10.53.1.1.46272 > 10.53.0.80.20011: Flags [S], seq 4242205724, win 64240, length 0
 00:00:00.000058 IP 10.53.0.80.20011 > 10.53.1.1.46272: Flags [R.], seq 0, ack 4242205725, win 0, length 0
=== 20017
 00:00:00.000000 IP 10.53.1.1.60936 > 10.53.0.80.20017: Flags [S], seq 1037064179, win 64240, length 0
 00:00:00.000064 IP 10.53.0.80.20017 > 10.53.1.1.60936: Flags [S.], seq 3498323426, ack 1037064180, win 65160, length 0
 00:00:00.025054 IP 10.53.1.1.60936 > 10.53.0.80.20017: Flags [.], ack 3498323427, win 502, length 0
 00:00:00.000074 IP 10.53.1.1.60936 > 10.53.0.80.20017: Flags [P.], seq 1037064180:1037064343, ack 3498323427, win 502, length 163
 00:00:00.000022 IP 10.53.0.80.20017 > 10.53.1.1.60936: Flags [.], ack 1037064343, win 508, length 0
 00:00:07.952202 IP 10.53.1.1.60936 > 10.53.0.80.20017: Flags [F.], seq 1037064343, ack 3498323427, win 502, length 0
 00:00:00.043568 IP 10.53.0.80.20017 > 10.53.1.1.60936: Flags [.], ack 1037064344, win 508, length 0
```

| Порт | Помилка `curl` | `connect`, мс | `total`, мс | Що видно в дампі | Спосіб поведінки | Що перевіряти власнику сервісу |
|---|---|---|---|---|---|---|
| | | | | | | |
| | | | | | | |
| | | | | | | |
| | | | | | | |
| | | | | | | |

1. Для кожного порту з помилкою `Couldn't connect to server` визначте за дампом, хто відхилив з'єднання і яким способом. Якщо такий порт у вашому варіанті один, опишіть, як у дампі виглядав би другий спосіб відхилення.

2. Два порти дають помилку `curl` з кодом 28. Чим вони відрізняються за значенням `connect` і за дампом? Що це означає для пошуку причини?

3. Для порту, на якому сервер не відповідає: чи отримав вузол `srv` тіло `order=42`? Наведіть рядок дампу. Чи можна з цього зробити висновок, що замовлення оброблено?

4. Для порту, на якому з'єднання скасовано: на якому етапі надійшов сегмент RST? Що клієнт може сказати про виконання запиту?

5. Скільки сегментів SYN надіслано на порт, пакети на який відкидаються, і з якими інтервалами? Чому інтервали саме такі в цій версії ядра?

## Завдання 4. Потік байтів і дейтаграми

Вивід кроків 4.1 і 4.2:

```text
root@se-201-01:/lab/work# cd /lab/work
timeout 4 tcpdump -ni eth0 -S 'host 10.53.0.80 and tcp port 9000 and tcp[tcpflags] & tcp-push != 0' > t4.txt 2>/dev/null &
sleep 1; msg-client tcp; wait
sed -E 's/, options \[[^]]*\]//' t4.txt

timeout 4 tcpdump -ni eth0 'host 10.53.0.80 and udp port 9000' > t4u.txt 2>/dev/null &
sleep 1; msg-client udp; wait
cat t4u.txt
[1] 270
connected from port 43636; waiting 0.5 s before reading
recv #1: 23 bytes: 1:ZULU;2:TANGO;3:HOTEL;
connection closed by server; recv() calls with data: 1
[1]+  Exit 124                timeout 4 tcpdump -ni eth0 -S 'host 10.53.0.80 and tcp port 9000 and tcp[tcpflags] & tcp-push != 0' > t4.txt 2> /dev/null
10:21:45.276590 IP 10.53.0.80.9000 > 10.53.1.1.43636: Flags [P.], seq 1585177217:1585177224, ack 4200584127, win 510, length 7
10:21:45.297071 IP 10.53.0.80.9000 > 10.53.1.1.43636: Flags [P.], seq 1585177224:1585177232, ack 4200584127, win 510, length 8
10:21:45.317355 IP 10.53.0.80.9000 > 10.53.1.1.43636: Flags [P.], seq 1585177232:1585177240, ack 4200584127, win 510, length 8

[1] 275
sent 1 datagram from port 57023; waiting 0.5 s before reading
recvfrom #1: 7 bytes: 1:ZULU;
recvfrom #2: 8 bytes: 2:TANGO;
recvfrom #3: 8 bytes: 3:HOTEL;
no more datagrams for 2 s; recvfrom() calls with data: 3
[1]+  Exit 124                timeout 4 tcpdump -ni eth0 'host 10.53.0.80 and udp port 9000' > t4u.txt 2> /dev/null
10:21:49.304570 IP 10.53.1.1.57023 > 10.53.0.80.9000: UDP, length 5
10:21:49.304804 IP 10.53.0.80.9000 > 10.53.1.1.57023: UDP, length 7
10:21:49.304837 IP 10.53.0.80.9000 > 10.53.1.1.57023: UDP, length 8
10:21:49.304851 IP 10.53.0.80.9000 > 10.53.1.1.57023: UDP, length 8
```

Вивід кроків 4.3 і 4.4:

```text
root@se-201-01:/lab/work# timeout 5 tcpdump -ni eth0 '(dst host 10.53.0.53 and dst port 53) or (src host 10.53.0.53 and src port 53)' > t4d.txt 2>/dev/null &
sleep 1; dig +short srv.se-201-01.lab; dig +short +tcp srv.se-201-01.lab; wait
echo "TCP: $(grep -c Flags t4d.txt)  UDP: $(grep -v Flags t4d.txt | grep -c IP)"

timeout 8 tcpdump -ni eth0 -ttt 'host 10.53.0.80 and (udp portrange 5398-5399 or icmp)' > t4i.txt 2>/dev/null &
sleep 1
dig @10.53.0.80 -p 5399 +tries=3 +time=1 srv.se-201-01.lab | grep -E '^;; (comm|no )'
dig @10.53.0.80 -p 5398 +tries=3 +time=1 srv.se-201-01.lab | grep -E '^;; (comm|no )'
wait; cat t4i.txt
[1] 280
10.53.0.80
10.53.0.80
[1]+  Exit 124                timeout 5 tcpdump -ni eth0 '(dst host 10.53.0.53 and dst port 53) or (src host 10.53.0.53 and src port 53)' > t4d.txt 2> /dev/null
TCP: 10  UDP: 2
[1] 293
;; communications error to 10.53.0.80#5399: connection refused
;; communications error to 10.53.0.80#5399: connection refused
;; communications error to 10.53.0.80#5399: connection refused
;; no servers could be reached
;; communications error to 10.53.0.80#5398: timed out
;; communications error to 10.53.0.80#5398: timed out
;; communications error to 10.53.0.80#5398: timed out
;; no servers could be reached
[1]+  Exit 124                timeout 8 tcpdump -ni eth0 -ttt 'host 10.53.0.80 and (udp portrange 5398-5399 or icmp)' > t4i.txt 2> /dev/null
 00:00:00.000000 IP 10.53.1.1.54365 > 10.53.0.80.5399: UDP, length 58
 00:00:00.000059 IP 10.53.0.80 > 10.53.1.1: ICMP 10.53.0.80 udp port 5399 unreachable, length 94
 00:00:00.025220 IP 10.53.1.1.36689 > 10.53.0.80.5399: UDP, length 58
 00:00:00.000065 IP 10.53.0.80 > 10.53.1.1: ICMP 10.53.0.80 udp port 5399 unreachable, length 94
 00:00:00.025240 IP 10.53.1.1.44830 > 10.53.0.80.5399: UDP, length 58
 00:00:00.000062 IP 10.53.0.80 > 10.53.1.1: ICMP 10.53.0.80 udp port 5399 unreachable, length 94
 00:00:00.046924 IP 10.53.1.1.41612 > 10.53.0.80.5398: UDP, length 58
 00:00:01.001194 IP 10.53.1.1.53764 > 10.53.0.80.5398: UDP, length 58
 00:00:01.000636 IP 10.53.1.1.42992 > 10.53.0.80.5398: UDP, length 58
```

| Показник | TCP | UDP |
|---|---|---|
| Кількість повідомлень, надісланих сервером | 3 | 3 |
| Кількість сегментів або дейтаграм з даними в дампі | 3 (три Flags [P.]) | 3 (три пакети length 7, 8, 8) |
| Кількість викликів читання з даними на клієнті | 1 (recv() calls with data: 1) | 3 (recvfrom() calls with data: 3) |
| Пакетів для одного запиту DNS (крок 4.3) | 10 | 2 |

| Порт UDP | Що відповів сервер (за дампом) | Повідомлення `dig` | Хто виконав повторні спроби |
|---|---|---|---|
| 5399 | ICMP ... udp port 5399 unreachable |	connection refused | клієнт (dig, зробив 3 спроби) |
| 5398 | нічого (пакети відкинуто) | timed out | клієнт (dig, зробив 3 спроби по 1 с) |

1. Сервер надіслав повідомлення окремими сегментами, а клієнт отримав їх одним викликом. Де повідомлення об'єдналися? Як клієнт мав би розділити їх, якби межі не були позначені символом `;`?

2. Чому та сама послідовність дій через UDP дала окремий результат для кожного повідомлення?

3. Чому запит DNS через TCP потребує стільки пакетів? Скільки з них припадає на встановлення і закриття з'єднання?

4. Чому для порту 5399 `dig` повідомляє `connection refused`, хоча з'єднання UDP не існує? Чим цей механізм відрізняється від відмови TCP у завданні 3?

## Завдання 5. Вікно і RTT

**Прогноз** (записати до вимірювання). RTT із завдання 1: ___ мс. Найбільша швидкість для вікна `<W>`: ___ Мбіт/с. Розрахунок:

Вивід кроків 5.2 і 5.3:

```text

```

<!-- snd_wnd і rtt брати з рядка з'єднання з даними. -->

| Вимірювання | Ключ `-w` | `snd_wnd`, байт | `rtt`, мс | `snd_wnd × 8 / rtt`, Мбіт/с | Виміряно `iperf3`, Мбіт/с | `rwnd_limited` |
|---|---|---|---|---|---|---|
| 5.2 | | | | | | |
| 5.3 | 2 × `<W>` | | | | | |
| 5.3 | без ключа | | | | | |

1. Чому `snd_wnd` не дорівнює значенню ключа `-w`? Чому для розрахунку слід брати саме `snd_wnd`?

2. Що означає поле `rwnd_limited` і чому його значення різко менше без ключа `-w`?

3. У вимірюванні без ключа `-w` значення `rtt` більше, ніж у завданні 1. Звідки взялася додаткова затримка?

4. Розрахуйте мінімальне вікно для швидкості 1 Гбіт/с за вашого RTT. Яке вікно знадобилося б для тієї самої швидкості за RTT 150 мс?

## Завдання 6. Втрати і перевантаження

Вивід кроку 6.1:

```text
root@se-201-01:/lab/work# cd /lab/work
tc -s qdisc show dev eth0 | grep -E 'Sent|dropped'
iperf3 -c srv.se-201-01.lab -p 5201 -C cubic -t 10 > t6a.txt 2>&1
tc -s qdisc show dev eth0 | grep -E 'Sent|dropped'
cat t6a.txt
 Sent 133326467 bytes 88645 pkt (dropped 0, overlimits 0 requeues 0) 
 Sent 1753162759 bytes 1158806 pkt (dropped 0, overlimits 0 requeues 0) 
Connecting to host srv.se-201-01.lab, port 5201
[  5] local 10.53.1.1 port 50444 connected to 10.53.0.80 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec   124 MBytes  1.04 Gbits/sec    0   8.59 MBytes       
[  5]   1.00-2.00   sec   147 MBytes  1.23 Gbits/sec    0   8.59 MBytes       
[  5]   2.00-3.00   sec   152 MBytes  1.27 Gbits/sec    0   8.59 MBytes       
[  5]   3.00-4.00   sec   152 MBytes  1.28 Gbits/sec    0   8.59 MBytes       
[  5]   4.00-5.00   sec   148 MBytes  1.24 Gbits/sec    0   8.59 MBytes       
[  5]   5.00-6.00   sec   153 MBytes  1.28 Gbits/sec    0   8.59 MBytes       
[  5]   6.00-7.00   sec   150 MBytes  1.26 Gbits/sec    0   8.59 MBytes       
[  5]   7.00-8.00   sec   150 MBytes  1.25 Gbits/sec    0   8.59 MBytes       
[  5]   8.00-9.00   sec   152 MBytes  1.27 Gbits/sec    0   8.59 MBytes       
[  5]   9.00-10.00  sec   150 MBytes  1.25 Gbits/sec    0   8.59 MBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  1.44 GBytes  1.24 Gbits/sec    0             sender
[  5]   0.00-10.03  sec  1.44 GBytes  1.24 Gbits/sec                  receiver

iperf Done.
```

Вивід кроку 6.2:

```text
root@se-201-01:/lab/work# lab-loss 0.5%
tc -s qdisc show dev eth0 | grep -E 'Sent|dropped'
iperf3 -c srv.se-201-01.lab -p 5201 -C cubic -t 10 > t6b.txt 2>&1
tc -s qdisc show dev eth0 | grep -E 'Sent|dropped'
cat t6b.txt
lab-loss 0
qdisc netem 8001: root refcnt 13 limit 1000  loss 0.5%
 Sent 1753163156 bytes 1158812 pkt (dropped 0, overlimits 0 requeues 0) 
 Sent 1753163156 bytes 1158812 pkt (dropped 0, overlimits 0 requeues 0) 
 Sent 1776589386 bytes 1174304 pkt (dropped 26, overlimits 0 requeues 0) 
Connecting to host srv.se-201-01.lab, port 5201
[  5] local 10.53.1.1 port 32988 connected to 10.53.0.80 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec  10.5 MBytes  88.0 Mbits/sec   46   99.0 KBytes       
[  5]   1.00-2.00   sec  2.12 MBytes  17.8 Mbits/sec    9   33.9 KBytes       
[  5]   2.00-3.00   sec  1.00 MBytes  8.39 Mbits/sec    6   22.6 KBytes       
[  5]   3.00-4.00   sec  1.12 MBytes  9.44 Mbits/sec    4   28.3 KBytes       
[  5]   4.00-5.00   sec  1.00 MBytes  8.39 Mbits/sec    4   29.7 KBytes       
[  5]   5.00-6.00   sec  2.00 MBytes  16.8 Mbits/sec    4   38.2 KBytes       
[  5]   6.00-7.00   sec  1.00 MBytes  8.39 Mbits/sec    4   41.0 KBytes       
[  5]   7.00-8.00   sec  1.00 MBytes  8.39 Mbits/sec    4   35.4 KBytes       
[  5]   8.00-9.00   sec  2.00 MBytes  16.8 Mbits/sec    2   43.8 KBytes       
[  5]   9.00-10.00  sec  2.00 MBytes  16.8 Mbits/sec    4   42.4 KBytes       
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  23.8 MBytes  19.9 Mbits/sec   87             sender
[  5]   0.00-10.03  sec  21.2 MBytes  17.8 Mbits/sec                  receiver

iperf Done.
qdisc netem 8001: root refcnt 13 limit 1000 
 Sent 1776589386 bytes 1174304 pkt (dropped 26, overlimits 0 requeues 0) 
```

| Показник | 6.1 (без втрат) | 6.2 (втрати `<L>`) |
|---|---|---|
| Швидкість, Мбіт/с | 1240 (1.24 Gbits/sec) | 19.9 (sender) / 17.8 (receiver) |
| `Retr` за весь час | 0 | 87 |
| Приріст `dropped` у `tc -s qdisc` | 0 (було 0, стало 0) | 26 (було 0, стало 26) |
| Найменше і найбільше `Cwnd` за секундами | 8.59 MBytes / 8.59 MBytes | 22.6 KBytes (сек 2–3) / 99.0 KBytes (сек 0–1) |

Розрахунок за формулою Матіса для вашого RTT і `<L>`:

1. У кроці 6.1 втрати не задавалися, проте `Retr` більше нуля. Що відкинуло ці сегменти? Підтвердіть рядком виводу `tc`.

2. Порівняйте `Retr` у кроці 6.2 з кількістю надісланих сегментів, помноженою на `<L>`. Кількість сегментів оцініть за обсягом переданих даних і MSS 1448 байтів.

3. Чим пояснюється розбіжність між виміряною швидкістю і оцінкою за формулою Матіса?

4. Як змінилася б швидкість у кроці 6.2, якби RTT був удвічі більшим? Відповідь обґрунтуйте формулою, а не вимірюванням.

## Висновки

**Що виявилося несподіваним.** Результат, який не збігся з очікуванням, і рядок виводу, де це видно:

**Яке питання залишилося без відповіді.** Що саме не вдалося пояснити; до якого рядка виводу це стосується; що вже перевірено і що з'ясувати не вдалося:

## Декларація використання ШІ

Використовувалися інструменти ШІ: так

<!-- Якщо так, заповнити таблицю: один рядок на кожне звернення, що вплинуло на звіт. Промпт наводити дослівно. -->

| № | Інструмент | Промпт (дослівно) | Що використано у звіті і що змінено самостійно |
|---|---|---|---|
| 1 | ChatGPT | Поясни, як виконувати цю лабораторну роботу, та покроково напиши, як ми її робимо по завданнях. Docker на Debian Linux, репозиторій https://github.com/sergeyscherba/opism-lab-TCP. Спочатку заповнюємо перші таблиці та параметри варіанта, докер уже запущений. Додатково пиши вимоги й текст з методички та теорію, але не заповнюй таблиці за мене. я буду заповнювати їх самостійно. (прикріплено методичку, бланк звіту, політику ШІ та минулий звіт ЛР 1 для прикладу) | Взяв звідси всю теорію, команди під свій варіант se-201-01 і пояснення до завдань. Всі команди запускав сам у контейнері, виводи вставив без змін, таблиці й відповіді заповнював сам. |
