# Настройка дисковой подсистемы

## Зеркалируемый LVM на SRV1

Находим диски, из которых будем собирать LVM через `lsblk`

Названия дисков будут отличаться. Выглядеть диски должны как неиспользуемые.

![Alt text](image.png)

Сначала диски надо подготовить

```bash
fdisk /dev/vdb
n
enter
enter
enter
t
8e
w
```

![Alt text](image-1.png)

Второй готовим по аналогии

Далее собирем LVM

```bash
pvcreate /dev/vdb1 /dev/vdc1
vgcreate hitech_vg /dev/vdb1 /dev/vdc1
lvcreate -l 100%FREE -m1 -n mirror hitech_vg
```

![Alt text](image-2.png)

Проверить можно через `lvdisplay -v`

![Alt text](image-3.png)

Далее, нужно автоматически монтировать в `/opt/data`

Делаем файловую систему

```bash
mkfs.xfs /dev/hitech_vg/mirror
```

![Alt text](image-4.png)

Создаем каталог и добавляем запись в fstab

```bash
mkdir /opt/data
vim /etc/fstab
/dev/hitech_vg/mirror /opt/data xfs defaults 0 0
```

Проверить можно через `mount -av`

![Alt text](image-5.png)

## Шифрованный на SRV2

Производим подготовку и сборку LVM точно также как на SRV1

Надо учесть, что в альте нет `fdisk`

Вместо него юзаем `sfdisk`

```bash
sfdisk /dev/vdb
,
write
```

![Alt text](image-6.png)

Только в командочке `lvcreate` не пишем ключ `-m1`

```bash
lvcreate -i 2 -I 4 -l +100%FREE -n encrypted hitech_vg
```

Теперь шифруем примерно вот так

```bash
cryptsetup luksFormat --hash=sha512 --key-size=512 --cipher=aes-xts-plain64 --verify-passphrase /dev/hitech_vg/encrypted
```

Пароль ставим не P@ssw0rd, что-нибудь посложнее
Главное не забыть

![Alt text](image-7.png)

Проверим, что оно зашифровалось через luksOpen

```bash
cryptsetup luksOpen /dev/hitech_vg/encrypted unencrypted
```

![Alt text](image-8.png)

Том, который называется unencrypted уже можно монтировать

Теперь надо сделать так, чтобы он нормально монтировался при загрузке без ввода пароля

Для начала создаем ключик

```bash
dd if=/dev/random bs=32 count=1 of=/root/key
cryptsetup luksAddKey /dev/sdb1 /root/key
```

![Alt text](image-9.png)

Потом добавляем в `/etc/crypttab` запись вида

```text
unencrypted /dev/hitech_vg/encrypted /root/key
```

![Alt text](image-10.png)

После этого надо ребутнуться и проверить, что диск расшифровывается при загрузке

Смотрим `lsblk` -- если есть расшифрованный раздел, норм

Не забываем добавить файлуху

Добавляем в `fstab` запись с расшифрованным разделом

```text
/dev/mapper/unencrypted /opt/data xfs defaults 0 0
```
