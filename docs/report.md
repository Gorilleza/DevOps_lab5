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
Команда terraform apply успешно создала контейнер Nginx, доступный на порту 8000. Затем командой terraform destroy инфраструктура была удалена. (скрин положить)
