# Инструкция по установке Pi-hole в Docker на Ubuntu Server в VirtualBox

Эта инструкция поможет вам установить и настроить Pi-hole в Docker на сервере Ubuntu, работающем в VirtualBox.

## Шаг 1: Обновление и установка Docker

1. Обновите систему и установите Docker:
    ```bash
    sudo apt update && sudo apt upgrade -y
    sudo apt install docker.io -y
    sudo systemctl enable docker
    sudo systemctl start docker
    ```

2. Установите Docker Compose:
    ```bash
    sudo apt install docker-compose -y
    ```

## Шаг 2: Создание каталога для Pi-hole

1. Создайте каталог для Pi-hole и перейдите в него:
    ```bash
    mkdir -p ~/pihole
    cd ~/pihole
    ```

2. Откройте файл `docker-compose.yml` для редактирования:
    ```bash
    nano docker-compose.yml
    ```

3. Вставьте следующий код в файл `docker-compose.yml`:
    ```yaml
    version: '3'

    services:
      pihole:
        container_name: pihole
        image: pihole/pihole:latest
        ports:
          - "53:53/tcp"    # Для изменения портов, если необходимо
          - "53:53/udp"    # Для изменения портов, если необходимо
          - "67:67/udp"    # Требуется, если используете Pi-hole как DHCP сервер
          - "80:80/tcp"
        environment:
          TZ: 'America/Chicago'
          # WEBPASSWORD: 'установите здесь надежный пароль или он будет случайным'
        volumes:
          - './etc-pihole:/etc/pihole'
          - './etc-dnsmasq.d:/etc/dnsmasq.d'
        cap_add:
          - NET_ADMIN      # Требуется, если используете Pi-hole как DHCP сервер, иначе не нужно
        restart: unless-stopped
    ```

4. Дополнительная информация:
    - [GitHub проект Pi-hole Docker](https://github.com/pi-hole/docker-pi-hole)
    - [Документация Pi-hole](https://docs.pi-hole.net/)

## Шаг 3: Установка зависимостей (по необходимости)

1. Установите Python и создайте виртуальное окружение (по необходимости):
    ```bash
    sudo apt install python3-pip
    sudo apt install python3-venv
    python3 -m venv env
    source env/bin/activate
    pip3 install setuptools
    ```

## Шаг 4: Запуск контейнера Pi-hole

1. Запустите контейнер с Pi-hole:
    ```bash
    docker-compose up -d
    ```

2. Если нужно остановить контейнер, выполните:
    ```bash
    sudo docker-compose down
    ```

## Шаг 5: Доступ к веб-интерфейсу Pi-hole

Остановите и отключите `systemd-resolved` (по необходимости):
    ```bash
    
    sudo systemctl stop systemd-resolved
    sudo systemctl disable systemd-resolved
    ```
    
1. Перейдите в веб-интерфейс Pi-hole по адресу:
    ```
    http://ubuntu_ip_address/admin/
    ```

2. Если необходимо установить или изменить пароль для веб-интерфейса:
    ```bash
    sudo docker exec pihole pihole -a -p 12345
    ```
    Замените `12345` на ваш новый пароль.

## Примечания

- Если вы используете Pi-hole как DHCP сервер, удалите порты из конфигурации и добавьте `network_mode: "host"`.
- Убедитесь, что порты 53 (TCP и UDP) и 67 (UDP) открыты на вашем сервере для корректной работы Pi-hole.
