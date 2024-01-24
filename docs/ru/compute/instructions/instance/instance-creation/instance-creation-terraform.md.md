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


#### См. также

