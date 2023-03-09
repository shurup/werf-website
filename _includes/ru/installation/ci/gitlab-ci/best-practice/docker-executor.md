## Требования

- GitLab;

- Хост для установки GitLab Runner, имеющий:
  
  - [Docker Engine](https://docs.docker.com/engine/install/).

## Установка GitLab Runner

Установите GitLab Runner на выделенный для него хост, следуя [официальным инструкциям](https://docs.gitlab.com/runner/install/linux-repository.html).

## Регистрация GitLab Runner

Для регистрации GitLab Runner в GitLab следуйте [официальным инструкциям](https://docs.gitlab.com/runner/register/index.html), указав Docker в качестве executor'а и любой образ в качестве image (например, `alpine`).

## Настройка GitLab Runner

На хосте GitLab Runner'а откройте его конфигурационный файл `config.toml` и добавьте зарегистрированному ранее GitLab Runner'у следующие опции:

```toml
[[runners]]
  name = <имя зарегистрированного Runner'а>
  [runners.docker]
    security_opt = ["seccomp:unconfined", "apparmor:unconfined"]
    volumes = ["/home/build/.werf"]
```

Если хост GitLab Runner'а имеет версию ядра Linux 5.12 или ниже, то установите на хост `fuse` и добавьте в файл `config.toml` ещё одну опцию:

```toml
[[runners]]
  name = <имя зарегистрированного Runner'а>
  [runners.docker]
    devices = ["/dev/fuse"]
```

При желании произведите [дополнительную конфигурацию](https://docs.gitlab.com/runner/configuration/advanced-configuration.html) GitLab Runner'а.

## Настройка проекта GitLab

- Включите [требование удачно выполненного pipeline для merge requests](https://docs.gitlab.com/ee/user/project/merge_requests/merge_when_pipeline_succeeds.html#require-a-successful-pipeline-for-merge).

- Включите [возможность автоматически отменять лишние pipelines](https://docs.gitlab.com/ee/ci/pipelines/settings.html#auto-cancel-redundant-pipelines).

- [Создайте и сохраните access token](https://docs.gitlab.com/ee/user/project/settings/project_access_tokens.html#create-a-project-access-token) для очистки ненужных образов из container registry со следующей конфигурацией:
  
  - **Token name:** `werf-images-cleanup`
  
  - **Role:** `developer`
  
  - **Scopes:** `api`

- [В переменных проекта](https://docs.gitlab.com/ee/ci/variables/#for-a-project) добавьте следующие переменные:
  
  - Версия werf:
    
    - **Key:** `WERF_VERSION`
    
    - **Value:** `{{ include.version }} {{ include.channel }}`
  
  - Access token для очистки ненужных образов:
    
    - **Key:** `WERF_IMAGES_CLEANUP_PASSWORD`
    
    - **Value:** `<сохранённый "werf-images-cleanup" access token>`
    
    - **Protect variable:** `yes`
    
    - **Mask variable:** `yes`

- [Добавьте плановое задание](https://docs.gitlab.com/ee/ci/pipelines/schedules.html#add-a-pipeline-schedule) на каждую ночь для очистки ненужных образов в container registry, указав ветку `main`/`master` в качестве **Target branch**.

## Конфигурация CI/CD проекта

TODO

## TODO

- ```
  image:
      name: "registry.werf.io/werf/werf:1.2-stable"
      pull_policy: always
  before_script:
  - source "$(werf ci-env gitlab --as-file)"
  ```