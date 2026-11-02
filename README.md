# Лабораторна робота. DNS

| Поле | Значення |
|---|---|
| ПІБ | Кириленко Олександра Вікторівна|
| Група | Іпз 2.01|
| Ідентифікатор (`sNN`) | se-201-10|
| Дата виконання | 01.10.2026|
| Середовище | GitHub Codespaces |
| Версія `dig` | DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu |

## Параметри варіанта
**Доступ до терміналу:** http://<адреса стенду>:7001/  логін `se-201-10`
**Зона:** `se-201-10.lab`  **Авторитативний сервер:** `10.53.1.1`
**Резолвери:** `10.53.0.53` (resolver-a), `10.53.0.54` (resolver-b)

## Параметри зони (вже записані у стартовий файл)

| Параметр | Значення |
|---|---|
| `$TTL` (за замовчуванням для записів) | 45 с |
| TTL самого запису SOA | 15 с |
| Поле `minimum` у SOA | 60 с |

## Записи, які потрібно додати (завдання 3)

| Ім'я | Тип | Значення | TTL |
|---|---|---|---|
| `www` | A | `10.53.0.80` | за замовчуванням |
| `blog` | CNAME | `www` | за замовчуванням |
| `mail` | A | `10.53.0.80` | за замовчуванням |
| `@` | MX | пріоритет 20, `mail` | за замовчуванням |
| `@` | TXT | `"lab-token=afdf6d2d"` | за замовчуванням |
| `test` | A | `10.53.0.81` | **25 с** (вказати явно) |

## URI для розбору (завдання 1)

1. `http://www.se-201-10.lab/media/users?sort=72&lang=x#results`
2. `http://blog.se-201-10.lab:80/users/?sort=%D0%BA%D0%B8%D1%97%D0%B2`
3. `http://user:pass@test.se-201-10.lab/media#summary`
4. `https://www.se-201-10.lab:4443/media/../users`
5. `mailto:admin@se-201-10.lab?subject=media`

## Імена для експериментів

- Завдання 4 (TTL): запис `www.se-201-10.lab` переводиться з `10.53.0.80` на `10.53.0.81`.
- Завдання 5 (негативне кешування): ім'я `new-wqdyy.se-201-10.lab` спочатку запитується, поки його не існує, і лише потім створюється (A `10.53.0.80`).

| Параметр | Значення |
|---|---|
| `$TTL`, с | 45|
| TTL запису SOA, с | 15|
| Поле `minimum` SOA, с | 60|
| Псевдонім (CNAME на `www`) | blog (значення www)|
| Другий хост і його TTL, с | test (TTL 25 секунд)|
| Пріоритет MX |20 (з посиланням на mail) |
| Токен TXT | lab-token=afdf6d2d|
| Ім'я для завдання 5 | new-wqdyy.se-201-10.lab|

## Завдання 1. Складові URI

### 1.1. Розбір

<!-- Відсутню складову позначати «—». Порт за замовчуванням записувати з позначкою «(за замовч.)». -->

| № | scheme | userinfo | host | port | path | query | fragment |
|---|---|---|---|---|---|---|---|
| 1 | http |- | www.se-201-10.lab|80(за замовч.) |/media/users |sort=72&lang=x |results |
| 2 | http| -|blog.se-201-10.lab | 80| /users/| sort=%D0%BA%D0%B8%D1%97%D0%B2 | -|
| 3 | http| user:pass| test.se-201-10.lab|80(за замовч.) | /media| -| summary|
| 4 | https| -| www.se-201-10.lab| 4443| /media/../users|- | -|
| 5 | mailto| -| -| -| admin@se-201-10.lab|subject=media |- |

| № | Ім'я, що запитується в DNS | Заголовок `Host` | Ціль запиту (request target) |
|---|---|---|---|
| 1 | www.se-201-10.lab| www.se-201-10.lab| /media/users?sort=72&lang=x|
| 2 | blog.se-201-10.lab|blog.se-201-10.lab:80 | /users/?sort=%D0%BA%D0%B8%D1%97%D0%B2|
| 3 | test.se-201-10.lab| test.se-201-10.lab| /media|
| 4 | www.se-201-10.lab| www.se-201-10.lab:4443| /media/../users|
| 5 |- | -|-|

Декодоване значення параметра URI 2:київ

Чому URI 5 не має складової host, хоча містить символ `@`:URI 5 не має складової Host, хоча містить символ @, тому що використовується схема mailto, а не http чи https.
У цьому випадку @ є частиною адреси електронної пошти та розділяє ім'я користувача (admin ) і домен пошти (se-201-10.lab). Оскільки після mailto: немає //, складова Host у цьому URI відсутня. Тому admin@se-201-10.lab вважається частиною Path, а не Host і Userinfo

### 1.2. Перевірка (після завдання 3)

<!-- Для URI 1–3: рядки запиту (починаються з ">") і тіло відповіді сервера. -->

```
> GET /media/users?sort=72&lang=x HTTP/1.1
> Host: www.se-201-10.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Mon, 02 Nov 2026 02:04:29 GMT
< Content-Type: text/plain
< Content-Length: 138
< Connection: keep-alive
< 
server: web-a
Host: www.se-201-10.lab
request URI: /media/users?sort=72&lang=x
path: /media/users
query: sort=72&lang=x
client: 10.53.1.1

> GET /users/?sort=%D0%BA%D0%B8%D1%97%D0%B2 HTTP/1.1
> Host: blog.se-201-10.lab
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Mon, 02 Nov 2026 02:10:11 GMT
< Content-Type: text/plain
< Content-Length: 159
< Connection: keep-alive
< 
server: web-a
Host: blog.se-201-10.lab
request URI: /users/?sort=%D0%BA%D0%B8%D1%97%D0%B2
path: /users/
query: sort=%D0%BA%D0%B8%D1%97%D0%B2
client: 10.53.1.1

> GET /media HTTP/1.1
> Host: test.se-201-10.lab
> Authorization: Basic dXNlcjpwYXNz
> User-Agent: curl/8.5.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Server: nginx/1.24.0 (Ubuntu)
< Date: Mon, 02 Nov 2026 02:10:43 GMT
< Content-Type: text/plain
< Content-Length: 98
< Connection: keep-alive
< 
server: web-b
Host: test.se-201-10.lab
request URI: /media
path: /media
query: 
client: 10.53.1.1

```

Значення заголовка `Authorization` і результат декодування:

```
> Authorization: Basic dXNlcjpwYXNz
root@ns-se-201-10:/# echo 'dXNlcjpwYXNz' | base64 -d
user:passroot@ns-se-201-10:/#

```

URI 4: етап, на якому запит не вдається (пошук імені / TCP / TLS), і рядки виводу, що це підтверджують:

```
root@ns-se-201-10:/# curl -v 'https://www.se-201-10.lab:4443/media/../users'
* Host www.se-201-10.lab:4443 was resolved.
* IPv6: (none)
* IPv4: 10.53.0.80
*   Trying 10.53.0.80:4443...
* connect to 10.53.0.80 port 4443 from 10.53.1.1 port 42494 failed: Connection refused
* Failed to connect to www.se-201-10.lab port 4443 after 1 ms: Couldn't connect to server
* Closing connection
curl: (7) Failed to connect to www.se-201-10.lab port 4443 after 1 ms: Couldn't connect to server


```

| Запит | `request URI`, отриманий сервером | `path`, отриманий сервером |
|---|---|---|
| звичайний | /users| /users|
| `--path-as-is` | /media/../users| /users|

Розбіжності між URI та тим, що отримав сервер, і їх пояснення:
У URI 1 і URI 2 сервер отримав адресу та параметри запиту. У URI 2 українські букви передаються у закодованому вигляді. У URI 3 логін і пароль передаються в заголовку Authorization. У URI 4 назва сайту знайшлася, але підключитися до порту 4443 не вдалося, тому запит не виконався.
Під час звичайного запиту шлях /media/../users автоматично змінився на /users, тому що curl сам прибирає зайву частину ... А коли я використала --path-as-is, сервер отримав початковий шлях /media/../users, але в результаті все одно визначив шлях як /users.

## Завдання 2. Проходження ієрархії вручну

```
root@ns-se-201-10:/# dig @10.53.0.2 ns.se-201-10.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.2 ns.se-201-10.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52934
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 7175d30b2846b4b6010000006ae7a76d3575f428c2e2adf8 (good)
;; QUESTION SECTION:
;ns.se-201-10.lab.              IN      A

;; AUTHORITY SECTION:
lab.                    172800  IN      NS      ns1.nic.lab.

;; ADDITIONAL SECTION:
ns1.nic.lab.            172800  IN      A       10.53.0.3

;; Query time: 79 msec
;; SERVER: 10.53.0.2#53(10.53.0.2) (UDP)
;; WHEN: Sun Nov 01 20:56:13 UTC 2026
;; MSG SIZE  rcvd: 117

root@ns-se-201-10:/# dig @10.53.0.3 ns.se-201-10.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.3 ns.se-201-10.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 4471
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 92d5c9c7d645137d010000006ae7a90640f2e3ab2a556672 (good)
;; QUESTION SECTION:
;ns.se-201-10.lab.              IN      A

;; AUTHORITY SECTION:
se-201-10.lab.          3600    IN      NS      ns.se-201-10.lab.

;; ADDITIONAL SECTION:
ns.se-201-10.lab.       3600    IN      A       10.53.1.1

;; Query time: 80 msec
;; SERVER: 10.53.0.3#53(10.53.0.3) (UDP)
;; WHEN: Sun Nov 01 21:03:02 UTC 2026
;; MSG SIZE  rcvd: 103

root@ns-se-201-10:/# dig @10.53.1.1 ns.se-201-10.lab A +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 ns.se-201-10.lab A +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 63075
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 50d9f8f6693719aa010000006ae7a9326fe0291a85834e45 (good)
;; QUESTION SECTION:
;ns.se-201-10.lab.              IN      A

;; ANSWER SECTION:
ns.se-201-10.lab.       45      IN      A       10.53.1.1

;; AUTHORITY SECTION:
se-201-10.lab.          45      IN      NS      ns.se-201-10.lab.

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Sun Nov 01 21:03:46 UTC 2026
;; MSG SIZE  rcvd: 103
```

| Сервер | status | flags | ANSWER | AUTHORITY | ADDITIONAL |
|---|---|---|---|---|---|
| 10.53.0.2 |NOERROR | qr| 0| 1(lab. 172800 IN NS ns1.nic.lab.)| 2(ns1.nic.lab. 172800 IN A 10.53.0.3)|
| 10.53.0.3 |NOERROR | qr| 0|1(se-201-10.lab. 3600 IN NS ns.se-201-10.lab.)| 2(ns.se-201-10.lab. 3600 IN A 10.53.1.1)|
| 10.53.1.1 | NOERROR|qr aa | 1(ns.se-201-10.lab. 45 IN  A 10.53.1.1) | 1(se-201-10.lab. 45 IN NS ns.se-201-10.lab.) | 1|

Вивід `dig ns.sNN.lab A +trace`:

```
root@ns-se-201-10:/# dig ns.se-201-10.lab A +trace

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> ns.se-201-10.lab A +trace
;; global options: +cmd
.                       78001   IN      NS      a.root-servers.net.
;; Received 59 bytes from 10.53.0.53#53(10.53.0.53) in 0 ms

lab.                    172800  IN      NS      ns1.nic.lab.
;; Received 117 bytes from 10.53.0.2#53(a.root-servers.net) in 80 ms

se-201-10.lab.          3600    IN      NS      ns.se-201-10.lab.
;; Received 103 bytes from 10.53.0.3#53(ns1.nic.lab) in 39 ms

ns.se-201-10.lab.       45      IN      A       10.53.1.1
se-201-10.lab.          45      IN      NS      ns.se-201-10.lab.
;; Received 103 bytes from 10.53.1.1#53(ns.se-201-10.lab) in 0 ms

```

| Запит | TTL запису NS | Прапорець `aa` | Секція, у якій міститься NS |
|---|---|---|---|
| `dig @10.53.0.3 sNN.lab NS +norec` |3600 | немає(тільки qr)| AUTHORITY|
| `dig @10.53.1.1 sNN.lab NS +norec` | 45 | qr aa| ANSWER|

1. Які відповіді є відсиланнями і за якими ознаками це визначено?
Відсиланнями є відповіді від кореневого сервера ( 10.53.0.2)та сервера TLD (10.53.0.3), це зрозуміло бо: у них секція ANSWER пуста (0 записів), корисна інформація про те, куди йти далі, записана в секції AUTHORITY, а прапорець aa відсутній
2. Для чого у відповіді TLD присутній запис у секції ADDITIONAL і що сталося б, якби його не було?
Цей запис в ADDITIONAL (IP-адреса 10.53.1.1 для ns.se-201-10.lab) якби його там не було, резолвер просто не знав би на яку IP- адресу надсилати наступний запит, і пошук би завис чи пішов по колу
3. Яка з двох копій запису NS є авторитативною і чому?
Авторитативна лише та, що на моєму власному сервері (10.53.1.1). Вона приходить одразу в секції ANSWER разом із прапорцем aa, а та що на TLD-сервері —це просто звичайне інформаційне посилання
## Завдання 3. Заповнення власної зони

Вивід `zone-reload`:

```
root@ns-se-201-10:/#  zone-reload
zone se-201-10.lab/IN: loaded serial 2026110103
OK
zone reload queued

```

<!-- Шість запитів до 10.53.1.1 з +norec, по одному на кожен рядок таблиці картки. -->

| Запис | status | Прапорець `aa` | Значення в ANSWER | TTL | Відповідає картці (так/ні) |
|---|---|---|---|---|---|
| `www` A | NOERROR| існує| 10.53.0.80| 45| так|
| псевдонім CNAME | NOERROR| існує| www.se-201-10.lab.|45 | так|
| `mail` A | NOERROR| існує|10.53.0.80 | 45|так |
| `@` MX | NOERROR| існує|20 mail.se-201-10.lab. | 45| так|
| `@` TXT | NOERROR| існує|lab-token=afdf6d2d | 45| так|
| другий хост A | NOERROR| існує| 10.53.0.81| 25| так|

Порівняння відповідей резолвера та власного сервера:

```
root@ns-se-201-10:/# dig blog.se-201-10.lab
;; communications error to 10.53.0.53#53: timed out

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> blog.se-201-10.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 34452
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;blog.se-201-10.lab.            IN      A

;; Query time: 669 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Mon Nov 02 00:04:08 UTC 2026
;; MSG SIZE  rcvd: 47

root@ns-se-201-10:/# dig @10.53.1.1 blog.se-201-10.lab +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 blog.se-201-10.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2611
;; flags: qr aa; QUERY: 1, ANSWER: 2, AUTHORITY: 1, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 74bb88e3feb5b18d010000006ae7d38488b8855c03025b76 (good)
;; QUESTION SECTION:
;blog.se-201-10.lab.            IN      A

;; ANSWER SECTION:
blog.se-201-10.lab.     45      IN      CNAME   www.se-201-10.lab.
www.se-201-10.lab.      45      IN      A       10.53.0.80

;; AUTHORITY SECTION:
se-201-10.lab.          45      IN      NS      ns.se-201-10.lab.

;; ADDITIONAL SECTION:
ns.se-201-10.lab.       45      IN      A       10.53.1.1

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Mon Nov 02 00:04:20 UTC 2026
;; MSG SIZE  rcvd: 142

root@ns-se-201-10:/# dig se-201-10.lab MX

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> se-201-10.lab MX
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 2102
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;se-201-10.lab.                 IN      MX

;; Query time: 80 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Mon Nov 02 00:04:34 UTC 2026
;; MSG SIZE  rcvd: 42

root@ns-se-201-10:/# dig @10.53.1.1 se-201-10.lab MX +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 se-201-10.lab MX +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 23213
;; flags: qr aa; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 3903536dd4ccf0de010000006ae7d39947dceb995df52807 (good)
;; QUESTION SECTION:
;se-201-10.lab.                 IN      MX

;; ANSWER SECTION:
se-201-10.lab.          45      IN      MX      20 mail.se-201-10.lab.

;; AUTHORITY SECTION:
se-201-10.lab.          45      IN      NS      ns.se-201-10.lab.

;; ADDITIONAL SECTION:
mail.se-201-10.lab.     45      IN      A       10.53.0.80
ns.se-201-10.lab.       45      IN      A       10.53.1.1

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Mon Nov 02 00:04:41 UTC 2026
;; MSG SIZE  rcvd: 140

```

| Запит | flags | ANSWER (кількість, типи) | AUTHORITY (кількість, типи) | ADDITIONAL (кількість, типи) | TTL |
|---|---|---|---|---|---|
| псевдонім через resolver-a | qr rd ra| 0| 0| 1,OPT| -|
| псевдонім через 10.53.1.1 | qr aa| 2 ( CNAME,  A)|1,NS | 2 (OPT, A)| 45|
| MX через resolver-a |qr rd ra |0 | 0|1,OPT |-|
| MX через 10.53.1.1 | qr aa|1,MX | 1, NS| 3 ( OPT, 2 A)|45 |

Пояснення відмінностей: Власний DNS-сервер 10.53.1.1 має прапорець aa, оскільки він є основним сервером, який обслуговує нашу DNS-зону.
Резолвер resolver-a має прапорці rd та ra, оскільки він може самостійно(рекурсивно) знаходити потрібні DNS-записи, звертаючись до інших серверів 
Наш сервер повертає статус NOERROR, тобто запит виконано успішно, а також надає додаткову інформацію про DNS- та поштовий сервери.
Резолвер у нашому результаті повернув SERVFAIL, що означає помилку під час обробки запиту. Це могло бути пов’язано з проблемами зв’язку з іншими DNS-серверами або перевищенням часу очікування

Повідомлення `zone-check` після тимчасового додавання `@ IN CNAME www`:

```
root@ns-se-201-10:/# zone-edit
root@ns-se-201-10:/# zone-check
dns_master_load: /lab/zone/se-201-10.lab.zone:16: se-201-10.lab: CNAME and other data
dns_master_load: /lab/zone/se-201-10.lab.zone:19: se-201-10.lab: CNAME and other data
dns_master_load: /lab/zone/se-201-10.lab.zone:19: se-201-10.lab: CNAME and other data
zone se-201-10.lab/IN: loading from master file /lab/zone/se-201-10.lab.zone failed: CNAME and other data
zone se-201-10.lab/IN: not loaded due to errors.
root@ns-se-201-10:/# zone-edit
root@ns-se-201-10:/# zone-check
zone se-201-10.lab/IN: loaded serial 2026110105
OK

```

1. Який TTL має кожен запис зони і звідки це значення взято?
Кожен запис зони за замовчуванням бере TTL із значення параметра $TTL у верхній частині файлу зони (у моєму варіанті це 45 секунд за замовчуванням в записах www, blog, mail, @ (MX) та @ (TXT)). Виняток становлять лише ті записи, для яких TTL вказано явно в індивідуальній картці варіанта (наприклад, для другого хоста test явно задано TTL 25 секунд) Для запису SOA час життя визначається окремо: TTL самого запису SOA становить 15 секунд, а поле minimum має значення 60 секунд
2. Скільки записів містить секція ANSWER у відповіді на запит псевдоніма і чому?
Містить 2 записи. Перший показує, що blog є псевдонімом для www, а другий містить IP-адресу www — 10.53.0.80. Тобто сервер показує, на який сайт посилається псевдонім, і його IP-адресу
3. Чому запис CNAME неможливо розмістити для імені `@`?
Запис CNAME не можна розмістити для імені @, тому що @ позначає головне ім'я домену. Для нього вже існують інші записи, наприклад SOA, NS, MX і TXT. А CNAME не можна поєднувати з іншими типами записів для одного імені.
## Завдання 4. TTL і видимість змін

Два запити з інтервалом 5 с:

```
root@ns-se-201-10:/# dig www.se-201-10.lab +noall +answer
www.se-201-10.lab.      45      IN      A       10.53.0.80
root@ns-se-201-10:/# dig www.se-201-10.lab +noall +answer
www.se-201-10.lab.      39      IN      A       10.53.0.80

```

**Прогноз** (записати до вимірювання). Формула часу появи нової адреси через залишок R, найбільша можлива затримка:T=t+R, максимальна затримка = TTL = 45 с

Перевірка, що сервер до `zone-reload` повертає стару адресу:

```
root@ns-se-201-10:/# dig @10.53.1.1 www.se-201-10.lab +norec +short
10.53.0.80

```

Залишок R і час (крок 4), час `zone-reload` (крок 5), перевірка на сервері (крок 6):

```
root@ns-se-201-10:/# date +%T; dig www.se-201-10.lab +noall +answer
04:21:02
www.se-201-10.lab.      45      IN      A       10.53.0.80
root@ns-se-201-10:/# zone-reload
zone se-201-10.lab/IN: loaded serial 2026110107
OK
zone reload queued
root@ns-se-201-10:/# date +%T
04:21:11
root@ns-se-201-10:/# dig @10.53.1.1 www.se-201-10.lab +norec +short
10.53.0.81

```

Очікуваний час появи нової адреси (підставити R у формулу): 04:21:11 + 45 = 04:21:56

Вивід циклу спостереження (перші два рядки, два останні рядки зі старою адресою, рядок з новою адресою):

```
oot@ns-se-201-10:/# while true; do echo "$(date +%T) $(dig +short www.se-201-10.lab)"; sleep 2; done
04:21:27 10.53.0.80
04:21:29 10.53.0.80
04:21:31 10.53.0.80
04:21:33 10.53.0.80
04:21:35 10.53.0.80
04:21:37 10.53.0.80
04:21:39 10.53.0.80
04:21:41 10.53.0.80
04:21:43 10.53.0.80
04:21:45 10.53.0.80
04:21:47 10.53.0.80
04:21:49 10.53.0.81
^C

```

Рядок `server:` з відповіді `curl`:

| Час зміни | Очікуваний час | Спостережений час | Виміряна затримка, с | Прогноз, с |
|---|---|---|---|---|
| 04:21:11|04:21:56|04:21:49 |38 c | 45 c|

1. Чим пояснюється розбіжність між прогнозом і виміряним значенням?
Наш цикл робить перевірку sleep 2 (тобто раз на 2 секунди). Резолвер міг оновити IP-адресу трохи раніше, але цикл зафіксував це з невеликим запізненням.   
 Поки я вручну вводила  запит zone-reload після того, як подивилася залишок R, пройшло пару секунд, тому реальний час зміни трохи змістився.
2. Якою була б найбільша можлива затримка для цього запису і за яких умов?
Найбільша затримка могла б становити 45 секунд, тобто весь час TTL. Таке було б, якби ми змінили IP одразу після того, як DNS-сервер зберіг стару адресу. Тоді він продовжував би показувати стару IP-адресу, поки не минули б усі 45 секунд
3. Як слід діяти, щоб перенести сервіс на нову адресу, якщо TTL запису становить 86 400 с?
Спочатку потрібно зменшити TTL з 86 400 наприклад до 120 секунд і виконати zone-reload. Потім почекати 24 години, щоб старі записи в кеші оновилися. Після цього змінити IP-адресу на нову та знову виконати zone-reload. Коли переконаємося, що сервіс працює за новою адресою, можна повернути TTL до попереднього значення— 86 400 секунд
## Завдання 5. Негативне кешування

**Прогноз** (записати до вимірювання). Формула за RFC 2308 і розраховане значення:Якщо клієнт запитує неіснуюче доменне ім’я, сервер повертає помилку NXDOMAIN. Щоб не повторювати такі запити, резолвер зберігає цю відповідь у кеші на певний час. Це називається негативним кешуванням.
Формула: негативного кешування = min(TTL SOA,minimum SOA)= min(15,60)=15 c

Перевірка, що сервер до `zone-reload` повертає `NXDOMAIN`:

```
root@ns-se-201-10:/# dig @10.53.1.1 new-wqdyy.se-201-10.lab +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 new-wqdyy.se-201-10.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 17305
;; flags: qr aa; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: c71a62698cca434c010000006ae81b954e4c8672dc3265e1 (good)
;; QUESTION SECTION:
;new-wqdyy.se-201-10.lab.       IN      A

;; AUTHORITY SECTION:
se-201-10.lab.          15      IN      SOA     ns.se-201-10.lab. admin.se-201-10.lab. 2026110107 3600 600 86400 60

;; Query time: 1 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Mon Nov 02 05:11:49 UTC 2026
;; MSG SIZE  rcvd: 125

root@ns-se-201-10:/# dig @10.53.1.1 new2-wqdyy.se-201-10.lab +norec

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.1.1 new2-wqdyy.se-201-10.lab +norec
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 25943
;; flags: qr aa; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 4bbd12609237e15d010000006ae81ed7e32af72ca87ca526 (good)
;; QUESTION SECTION:
;new2-wqdyy.se-201-10.lab.      IN      A

;; AUTHORITY SECTION:
se-201-10.lab.          15      IN      SOA     ns.se-201-10.lab. admin.se-201-10.lab. 2026110108 3600 600 86400 60

;; Query time: 0 msec
;; SERVER: 10.53.1.1#53(10.53.1.1) (UDP)
;; WHEN: Mon Nov 02 05:25:43 UTC 2026
;; MSG SIZE  rcvd: 126


```

Запит через resolver-a (крок 3), час `zone-reload` (крок 4), перевірка на сервері (крок 5):

```
root@ns-se-201-10:/# date +%T; dig new-wqdyy.se-201-10.lab
05:13:14

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> new-wqdyy.se-201-10.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 49044
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;new-wqdyy.se-201-10.lab.       IN      A

;; AUTHORITY SECTION:
se-201-10.lab.          15      IN      SOA     ns.se-201-10.lab. admin.se-201-10.lab. 2026110107 3600 600 86400 60

;; Query time: 404 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Mon Nov 02 05:13:15 UTC 2026
;; MSG SIZE  rcvd: 97

root@ns-se-201-10:/# zone-reload; date +%T
zone se-201-10.lab/IN: loaded serial 2026110108
OK
zone reload queued
05:16:12
root@ns-se-201-10:/# zone-reload; date +%T
zone se-201-10.lab/IN: loaded serial 2026110108
OK
zone reload up-to-date
05:16:18

root@ns-se-201-10:/# date +%T; dig new2-wqdyy.se-201-10.lab
05:25:58

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> new2-wqdyy.se-201-10.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 29261
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;new2-wqdyy.se-201-10.lab.      IN      A

;; AUTHORITY SECTION:
se-201-10.lab.          15      IN      SOA     ns.se-201-10.lab. admin.se-201-10.lab. 2026110108 3600 600 86400 60

;; Query time: 0 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Mon Nov 02 05:25:58 UTC 2026
;; MSG SIZE  rcvd: 98

```

Вивід циклу спостереження (перші два рядки, останній рядок з `NXDOMAIN`, перший рядок з `NOERROR`):(Повторюєм завдання через негативна відповідь
сплила раніше, ніж запущено цикл)

```
root@ns-se-201-10:/# while true; do echo "$(date +%T) $(dig new-wqdyy.se-201-10.lab +noall +comments | grep -o 'status: [A-Z]*')"; sleep 2; done
05:18:35 status: NOERROR
^C


root@ns-se-201-10:/# zone-reload; date +%T; while true; do echo "$(date +%T) $(dig new2-wqdyy.se-201-10.lab +noall +comments | grep -o 'status: [A-Z]*')"; sleep 2; done
zone se-201-10.lab/IN: loaded serial 2026110109
OK
zone reload queued
05:26:06
05:26:06 status: NXDOMAIN
05:26:08 status: NXDOMAIN
05:26:10 status: NXDOMAIN
05:26:12 status: NXDOMAIN
05:26:14 status: NOERROR
^C
```

| Негативну відповідь закешовано для new2 | Запис створено | Ім'я резолвиться через resolver-a | Виміряно, с | Прогноз, с |
|---|---|---|---|---|
| 05:25:58| 05:26:06| 05:26:14| 8 с| 15 с|

Завдання повторилося з іншим ім'ям new2-wqdyy.se-201-10.lab, оскільки під час першої спроби з іменем new-wqdyy негативна відповідь сплила раніше, ніж був запущений цикл спостереження (цикл одразу показав NOERROR)

1. Чому тривалість негативного кешування визначається записом SOA, а не записом, який запитували?
Оскільки доменне ім’я не існує, сервер не може повернути запитуваний запис із TTL. Тому він надсилає запис SOA у секції AUTHORITY. Запис SOA містить інформацію, необхідну резолверу для визначення часу зберігання негативної відповіді в кеші. Саме тому тривалість негативного кешування визначається записом SOA, а не запитуваним записом, якого не існує
2. Чи збігається TTL запису SOA в секції AUTHORITY з TTL запису SOA у файлі зони? Поясніть.
Не завжди, TTL запису SOA задається параметром $TTL або окремо для самого запису. Але для негативних відповідей NXDOMAIN резолвер за стандартом RFC 2308 бере найменше значення між TTL запису SOA та полем Minimum SOA. Тому TTL у відповіді секції AUTHORITY може відрізнятися від значення у файлі зони
3. Скрипт розгортання перевіряє ім'я сервісу до створення запису, а потім одразу після. Що він побачить і як це виправити? Що побачить: Спочатку скрипт отримає помилку NXDOMAIN, оскільки запису ще не існує. Навіть після його створення скрипт може продовжувати отримувати стару негативну відповідь через кеш резолвера, доки не завершиться TTL негативного кешування.
Як виправити: Потрібно дочекатися завершення TTL, очистити кеш резолвера або перевірити запис безпосередньо на авторитативному сервері, наприклад:dig @10.53.1.1 new-wqdyy.se-201-10.lab

## Завдання 6. Затримка пошуку імені

**Прогноз** (записати до вимірювання). До яких серверів звернеться resolver-a:

| Ім'я | Перший запит | Повторний запит |
|---|---|---|
| A: нове ім'я у власній зоні |1-5 мс | Кеш resolver-a, 0 мс|
| B: нове ім'я в зоні `lab.` |40 мc | Кеш resolver-a, 0 мс|
| C: ім'я в неіснуючому TLD |80 мc |Кеш resolver-a, 0 мс|
| D: `www.sNN.lab` після перезапуску резолвера |120-160 мс | Кеш resolver-a, 0 мс|

Вивід вимірювань:

```
root@ns-se-201-10:/# A="y$RANDOM.se-201-10.lab"; dig $A | grep 'Query time'; dig $A | grep 'Query time'
;; Query time: 1 msec
;; Query time: 0 msec
root@ns-se-201-10:/# B="x$RANDOM.lab"; dig $B | grep 'Query time'; dig $B | grep 'Query time'
;; Query time: 41 msec
;; Query time: 0 msec
root@ns-se-201-10:/# C="x$RANDOM.zz$RANDOM"; dig $C | grep 'Query time'; dig $C | grep 'Query time'
;; Query time: 80 msec
;; Query time: 0 msec
@OleksandraKyrylenk0 ➜ /workspaces/opism-lab-DNS (master) $ docker compose -f stand/compose.yaml restart resolver-a
[+] restart 0/1
 ⠴ Container dnslab-resolver-a-1 Restarting
root@ns-se-201-10:/# dig www.se-201-10.lab | grep 'Query time'; dig www.se-201-10.lab | grep 'Query time'
;; Query time: 402 msec
;; Query time: 1 msec

```

| Ім'я | Перший запит, мс | Повторний, мс | Сервери, до яких звернувся резолвер (за журналами) |
|---|---|---|---|
| A |1 | 0|Власний авторитативний сервер (10.53.1.1) |
| B |41 |0 |Кореневий сервер (10.53.0.2), TLD-сервер зони lab. (10.53.0.3) |
| C | 80| 0| Кореневий сервер (10.53.0.2)|
| D |402 | 0|Кореневий (10.53.0.2), TLD (10.53.0.3), власний авторитативний (10.53.1.1) |

Значення `time_namelookup` з `curl` після перезапуску резолвера:0.404254 с

Фрагменти журналів запитів root, TLD і власного сервера для рядка D:

```
root:
02-Nov-2026 13:32:55.952 client @0x7f7dd5293868 10.53.0.53#59143 (.): query: . IN NS -E(0)DC (10.53.0.2)
02-Nov-2026 13:32:56.032 client @0x7f7dd5293868 10.53.0.53#42885 (a.root-servers.net): query: a.root-servers.net IN A -E(0)DC (10.53.0.2)
02-Nov-2026 13:32:56.113 client @0x7f7dd5293868 10.53.0.53#17956 (www.se-201-10.lab): query: www.se-201-10.lab IN A -E(0)DC (10.53.0.2)

tld:
02-Nov-2026 13:32:56.233 client @0x75905406ac88 10.53.0.53#65242 (www.se-201-10.lab): query: www.se-201-10.lab IN A -E(0)DC (10.53.0.3)

власний сервер (querylog):
02-Nov-2026 13:32:56.274 client @0x797afc001f38 10.53.0.53#32899 (www.se-201-10.lab): query: www.se-201-10.lab IN A -E(0)DC (10.53.1.1)

```

1. Яку затримку вносить кожен із серверів root, TLD і власний сервер? Покажіть, з яких рядків таблиці це випливає.
За результатами вимірювань можна приблизно визначити таку затримку:Root-сервер близько 80 мс, це видно з рядка C, де запит до неіснуючого TLD тривав 80 мс
TLD-сервер близько 40 мс, це можна побачити з рядка B, де запит до імені в зоні lab. тривав 41 мс
Власний авторитативний сервер  приблизно 1 мс, це видно з рядка A, де перший запит тривав 1 мс
2. Чому в рядку A перший запит не проходить через root і TLD, хоча ім'я нове?
Тому що ім'я нове, але сама зона вже відома резолверу. Він уже знає адресу власного авторитативного сервера та може звернутися до нього напряму. Тому немає потреби проходити через root і TLD. Саме через це запит виконується дуже швидко — за 1 мс
3. Вебсторінка завантажує ресурси з п'яти різних доменів. Як це впливає на час першого відкриття сторінки?
Якщо вебсторінка завантажує ресурси з п'яти різних доменів, резолверу потрібно знайти IP-адреси кожного з них. Якщо цих записів ще немає в кеші, для кожного домену доведеться виконувати DNS-запит, що збільшить час першого відкриття сторінки.Якщо інформація про домени вже є в кеші, запити виконуватимуться набагато швидше. Тому перше відкриття сторінки може тривати довше, ніж наступні
## Завдання 7. Інцидент: зупинка власного сервера

**Прогноз** (записати до вимірювання). Що поверне кожен резолвер після зупинки сервера та спливання TTL і скільки часу займе відповідь:resolver-a (звичайний резолвер): Після завершення TTL (25 с) спробує оновити запис, але через недоступність авторитативного сервера отримає таймаут або помилку SERVFAIL.
- resolver-b (serve-stale, RFC 8767): Якщо сервер недоступний, поверне застарілу адресу 10.53.0.81 зі свого кешу,навіть після завершення TTL.Тривалість запитів:resolver-a — до 30 секунд через очікування відповіді, після чого помилка.
resolver-b — майже миттєво, оскільки використовує дані зі свого кешу.

Кешування запису в обох резолверах (крок 2) і час зупинки сервера (крок 3):

```
root@ns-se-201-10:/# dig @10.53.0.53 test.se-201-10.lab +noall +answer
test.se-201-10.lab.     25      IN      A       10.53.0.81
root@ns-se-201-10:/# dig @10.53.0.54 test.se-201-10.lab +noall +answer
test.se-201-10.lab.     25      IN      A       10.53.0.81
root@ns-se-201-10:/# named-stop

```

Запити із заміром часу (крок 5):

```
root@ns-se-201-10:/# sleep 35
time dig @10.53.0.53 test.se-201-10.lab +tries=1 +timeout=30
time dig @10.53.0.53 test.se-201-10.lab +tries=1 +timeout=30
time dig @10.53.0.54 test.se-201-10.lab +tries=1 +timeout=30
;; communications error to 10.53.0.53#53: timed out

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.53 test.se-201-10.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; no servers could be reached

real    0m30.018s
user    0m0.004s
sys     0m0.006s

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.53 test.se-201-10.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 8029
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;test.se-201-10.lab.            IN      A

;; Query time: 0 msec
;; SERVER: 10.53.0.53#53(10.53.0.53) (UDP)
;; WHEN: Mon Nov 02 14:19:54 UTC 2026
;; MSG SIZE  rcvd: 47


real    0m0.021s
user    0m0.004s
sys     0m0.006s

; <<>> DiG 9.18.39-0ubuntu0.24.04.8-Ubuntu <<>> @10.53.0.54 test.se-201-10.lab +tries=1 +timeout=30
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7462
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;test.se-201-10.lab.            IN      A

;; ANSWER SECTION:
test.se-201-10.lab.     30      IN      A       10.53.0.81

;; Query time: 1800 msec
;; SERVER: 10.53.0.54#53(10.53.0.54) (UDP)
;; WHEN: Mon Nov 02 14:19:56 UTC 2026
;; MSG SIZE  rcvd: 63


real    0m1.814s
user    0m0.006s
sys     0m0.005s

```

| Запит | status або тайм-аут | ANSWER, TTL | `real` |
|---|---|---|---|
| resolver-a, перший | таймаут (no servers could be reached)|відсутня| 0m30.018s|
| resolver-a, другий |SERVFAIL |0 | 0m0.021s|
| resolver-b | NOERROR| 1(test.se-201-10.lab. 30 IN A 10.53.0.81)| 0m1.814s|

Час `named-start` і вивід циклу спостереження до першої успішної відповіді (крок 6):

```
root@ns-se-201-10:/# named-start; date +%T; while true; do echo "$(date +%T) $(dig @10.53.0.53 test.se-201-10.lab +short +tries=1 +timeout=3)"; sleep 2; done
14:22:34
14:22:34 10.53.0.81
^C

```

Перевірка після завдання (`dig @10.53.1.1 sNN.lab SOA +norec +short`):

```
root@ns-se-201-10:/# dig @10.53.1.1 se-201-10.lab SOA +norec +short
ns.se-201-10.lab. admin.se-201-10.lab. 2026110109 3600 600 86400 60

```

### Постмортем

1. Що сталося:Авторитативний DNS-сервер (10.53.1.1) зупинили командою named-stop, тому він більше не міг відповідати на запити.
2. Вплив на клієнтів resolver-a і resolver-b:У resolver-a перестав працювати DNS, оскільки він не міг отримати нові дані від сервера. А resolver-b продовжив працювати, бо використовував старі дані зі свого кешу (serve-stale)
3. Хронологія з виміряним часом:Через 35 секунд після зупинки сервера resolver-a чекав 30 секунд, а потім видав помилку. Повторний запит одразу повернув SERVFAIL. Resolver-b відповів за 1,8 секунди та повернув IP-адресу 10.53.0.81. Після запуску сервера resolver-a знову почав працювати
4. Чому кеш допоміг або не допоміг:У resolver-a закінчився час зберігання запису (TTL), тому він більше не міг використовувати старі дані. А resolver-b завдяки serve-stale зміг використовувати їх навіть після завершення TTL.
5. Що слід змінити:Використовувати DNS-резолвери з підтримкою serve-stale (RFC 8767) та налаштувати резервні DNS-сервери, щоб у разі зупинки основного сервера робота не припинялася

1. Який TTL мав запис у відповіді resolver-b і навіщо такий TTL?
У відповіді resolver-b запис мав TTL = 30 секунд. Він потрібен, щоб клієнт знав, що запис має короткий час дії, а резолвер продовжував перевіряти, чи не з'явилися нові дані на авторитативному сервері
2. Порівняйте результат і тривалість двох запитів до resolver-a. Якщо вони різняться, сформулюйте гіпотезу щодо причини; якщо ні, поясніть, що робить резолвер протягом цього часу.
Перший запит тривав 30 секунд і завершився таймаутом, бо сервер не відповідав. Другий тривав лише 0,021 секунди та повернув помилку SERVFAIL. Це сталося тому, що резолвер запам'ятав попередню невдалу спробу й не став знову чекати відповіді від сервера
3. Чому після `named-start` ім'я через resolver-a зарезолвилося не одразу?
У нашому випадку ім'я зарезолвилося одразу, оскільки сервер швидко запустився, а попередня помилка вже не заважала отримати відповідь. В інших випадках може бути невелика затримка, поки резолвер знову почне надсилати запити
4. У якій ситуації serve-stale шкодить, а не допомагає? Пов'яжіть відповідь із завданням 4.
Serve-stale може зашкодити, якщо на сервері терміново змінили IP-адресу, наприклад через злам або перенесення сервісу на інший сервер (як у завданні 4). Тоді резолвер може продовжувати віддавати стару адресу, через що клієнти будуть підключатися не туди
## Фінальний файл зони

<!-- Повний вміст stand/students/sNN/zone/sNN.lab.zone після завершення роботи. -->

```
root@ns-se-201-10:/# cat /lab/zone/se-201-10.lab.zone
; Zone se-201-10.lab — starting version.
; After every change: increase the serial, then run  zone-reload
$TTL 45
@     15 IN SOA ns.se-201-10.lab. admin.se-201-10.lab. (
            2026110109 ; serial
            3600       ; refresh
            600        ; retry
            86400      ; expire
            60         ; minimum
        )
@     IN NS ns.se-201-10.lab.
ns    IN A  10.53.1.1

; ---- add records from your variant card below this line ----
www   IN A 10.53.0.81
blog    IN CNAME www
mail    IN A 10.53.0.80
@        IN MX 20 mail
@        IN TXT "lab-token=afdf6d2d"
test   25   IN A 10.53.0.81
new-wqdyy    IN    A    10.53.0.80
new2-wqdyy    IN    A    10.53.0.80

```

## Висновки

**Що виявилося несподіваним.** Результат, який не збігся з очікуванням, і рядок виводу, де це видно: несподіваним виявилося те, що під час виконання завдання 5 негативний кеш (NXDOMAIN) для першого імені (new-wqdyy) сплив настільки швидко (або через малу затримку між створенням запису і запуском циклу), що перший же запит циклу одразу повернув статус NOERROR замість очікуваного NXDOMAIN. Через це довелося повторювати експеримент із новим ім'ям (new2-wqdyy); рядок: "05:18:35 status: NOERROR";

Також (завдання 2 — проходження ієрархії DNS) несподіваним було те, що TLD-сервер (10.53.0.2) сам передав IP-адресу наступного DNS-сервера (10.53.0.3) у секції ADDITIONAL, без необхідності виконувати окремий запит для отримання цієї адреси. Це дало змогу зрозуміти, що glue records допомагають швидше знаходити потрібні DNS-сервери та зменшують кількість додаткових запитів під час резолвінгу; рядок- ;; ADDITIONAL SECTION:
ns1.nic.lab.            172800  IN      A       10.53.0.3

І завдання 7 - падіння сервера:
Несподіваною виявилася різниця двох резолверів під час падіння авторитативного сервера: звичайний резолвер (resolver-a) після завершення TTL  чекав цілих 30 секунд і видав таймаут, тоді як резолвер із serve-stale (resolver-b) відповів миттєво (за 1.8 секунди), зберігши доступність сайту завдяки застарілому кешу; рядок - real  0m30.018s (для resolver-a); real    0m1.814s   (для resolver-b із serve-stale)
 

**Яке питання залишилося без відповіді.** Що саме не вдалося пояснити: я так і не зрозуміла, від якого саме моменту потрібно правильно рахувати тривалість негативного кешування в завданні 5: від моменту отримання та кешування відповіді NXDOMAIN чи від моменту створення нового DNS-запису?

У моєму експерименті від створення запису до першої успішної відповіді NOERROR пройшло 8 секунд, тоді як від першої негативної відповіді NXDOMAIN до NOERROR — приблизно 16 секунд. Розрахунковий час за SOA становив 15 секунд;

Перевірено формулу відповідно до RFC 2308 та виконано прямі запити до авторитативного сервера, які підтвердили наявність створеного запису;

Залишилося питання: який із цих проміжків часу потрібно використовувати для оцінки фактичної тривалості негативного кешування і чи можна вважати результат експерименту таким, що відповідає прогнозу в 15 секунд?

## Декларація використання ШІ

Використовувалися інструменти ШІ: так 

| № | Інструмент | Промпт (дослівно) | Що використано у звіті і що змінено самостійно |
|---|---|---|---|
| 1 |Gemini 3.6 Flash|Як зайти у середовище Codespace ?|За допомогою ШІ уточнено порядок відкриття середовища GitHub Codespaces та способи доступу до нього. Самостійно виконано вхід у середовище та подальшу роботу в ньому|
| 2 |Gemini 3.6 Flash |Чому у деяких заголовках Host ми пишем порт а в деяких ні ?  | За допомогою ШІ уточнено, у яких випадках порт потрібно вказувати в заголовку Host, а в яких його можна опустити, оскільки він визначений за замовчуванням|
| 3 | Gemini 3.6 Flash |У чому різниця між прапорцями aa і flags: qr rd ra;|За допомогою ШІ уточнено значення прапорців у заголовку DNS-відповіді (AA, QR, RD, RA) та їхнє призначення. Самостійно проаналізовано ці прапорці у результатах виконаних DNS-запитів|
