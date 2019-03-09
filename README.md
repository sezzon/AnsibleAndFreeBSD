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
Установка проходит стандартно, описывать не буду, опишу нюансы.

В процессе установки необходимо создать пользователя 
например user1 и добавить его в группу wheel на стадии

Login group is user1. Invite user1 into other groups? []:

дописать wheel

Login group is user1. Invite user1 into other groups? []:wheel

если этого не сделать, то FreeBSD не пустит вас по SSH,
не даст пользоваться командой su и т.д.

Ansible использует для работы sudo и python,
поэтому SUDO и PYTHON необходимо установить сразу

pkg install sudo

pkg install python27

И добавить группу wheel в судоерсы

echo '%wheel ALL=(ALL) ALL' >/usr/local/etc/sudoers.d/allow-wheel-user-login

В файле /etc/ssh/sshd_config раскомментарить строку

PubkeyAuthentication yes

и перезапустить sshd

service sshd restart

### 3. Установка Ansible в Ubuntu
sudo apt-get install software-properties-common

sudo apt-add-repository --yes --update ppa:ansible/ansible

sudo apt-get install ansible -y

Подготовим Ansible.

Добавим в конец файла /etc/ansible/hosts

[freebsd]

192.168.250.21 ansible_ssh_user=user1

[freebsd:vars]

ansible_python_interpreter=/usr/local/bin/python2.7

Генерим ключи SSH и передаем на FreeBSD

ssh-keygen

Передадим ключ

ssh-copy-id user1@192.168.250.21

Проверяем виден ли наш удаленный хост (freebsd)

ansible all -m ping

ansible -u user1 -m raw -a 'uptime' freebsd


### 4. Настрой FreeBSD как офисный фшлюз с потдержкой VPN (ipsec + racoon)
Далее настройка будет осуществляться средствами Ansible 
файл gateway.yml

