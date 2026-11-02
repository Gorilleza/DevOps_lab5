# Лабораторная работа №5. Проектирование модели управления версиями ПО, конфигурациями и непрерывной поставкой

**Студент:** Ходжамухаммедов Эзиз  
**Группа:** 13  

---

## Цель работы
Изучение решений для автоматизации управления инфраструктурой и установки приложений.

---

## Задание 1. Основы управления конфигурацией инфраструктуры с Terraform

В ходе выполнения задания был установлен Terraform и настроен провайдер для Docker. Был создан конфигурационный файл `main.tf`, который автоматически разворачивает контейнер Nginx.

**Файл main.tf:**
```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0.1"
    }
  }
}

provider "docker" {
  host = "unix://${pathexpand("~/.docker/run/docker.sock")}"
}

resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = false
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "tutorial"
  ports {
    internal = 80
    external = 8000
  }
}
```
Результат выполнения:
Команда terraform apply успешно создала контейнер Nginx, доступный на порту 8000. Затем командой terraform destroy инфраструктура была удалена.
<img width="1470" height="956" alt="Снимок экрана — 2026-11-02 в 13 22 28" src="https://github.com/user-attachments/assets/a101df70-f766-41bc-a7c5-f06660b31c5a" />

<img width="1470" height="956" alt="Снимок экрана — 2026-11-02 в 13 15 12" src="https://github.com/user-attachments/assets/d4c8369e-64ba-4284-ba2a-64d779693f72" />

Задание 2. Управление конфигурацией серверов с помощью Ansible

Был изучен Ansible и его основные компоненты (инвентарь, playbook, ad-hoc команды). Был написан playbook для проверки установки пакета zsh и демонстрации работы Ansible.

Файл inventory.ini:
```
[local]
localhost ansible_connection=local
```

Файл playbook.yml:
```
- name: Install zsh and test Ansible
  hosts: local
  connection: local
  tasks:
    - name: Check if zsh is installed (macOS specific)
      shell: "which zsh"
      register: zsh_check
      ignore_errors: yes

    - name: Display zsh status
      debug:
        msg: "zsh already installed at {{ zsh_check.stdout }}"
      when: zsh_check.rc == 0

    - name: Create a test file to prove Ansible works
      file:
        path: /tmp/ansible_test.txt
        state: touch

    - name: Write text to the file
      copy:
        content: "Ansible playbook executed successfully!\n"
        dest: /tmp/ansible_test.txt
```
Результат выполнения:
Команда `ansible-playbook` успешно выполнила задачи (ok=5, changed=3, failed=0), подтвердив наличие zsh и создав файл /tmp/ansible_test.txt.

<img width="1147" height="403" alt="Снимок экрана — 2026-11-02 в 13 16 49" src="https://github.com/user-attachments/assets/f096d444-5a81-4276-a70d-eb9e739b45d6" />

Ответы на контрольные вопросы

1. Для каких задач применяется Ansible?
Ansible применяется для автоматизации управления конфигурациями, развертывания приложений, оркестрации задач (например, обновление серверов), а также для непрерывной доставки (CI/CD).

2. Какие способы управления серверами применяются в Ansible?
Основные способы: Ad-hoc команды (быстрое выполнение одной команды на группе хостов) и Playbooks (сложные сценарии, описанные в YAML).

3. Что входит в состав файла Ansible playbook?
Playbook состоит из списка "плей" (plays). Каждый play содержит: hosts (группа хостов), become (повышение привилегий), tasks (список задач, где каждая задача вызывает модуль Ansible).

4. Что такое группы узлов (хостов) в Ansible?
Это логическое объединение серверов в файле инвентаризации (inventory) для применения к ним одинаковых действий (например, `[web_servers]`, `[db_servers]`).

5. Как одной командой выполнить ping группы узлов (хостов)?
`ansible <gIoup_name> -m ping -i inventory.ini` (например, `ansible all -m ping`).

6. Что такое роль Ansible?
Роль — это способ структурирования playbook для повторного использования. Она включает в себя задачи, переменные, шаблоны и файлы, организованные в стандартной структуре папок.

7. Что такое файл инвентаризации Ansible и как он используется?
Это файл (обычно inventory.ini или hosts), в котором перечислены управляемые хосты и их группы. Ansible читает его, чтобы знать, к каким серверам подключаться.

8. Что такое специальная команда Ansible?
Специальная (ad-hoc) команда — это однострочная команда Ansible, которая выполняется без написания playbook (например, `ansible all -m command -a 'uptime')`.

9. Как перезапустить службу с помощью Ansible?
Использовать модуль service:

```
- name: Restart nginx
  service:
    name: nginx
    state: restarted
```

10. Что такое Hashicorp Terraform?
Terraform — это инструмент Infrastructure as Code (IaC) от HashiCorp. Он позволяет описывать инфраструктуру (серверы, сети, базы данных) в виде кода (HCL) и автоматически разворачивать её в облачных или локальных провайдерах.

11. Что такое инфраструктура как код (Infrastructure as code — IaC)?
IaC — это подход, при котором инфраструктура (серверы, сети, балансировщики) описывается в текстовых файлах (коде) и управляется так же, как обычное программное обеспечение (с версионированием, тестами и автоматизацией).

12. Как установить Terraform?
Скачать бинарный файл с официального сайта terraform.io (или установить через пакетный менеджер `brew install terraform` / `choco install terraform`).

13. Что такое провайдер (поставщик) в Terraform?
Провайдер — это плагин, который позволяет Terraform взаимодействовать с конкретным API (например, AWS, Google Cloud, Docker, VirtualBox). Он определяет, какие ресурсы можно создавать.

14. Объясните базовую структуру файла конфигурации Terraform.
Файл `.tf` обычно содержит:

Блок `terraform` (настройки версий и провайдеров).
Блок `provider` (настройки подключения к облаку).
Блоки `resource` (описание создаваемых объектов).

15. Что такое файл состояния Terraform?
Файл `terraform.tfstate` — это JSON-файл, в котором Terraform хранит информацию о текущем состоянии инфраструктуры (какие ресурсы были созданы и их свойства). Он нужен для сопоставления реального мира с кодом.

16. Для чего нужна terraform destroy команда?
Она удаляет всю инфраструктуру, которая была создана с помощью terraform apply, и очищает файл состояния.

17. Что такое состояние в terraform.
Состояние (State) — это снимок инфраструктуры, которой управляет Terraform. Он хранится в файле terraform.tfstate и позволяет Terraform понимать, какие изменения нужно внести, чтобы привести реальную инфраструктуру в соответствие с кодом.

18. Как определить зависимости в Terraform?
Зависимости определяются неявно (если один ресурс использует атрибуты другого) или явно с помощью блока `depends_on`:

```
resource "docker_container" "nginx" {
  # ...
  depends_on = [docker_image.nginx]
}
```
