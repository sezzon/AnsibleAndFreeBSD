# AnsibleAndFreeBSD
(Изучаем Ansible)


## Задачи

Подготовка FreeBSD с помощью Ansible и минимальным ручным вмешательством.

Для усложнения задачи будем скрещивать разные ОС.
Под рукой есть живой тестовый Windows server 2008 r2
с поднятой ролью Hyper-V.

И так
1. Установить Ubuntu 16.04 в качестве гостя под Hyper-V Windows server 2008 r2
2. Установить FreeBSD 12.0 в качестве гостя под Hyper-V Windows server 2008 r2
3. Установить в Ubuntu Ansible
4. Настроить FreeBSD как офисный фшлюз с потдержкой VPN (ipsec + racoon)
5. (Опционально, если придумаю что засунуть в Docker, например Apache или nginx + owncloud)



## Ну поехали

Назначим IP адреса:

192.168.250.20 - Ubuntu

192.168.250.21 - FreeBSD внешний IP

10.0.2.1       - FreeBSD локальный IP

### 1. Устновка Ubuntu
Создаем виртуальную машину в Hyper-V как обычно.
Далее заходим в настройки виртуальной машины удаляем "сетевой адаптер" и добавляем "устаревший сетевой адаптер".
Так же необходимо поставить галочку "Включить спуфинг MAC-адресов". Иначе в Ubuntu не заработает сеть.
Установку Ubuntu описывать не буду, так как все стандартно.
Надеюсь читатели умеют устанавливать Ubuntu и подключаться по SSH.

Обновляемся:

sudo apt-get update

Если необходимо

sudo apt-get upgrade

Для скрещивания с Hyper-V устанавливаем

sudo apt-get install --install-recommends linux-virtual-lts-xenial -y

sudo apt-get install --install-recommends linux-tools-virtual-lts-xenial linux-cloud-tools-virtual-lts-xenial -y

Перезагружаемся и вуаля, мы дружим с Hyper-V.

### 2. Установка FreeBSD
В отличие от Ubuntu, FreeBSD дружит с Hyper-V из коробки, начиная с 10-й версии. Поэтому заменять сетевой адаптер на старую
версию не нежно. Но "Включить спуфинг MAC-адресов" тоже необходимо.
Установка проходит стандартно, описывать не буду.

### 3. Установка Ansible в Ubuntu
sudo apt-get install software-properties-common

sudo apt-add-repository --yes --update ppa:ansible/ansible

sudo apt-get install ansible




