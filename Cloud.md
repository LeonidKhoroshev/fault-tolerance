# Домашнее задание к занятию «Отказоустойчивость в облаке» - Леонид Хорошев

 ---

## Задание 1 

Возьмите за основу [решение к заданию 1 из занятия «Подъём инфраструктуры в Яндекс Облаке»](https://github.com/netology-code/sdvps-homeworks/blob/main/7-03.md#задание-1).

1. Теперь вместо одной виртуальной машины сделайте terraform playbook, который:

- создаст 2 идентичные виртуальные машины. Используйте аргумент [count](https://www.terraform.io/docs/language/meta-arguments/count.html) для создания таких ресурсов;
- создаст [таргет-группу](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/lb_target_group). Поместите в неё созданные на шаге 1 виртуальные машины;
- создаст [сетевой балансировщик нагрузки](https://registry.terraform.io/providers/yandex-cloud/yandex/latest/docs/resources/lb_network_load_balancer), который слушает на порту 80, отправляет трафик на порт 80 виртуальных машин и http healthcheck на порт 80 виртуальных машин.

Рекомендуем изучить [документацию сетевого балансировщика нагрузки](https://cloud.yandex.ru/docs/network-load-balancer/quickstart) для того, чтобы было понятно, что вы сделали.

2. Установите на созданные виртуальные машины пакет Nginx любым удобным способом и запустите Nginx веб-сервер на порту 80.

3. Перейдите в веб-консоль Yandex Cloud и убедитесь, что: 

- созданный балансировщик находится в статусе Active,
- обе виртуальные машины в целевой группе находятся в состоянии healthy.

4. Сделайте запрос на 80 порт на внешний IP-адрес балансировщика и убедитесь, что вы получаете ответ в виде дефолтной страницы Nginx.



Terraform Playbook.

```
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
  required_version = ">= 0.14"
}

provider "yandex" {
  zone = "ru-central1-a"
  token = "y0_AgAAA....................................PQxQr0K1VqSs"
  cloud_id = "b1g...............qb"
  folder_id = "b1g........0cohh2hk2"
}
resource "yandex_compute_instance" "vm" {
count = 2
name = "vm${count.index}"

resources {
  cores = 2
  memory = 2
  core_fraction = 5
}

boot_disk {
  initialize_params {
  image_id = "fd8gofoi2nd7l75a612j"
  size = 10
  }
}

network_interface {
  subnet_id = yandex_vpc_subnet.subnet-1.id
  nat = true
}

placement_policy {
  placement_group_id = "${yandex_compute_placement_group.group1.id}"
}

metadata = {
user-data = file("./metadata.yaml")
}

}

resource "yandex_compute_placement_group" "group1" {
  name = "test-pg1"
}

resource "yandex_vpc_network" "network-1" {
  name = "network1"
}

resource "yandex_vpc_subnet" "subnet-1" {
  name           = "subnet1"
  zone           = "ru-central1-a"
  network_id     = "${yandex_vpc_network.network-1.id}"
  v4_cidr_blocks = ["10.128.0.0/24"]
}

resource "yandex_lb_network_load_balancer" "lb-1" {
  name = "lb-1"
  listener {
    name = "my-lb1"
    port = 80
    external_address_spec {
      ip_version = "ipv4"
    }
  }
  attached_target_group {
    target_group_id = yandex_lb_target_group.test-1.id
    healthcheck {
      name = "http"
      http_options {
        port = 80
        path = "/"
      }
    }
  }
}

resource "yandex_lb_target_group" "test-1" {
  name           = "test-1"
  target {
    subnet_id    = yandex_vpc_subnet.subnet-1.id
    address   = yandex_compute_instance.vm[0].network_interface.0.ip_address
  }

  target {
    subnet_id    = yandex_vpc_subnet.subnet-1.id
    address   = yandex_compute_instance.vm[1].network_interface.0.ip_address
  }
}
```
Metadata

```
#cloud-config
disable_root: true
timezone: Europe/Moscow
repo_update: true
repo_upgrade: true

users:
  - name: leonid
    groups: sudo
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh-authorized-keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDEG+sEeRs/TjbFhL+HiJGnjYEuCReycMND5n6Ke3Y1EayrVrgl9MvDv/1XXxPSRAaqZSvSqKp4/vt1xeNqJunu0dnHpY89ZTI0mKyjxHnUvhj58XnhvOalJxhEhtdLuFyFqSXsux2na+Nn>ZcnLUJO2Skmw7n8= root@1392396-cz43230.tw1.ru
```
В качестве ОС виртуальных машин выбран Centos7, содержащий пакеты python и соответствующие зависимости.

Следовательно, nginx на 2 хоста устанавливаем с помощью Ansible.

Создаем файл inventory

```
[hosts]
vm1 ansible_host=51.250.92.72 ansible_connection=ssh  ansible_ssh_user=leonid
vm0 ansible_host=51.250.8.148 ansible_connection=ssh  ansible_ssh_user=leonid
```

Далее устанавливаем необходимые пакеты

```
ansible -i inventory all -m yum -a "name=epel-release" -b
ansible -i inventory all -m yum -a "name=nginx" -b
```

Скриншот статуса балансировщика и целевой группы

![Alt text](https://github.com/LeonidKhoroshev/fault-tolerance/blob/main/cloud/cloud1.1.png)

Скриншот страницы, которая открылась при запросе IP-адреса балансировщика

![Alt text](https://github.com/LeonidKhoroshev/fault-tolerance/blob/main/cloud/cloud1.2.png)


---

## Задания со звёздочкой*
Эти задания дополнительные. Выполнять их не обязательно. На зачёт это не повлияет. Вы можете их выполнить, если хотите глубже разобраться в материале.

---

## Задание 2*

1. Теперь вместо создания виртуальных машин создайте [группу виртуальных машин с балансировщиком нагрузки](https://cloud.yandex.ru/docs/compute/operations/instance-groups/create-with-balancer).

2. Nginx нужно будет поставить тоже автоматизированно. Для этого вам нужно будет подложить файл установки Nginx в user-data-ключ [метадаты](https://cloud.yandex.ru/docs/compute/concepts/vm-metadata) виртуальной машины.

- [Пример файла установки Nginx](https://github.com/nar3k/yc-public-tasks/blob/master/terraform/metadata.yaml).
- [Как подставлять файл в метадату виртуальной машины.](https://github.com/nar3k/yc-public-tasks/blob/a6c50a5e1d82f27e6d7f3897972adb872299f14a/terraform/main.tf#L38)

3. Перейдите в веб-консоль Yandex Cloud и убедитесь, что: 

- созданный балансировщик находится в статусе Active,
- обе виртуальные машины в целевой группе находятся в состоянии healthy.

4. Сделайте запрос на 80 порт на внешний IP-адрес балансировщика и убедитесь, что вы получаете ответ в виде дефолтной страницы Nginx.

*В качестве результата пришлите*

*1. Terraform Playbook.*

*2. Скриншот статуса балансировщика и целевой группы.*

*3. Скриншот страницы, которая открылась при запросе IP-адреса балансировщика.*


