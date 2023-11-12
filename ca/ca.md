# Настройка центра сертификации

## Создаем запрос для сертификата

```bash
vim /etc/openssl/openssl.cnf

# Параметры уже прописаны, находим, меняем

dir = /etc/ca
copy_extensions = copy
policy = policy_anything

basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment

subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer
basicConstraints = CA:true
```

```bash
vim /var/lib/ssl/misc/CA.pl

$CATOP = "/etc/ca";
```

Генерим ключ для подчиненного ца

```bash
openssl genrsa -out subca.key 4096
```

![Alt text](image.png)

Генерируем запрос

```bash
openssl req -new -key subca.key -out subca.csr
```

![Alt text](image-1.png)

## Подписываем запрос

Создаем новый профиль

```bash
ipa certprofile-show caIPAserviceCert --out sub.cfg
```

Открываем sub.cfg

```text
policyset.serverCertSet.1.constraint.class_id=noConstraintImpl
policyset.serverCertSet.1.constraint.name=No Constraint
policyset.serverCertSet.1.default.class_id=userSubjectNameDefaultImpl
policyset.serverCertSet.1.default.name=Subject Name Default

keyUsageCrlSign=true
keyUsageDataEncipherment=false
keyUsageDecipherOnly=false
keyUsageDigitalSignature=true
keyUsageEncipherOnly=false
keyUsageKeyAgreement=false
keyUsageKeyCertSign=true
keyUsageKeyEncipherment=false
keyUsageNonRepudiation=true

policyset.serverCertSet.15.constraint.class_id=basicConstraintsExtConstraintImpl
policyset.serverCertSet.15.constraint.name=Basic Constraint Extension Constraint
policyset.serverCertSet.15.constraint.params.basicConstraintsCritical=true
policyset.serverCertSet.15.constraint.params.basicConstraintsIsCA=true
policyset.serverCertSet.15.constraint.params.basicConstraintsMinPathLen=0
policyset.serverCertSet.15.constraint.params.basicConstraintsMaxPathLen=0
policyset.serverCertSet.15.default.class_id=basicConstraintsExtDefaultImpl
policyset.serverCertSet.15.default.name=Basic Constraints Extension Default
policyset.serverCertSet.15.default.params.basicConstraintsCritical=true
policyset.serverCertSet.15.default.params.basicConstraintsIsCA=true
policyset.serverCertSet.15.default.params.basicConstraintsPathLen=0

policyset.serverCertSet.list=1,2,3,4,5,6,8,9,10,11,15
```

Импортируем новый профиль

```bash
ipa certprofile-import SubCA --desc "123" --file sub.cfg --store=1
```

Далее во freeipa создаем http сервис subca и подписываем сертификат с использованием нашего профиля

После того, как сертификат был подписан, надо вернуть его на машину с подчиненным ca

Теперь можно создать центр сертификации

```bash
./CA.pl -newca

cat newkey.pem > /demoCA/private/cakey.pem
echo "02" > /demoCA/private/serial
```

Открываем openssl.cnf и комментим

```bash
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer
basicContraints=CA:true
```

Раскоменчиваем

```bash
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
```

Добавляем

```bash
subjectAltName=@altNames
[ altNames ]
DNS.1 = *.company.prof
```

Генерим и подписываем

```bash
./CA.pl -newreq-nodes
./CA.pl -sign
```
