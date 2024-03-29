## Подключение плавающего IP к ВМ

Убедитесь что OpenStack CLI [установлен]() и пройдите [аутентификацию]() в проекте.

1. [Перейдите](https://console.ps.kz/cloud/) в консоль управления PS Cloud.
2. Выберите услугу облачных серверов и перейдите в проект облачного сервера.
3. Откройте вкладку **Cерверы**.
4. Измените состояние ВМ одним из способов:
	- Через контекстное меню — для одной ВМ:
		1. В списке виртуальных машин найдите ВМ, состояние которой необходимо изменить.
		2. Откройте контекстное меню справа от названия ВМ.
		3. Выберите действие **Привязать плавающий IP**.
		4. Выберите плавающий IP из списка или выберите новый.
		5. Нажмите кнопку **Подтвердить**.

	- На странице виртуальной машины:
		1. В списке виртуальных машин нажмите на название ВМ, состояние которой необходимо изменить.
		2. Нажмите кнопку **Управление**.
		3. Выберите действие **Привязать плавающий IP**.
		4. Выберите плавающий IP из списка или выберите новый.
		5. Нажмите кнопку **Подтвердить**.
5. После подключения плавающего IP виртуальная машина будет доступна по публичному IP-адресу.

Отключить плавающий IP вы можете с помощью действия **Отвязать плавающий IP**.

---
## CLI
##### Подключение плавающего IP

1. Скопируйте ID сети FloatingIP Net:

```shell
openstack network list
```

2. Создайте плавающий IP в сети FloatingIP Net:

```shell
openstack floating ip create <ID сети FloatingIP Net>
```

3. Скопируйте IP-адрес:

```shell
openstack floating ip list
```

4. Подключите плавающий IP адрес к ВМ:

```shell
openstack server add floating ip <ID виртуальной машины> <IP-адрес>
```

Для отключения плавающего IP введите команду:

```shell
openstack server remove floating ip <ID виртуальной машины> <IP-адрес>
```


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

- <ID-проекта> - можете посмотреть в обзоре проекта.
- <Пароль> - пароль для аутентификации в проект.
- <Имя проекта> - название облачного проекта.

2. Получите токен аутентификации:

```shell
curl -i -H "Content-Type: application/json" \

-d '@auth.json' \

https://auth.pscloud.io/v3/auth/tokens | grep -i 'x-subject-token' | awk '{print $2}'
```

Токен будет содержаться в заголовке ответа в значении x-subject-token.

3. Получите ID сети с именем Floating IP Net:

```shell
curl -X GET https://network.kz-ala-1.pscloud.io/v2.0/networks \                     
-H "Content-Type: application/json" \
-H "X-Auth-Token: <token>"
```

4. Выполните запрос на создание плавающего IP:

```shell
curl -X POST https://network.kz-ala-1.pscloud.io/v2.0/floatingips \
-H "Content-Type: application/json" \
-H "X-Auth-Token: <token>" \
-d '{"floatingip": {"floating_network_id": "ID сети Floation IP Net"}}' \
```

5. Получите ID виртуальной машины, для которой необходимо подключить плавающий IP:

```shell
curl -X GET https://compute.kz-ala-1.pscloud.io/v2.1/servers \
-H "Content-Type: application/json" \
-H "X-Auth-Token: <токен>"
```

6. Подключите плавающий IP:
   
```shell
curl -X POST https://compute.kz-ala-1.pscloud.io/v2.1/servers/<ID виртуальной машины>/action \
-H "Content-Type: application/json" \
-H "X-Auth-Token: <токен>" \
-d '{"addFloatingIp": {"address": "<IP-адрес>"}}'

```

Для отключения плавающего IP от ВМ воспользуйтесь запросом:

```shell
curl -X POST https://compute.kz-ala-1.pscloud.io/v2.1/servers/<ID виртуальной машины>/action \
-H "Content-Type: application/json" \
-H "X-Auth-Token: <токен>" \
-d '{"removeFloatingIp": {"address": "<IP-адрес>"}}'

```

Для освобождения плавающего IP необходимо получить его ID:

```shell
curl -X GET https://network.kz-ala-1.pscloud.io/v2.0/floatingips \
-H "Content-Type: application/json" \
-H "X-Auth-Token: <токен>"
```

Освободить командой:

```shell
curl -X DELETE -H "X-Auth-Token: <токен>" https://network.kz-ala-1.pscloud.io/v2.0/floatingips/<ID плавающего IP>
```


---

## Terraform

```hcl

data "openstack_networking_network_v2" "external_network" {
	name = "FloatingIP Net"
}

resource "openstack_networking_floatingip_v2" "fip" {
	pool = data.openstack_networking_network_v2.external_network.name
}

resource "openstack_compute_floatingip_associate_v2" "fip_association" {
	floating_ip = openstack_networking_floatingip_v2.fip.address
	instance_id = openstack_compute_instance_v2.instance.id
	fixed_ip = openstack_compute_instance_v2.instance.access_ip_v4
}
```

Где:

- `openstack_networking_floatingip_v2` - описание создания плавающего IP.
- `openstack_compute_floatingip_associate_v2` - описание подключения плавающего IP.
