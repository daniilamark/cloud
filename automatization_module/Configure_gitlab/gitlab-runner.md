# Configure gitlab runners, pipelines

```text
Установите два gitlab раннера на ВМ worker1 и worker2
Подключите раннеры к gitlab
Раннеры должны быть сконфигурированы в режиме docker
```

Переходим на машины worker1&worker2 и ставим пакет gitlab-runner

```bash
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | bash
```

```bash
apt install gitlab-runner
```

Конфигурим раннеры

a) Переходим: **Settings > CI / CD > Runners > Expand > Create gitlab runner**

1) Выбираем OS = Linux
2) Ставим галочку Run untagged jobs
3) Создаем раннер

b) Следуем инструкции и вводим команду:

```bash
gitlab-runner register  --url https://gitlab.company.prof  --token your_token_from_gitlab
```
1) В качестве executor пишем docker
2) В качестве image пишем alpine

После регистрации раннера следует поправить конфигурацию под наши нужды:

```bash
vim /etc/gitlab-runner/config.toml
```

Нас интересуют строки:

```text
url
executor
image
privileged
volumes
network_mode
```

```ini
concurrent = 1
check_interval = 0
shutdown_timeout = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "debian"
  url = "https://gitlab.company.prof"
  id = 2
  token = "your_token_from_gitlab"
  token_obtained_at = 2023-09-12T12:53:14Z
  token_expires_at = 0001-01-01T00:00:00Z
  executor = "docker"
  [runners.cache]
    MaxUploadedArchiveSize = 0
  [runners.docker]
    tls_verify = false
    image = "alpine:latest"
    privileged = true 
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock", "/etc/ssl/ca.crt:/etc/gitlab-runner/certs/ca.crt:ro"]
    network_mode = 'host'
    shm_size = 0
```

После правки конфига надо ребутнуть сервис раннера и отследить его доступность в интерфейсе гитлаба. Если зеленый - значит раннер приконнектился и с ним можно работать.

```(Повторить все действия выше на worker2)```
