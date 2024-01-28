С помощью **Облачных серверов** можно создавать виртуальные машины через личный кабинет, OpenStack CLI, Terraform и REST API. 

## Подготовительные шаги

1. [Зарегистрируйтесь](https://auth.ps.kz/register) на платформе PS Cloud.
2. [Закажите услугу облачных серверов](https://www.ps.kz/hosting/vpc) 

Баланс счета должен быть положительным, а квот должно быть достаточно для создания желаемой конфигурации виртуальной машины.

## Создайте ВМ

1. [Перейдите](https://console.ps.kz/) в консоль управления PS Cloud и выберите услугу **Облачные серверы**.
2. Выберите название проекта, который был зарегистрирован и выберите пункт **Серверы**.
3. Нажмите кнопку **Создать**.
4. Задайте необходимые параметры ВМ:
    - **Название виртуальной машины**.
    - **Образ**: выберите из списка публичный образ операционной системы или [собственный образ](). Для создания виртуальной машины также может быть использован ранее созданный [диск]() или [снимок диска]().
    - Использование [конфигурационного диска]().
    - **Тип виртуальной машины**: выберите предустановленную конфигурацию ВМ. Подробнее прочитать про конфигурации можно [тут]().
    - Выберите [тип диска](), измените размер или добавьте дополнительные диски.
    - Подключите виртуальную машину к существующей сети или [создайте новую]()Подключить виртуальную машину можно к ранее созданному [порту]() в сети.
    - Укажите использование [плавающего IP](), если необходим доступ к виртуальной машине по публичному IP-адресу.
    - Укажите предустановленые настройки [групп безопасности]() в разделе **Firewall**.
    - Выберите или добавьте **SSH-ключ**. Можно также выбрать **Пароль**, он будет отправлен на вашу почту после установки ОС.
5. Если необходимо, вы можете добавить сценарий настройки виртуальной машины в пункте **Установка приложения**. Подробнее про сценарии настройки [cloud-init]().
6. Нажмите кнопку **Создать сервер**.
7. Дождитесь, когда процесс создания виртуальной машины завершится. Статус ВМ изменится с `CREATING` на `ACTIVE`.

---
## CLI

## Подготовительные шаги

1. Выполните [установку]() и пройдите [аутентификацию]() в проекте.

## Создайте ВМ с помощью CLI

1. Получите список доступных образов ОС и сохраните ID образа:
   
```shell
openstack image list
```

2. Получите список доступных [типов конфигураций]() ВМ и сохраните нужный ID:

```shell
openstack flavor list
```

3. Получите список доступных сетей и сохраните ID нужной сети:
   
```shell
openstack network list
```
-  Если выбрана сеть ext-net, то виртуальной машине будет автоматически назначен внешний IP-адрес.
- Если выбрана приватная сеть, то виртуальной машине после создания можно [назначить плавающий IP-адрес]().
  
4. Получите список доступных [групп безопасности]() и сохраните ID:
   
```shell
openstack security group list
```

5. Получите список доступных ключевых пар и сохраните `keypair_name`:
   
```shell
openstack keypair list
```

- Чтобы создать новую ключевую пару:

1. Сгенерируйте ключ:

```
ssh-keygen -q -N ""
```

2. Загрузите ключ:
		
```shell
openstack keypair create --public-key ~/.ssh/id_rsa.pub --type ssh <keypair_name>
```

6. Создайте загрузочный диск: 
   
```shell
openstack volume create root-volume --size 10 --image <image_id> --bootable
```

7. Создайте ВМ:
   
```shell
openstack server create <VM_name> \ 
   --volume <volume_ID> \
   --network <network_ID> \ 
   --flavor <flavor_ID> \ 
   --key-name <keypair_name> \
```

8. Проверьте состояние созданной ВМ:
```shell
openstack server list
```
	

Созданная машина должна появиться в списке доступных ВМ и иметь статус `ACTIVE`.

---
## API

1. Создайте JSON файл auth.json с телом запроса для аутентификации:

```json
{
  "auth":{
    "identity":{
      "methods":[
        "password"
      ],
      "password":{
        "user":{
          "name":"<ID-проекта>",
          "domain":{
            "name":"Default"
          },
          "password":"<Пароль>"
        }
      }
    },
    "scope":{
      "project":{
        "name":"<Имя проекта>",
        "domain":{
          "name":"Default"
        }
      }
    }
  }
}
```
Где:
<ID-проекта> - можете посмотреть в обзоре проекта.
<Пароль> - пароль для аутентификации в проект.
<Имя проекта> - название облачного проекта.

2. Получите токен аутентификации:

```shell
curl -i \

-H "Content-Type: application/json" \

-d '@auth.json' \

https://auth.pscloud.io/v3/auth/tokens | grep -i 'x-subject-token' | awk '{print $2}'
```

Токен будет содержаться в заголовке ответа в значении x-subject-token.

3. Получите список доступных типов конфигураций:

```
curl -X GET https://compute.kz-ala-1.pscloud.io/v2.1/flavors \
-H "Content-Type: application/json" \
-H "X-Auth-Token: <токен>"
```

4. Получите список доступных образов ОС и выберите нужный:

```
curl -X GET https://compute.kz-ala-1.pscloud.io/v2.1/images \
-H "Content-Type: application/json" \
-H "X-Auth-Token: <токен>"
```

5. Получите ID сети в которой необходимо создать инстанс:

```
curl -X GET https://network.kz-ala-1.pscloud.io/v2.0/networks \
-H "Content-Type: application/json" \
-H "X-Auth-Token: <токен>"
```

6. Создайте JSON файл body.json с телом запроса:

```json
{
  "server":{
    "name":"<Название ВМ>",
    "flavorRef":"<ID типа конфигурации>",
    "availability_zone":"kz-ala-1",
    "networks":[
      {
        "uuid":"ID сети в которой требуется создать ВМ"
      }
    ],
    "block_device_mapping_v2":[
      {
        "uuid":"ID образа ОС",
        "source_type":"image",
        "destination_type":"volume",
        "boot_index":0,
        "volume_size":"10"
      }
    ],
    "metadata":{
      "My Server Name":"Apache1"
    },
    "security_groups":[
      {
        "name":"default"
      }
    ]
  }
}
```

7. Передайте тело запроса с помощью POST запроса:

```
curl -X POST https://compute.kz-ala-1.pscloud.io/v2.1/servers \
-H "Content-Type: application/json" \
-H "X-Auth-Token: <токен>" \
-d '@body.json'
```

8. Проверьте статус сервера с помощью запроса:

```
curl -X GET https://compute.kz-ala-1.pscloud.io/v2.1/servers \                
-H "Content-Type: application/json" \
-H "X-Auth-Token: <токен>"
```



---
## Terraform

Если у вас еще нет Terraform, [установите его и настройте провайдер Openstack]()

1. Опишите в конфигурационном файле параметры создания локальной сети, маршрутизатора, группы безопасности и виртуальной машины с плавающим IP:

```
data "openstack_images_image_v2" "image_data" {
	most_recent = true
	properties = {
	os_distro = "almalinux"
	os_version = "90"
	}
}

data "openstack_networking_network_v2" "external_network" {
	name = "FloatingIP Net"
}

resource "openstack_networking_network_v2" "network" {
	name = "network_name"
	admin_state_up = true
}

resource "openstack_networking_subnet_v2" "subnet" {
	name = "subnet_name"
	network_id = openstack_networking_network_v2.network.id
	cidr = "192.168.10.0/24"
	enable_dhcp = true
	depends_on = [openstack_networking_network_v2.network]
}

resource "openstack_networking_router_v2" "router" {
	name = "router_name"
	admin_state_up = true
	external_network_id = data.openstack_networking_network_v2.external_network.id
}

resource "openstack_networking_router_interface_v2" "router_interface" {
	router_id = openstack_networking_router_v2.router.id
	subnet_id = openstack_networking_subnet_v2.subnet.id
	depends_on = [openstack_networking_router_v2.router]
}

resource "openstack_compute_secgroup_v2" "security_group" {
	name = "my_security_group"
	description = "open ssh and http"
	dynamic "rule" {
		for_each = ["22", "80"]
		content {
			from_port = rule.value
			to_port = rule.value
			ip_protocol = "tcp"
			cidr = "0.0.0.0/0"
		}
	}
	rule {
		from_port = -1
		to_port = -1
		ip_protocol = "icmp"
		cidr = "0.0.0.0/0"
	}
}

resource "openstack_compute_keypair_v2" "keypair" {
	name = "my-keypair"
	public_key = file("~/.ssh/public_key")
}

resource "openstack_compute_instance_v2" "instance" {
	name = "instance_name"
	flavor_name = "d1.ram1cpu1"
	security_groups = [openstack_compute_secgroup_v2.security_group.id]
	config_drive = true
	key_pair = openstack_compute_keypair_v2.keypair.name
	depends_on = [openstack_networking_subnet_v2.subnet]
	network {
		uuid = openstack_networking_network_v2.network.id
	} 
	block_device {
		source_type = "image"
		uuid = data.openstack_images_image_v2.image_data.id
		destination_type = "volume"
		volume_type = "ceph-ssd" 
		volume_size = 10
		delete_on_termination = true # false
	}
}

resource "openstack_networking_floatingip_v2" "floating_ip" {
	pool = data.openstack_networking_network_v2.external_network.name
}

resource "openstack_compute_floatingip_associate_v2" "floating_ip_association" {
	floating_ip = openstack_networking_floatingip_v2.floating_ip.address
	instance_id = openstack_compute_instance_v2.instance.id
	fixed_ip = openstack_compute_instance_v2.instance.access_ip_v4
}
```

Где:
- `openstack_networking_network_v2` - описание создания сети.
- `openstack_networking_subnet_v2` - описание создания подсети.
- `openstack_networking_router_v2` - описание создания маршрутизатора.
- `openstack_networking_router_interface_v2` - описание подключения маршрутизатора.
- `openstack_compute_secgroup_v2` - описание создания группы безопасности.
- `openstack_compute_keypair_v2` - описание создания ключевой пары.
- `openstack_compute_instance_v2` - описание создания виртуальной машины.
	- `name` - название ВМ.
	- `flavor_name` - тип конфигурации ВМ.
	- `security_groups` - группы безопасности ВМ.
	- `config_drive` - [конфигурационный диск]()
	- `key_pair` - ключевая пара.
	- `network` - сеть ВМ.
	- block_device - описание блочного устройства.
		- `volume_type` - тип диска.
		- `volume_size` - размер диска.
- `openstack_networking_floatingip_v2` - описание создания плавающего IP.
- `openstack_compute_floatingip_associate_v2` - описание подключения плавающего IP.

Если вы ранее создавали какие-либо ресурсы (облачная сеть и подсеть), вы можете не описывать их повторно. Используйте их имена и идентификаторы в соответствующих параметрах.

### Создайте ресурсы

1. Перейдите в терминале в директорию где находится созданный вами конфигурационный файл.
2. Проверьте корректность конфигурационного файла:

```
terraform validate
```

3. Выполните команду:

```
terraform plan
```
В терминале будет выведен список ресурсов с параметрами. На этом этапе изменения не будут внесены. Если в конфигурации есть ошибки, Terraform на них укажет.

4. Примените изменения конфигурации:

```
terraform apply
```
5. Подтвердите изменения: введите в терминале слово `yes` и нажмите **Enter**.
6.  Удалите ресурсы с помощью команды:

```
terraform destroy
```

---
## Смотрите также

- [Управление виртуальной машиной]().
- [Подключение к виртуальной машине по SSH]().

