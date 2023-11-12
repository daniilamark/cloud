# Настройка прокси сервера

Устанавливаем squid

```bash
apt-get install -y squid squid-helpers squid-kerberos-ldap-helper
```

В `/etc/squid/squid.conf` пишем

```text
# Порт, который будет слушаться и с какого адреса сервера
http_port 0.0.0.0:3128
# Указываем DNS-сервер, к которому будут идти обращения
dns_nameservers 10.0.10.40
# Настройка параметров аутентфикации, подсмотреть можно в squid.config.documented
auth_param basic program /usr/lib/squid/basic_ldap_auth -d -b "dc=company,dc=prof" -D "uid=admin,cn=users,cn=compat,dc=company,dc=prof" -w P@ssw0rd -f uid=%s 10.0.10.40
auth_param basic realm squid
auth_param basic children 5
external_acl_type ldapgroup %LOGIN /usr/lib/squid/ext_ldap_group_acl -b "dc=company,dc=prof" -D "uid=admin,cn=users,cn=compat,dc=company,dc=prof" -w P@ssw0rd -f (&(memberOf=cn=%g,cn=groups,cn=accounts,dc=company,dc=prof)(uid=%u)) 10.0.10.40

# Термин REQUIRED означает, что любой уже аутентифицированный пользователь будет соответствовать ACL с именем auth
# Всплывающее диалоговое окно входа в систему с запросом имени пользователя и пароля является особенностью вашего веб-браузера. Это происходит только тогда, когда у веб-браузера нет рабочих учетных данных, которые он мог бы передать Squid при запросе на вход в систему
acl auth proxy_auth REQUIRED

# ACL,в которых указываем по сути сопоставления в виде src и dst #
acl ld_group1 external ldapgroup group1
acl ld_group2 external ldapgroup group2
acl ld_group3 external ldapgroup group3
acl local_domain dstdomain .company.prof
acl monitoring dstdomain .monitoring.company.prof
acl black_list_cli1 dstdomain .vk.com .mail.yandex.ru .worldskills.org
acl cli1_net src 10.0.10.134

# Rules, тут пишем Allow и Deny правила для http access #
# Работает по принципу, что пришло первым то и отработало
# Rule 3 должно быть выше, чтобы не запрашивало авторизацию
http_access deny cli1_net black_list_cli1
http_access allow cli1_net all
# Rule 1
http_access allow auth ld_group1 local_domain
# Rule 2
http_access allow auth ld_group2 monitoring
# Deny all
http_access deny all

# После настройки необходимо перечитать конфигурацию SQUID. Если перезапускать службу squid, то это займет длительное время
# Гораздо проще воспользоваться специальной командой
squid -k reconfigure
# Если ошибок не будет, то конфиг перечитается
```
