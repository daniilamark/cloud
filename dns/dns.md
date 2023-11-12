# Настройка DNS

## Основной сервер

Ставим бинд

```bash
apt install bind9
```

Первый файл, который нужно поправить -- `/etc/bind/named.conf.options`

```text
forwarders { 77.88.8.8; };
dnssec-validation no;
listen-on { any; };
```

![Alt text](image.png)

Потом идем в `/etc/bind/named.conf.default-zones`

В самом низу описываем нашу зону

Прямую и обратную

```text
zone "company.prof" {
    type master;
    file "/etc/bind/company.prof";
};

zone "0.10.in-addr.arpa" {
    type master;
    file "/etc/bind/0.10.in-addr.arpa";
};
```

![Alt text](image-1.png)

Идем в `/etc/bind` и создаем зоны из шаблона

![Alt text](image-2.png)

Открываем для редактирования прямую зону `company.prof`

Заполняем зону

![Alt text](image-3.png)

Аналогично с обратной зоной

![Alt text](image-4.png)

Когда все сделано -- проверяемся через `named-checkconf -z`

![Alt text](image-5.png)

Если все в порядке -- `systemctl restart bind9`

![Alt text](image-6.png)

## split-brain

Для реализации нужно настроить acl и view

Заходим в `/etc/bind/named.conf.local`

Пишем 2 acl

```text
acl "left" {10.0.10.0/24; };
acl "right" {10.0.20.0/24; };
```

![Alt text](image-7.png)

Далее в файле `/etc/bind/named.conf.default-zones` создаем views

![Alt text](image-8.png)

Очень важно поместить все созды в view left, то есть закрывающая скобка ставится внизу файла, после описания всех зон

Внизу же размещается view right

Во view right размещается зона company.prof, но путь к файлу будет другим

![Alt text](image-9.png)

Открываем для редактирования файл с зоной company.prof и добавляем туда запись test

![Alt text](image-10.png)

Копируем этот файл в company.prof.right и открываем файл для редактирования

Меняем адрес записи test на адрес srv2

![Alt text](image-11.png)

Все готово, теперь с из сети left запись будет резолвиться в адрес srv1, а из сети right в адрес srv2

![Alt text](image-12.png)

![Alt text](image-13.png)

## Второй DNS сервер

На мастер сервере заходим в файл с зоной и разрешаем ее скачивать

![Alt text](image-14.png)

На SRV2 ставим bind

```bash
apt-get install bind
```

Открываем `/etc/bind/options.conf`

```text
listen-on { any; };
```

Далее открываем `/etc/bind/local.conf` и прописываем зоны

![Alt text](image-15.png)

После того, как все сделали  перезапускаем bind

```bash
systemctl restart bind
```

В логах можно посмотреть, приехал ли файл

![Alt text](image-16.png)

Файл недоступен для редактирования

![Alt text](image-17.png)

Резолв работает

![Alt text](image-18.png)
