Образ — это файл, который содержит операционную систему и все необходимые компоненты для создания и запуска виртуальной машины.



## CLI
## API

## Terraform
---
```
resource "openstack_images_image_v2" "fedora_37_image" {
	name = "Fedora-Cloud-Base-37-1.7"
	image_source_url = "https://mirror.ps.kz/fedora/linux/releases/37/Cloud/x86_64/images/Fedora-Cloud-Base-37-1.7.x86_64.qcow2"
	web_download = true
	container_format = "bare"
	disk_format = "qcow2"
	region = "kz-ala-1"
	visibility = "private"

	properties = {
		os_distro = "Fedora"
		os_version = "37"
		os_require_quiesce = "true"
		os_type = "linux"
		hw_qemu_guest_agent = "yes"
		hw_scsi_model = "virtio-scsi"
		hw_disk_bus = "scsi"

	}
}
```

Где:

- `name` - имя образа
- `image_source_url` - ссылка на исходный образ. По умолчанию образ скачивается на локальный клиент, после чего импортируется в Glance. Несовместим с аргументом `local_file_path`.
- `web_download` - по умолчанию false. Если true, позволяет скачивать образ напрямую в OpenStack. 

Образ также может быть загружен с локального клиента с помощью аргумента `local_file_path` и указанием пути до образа. Несовместим с `image_source_url` и `web_download`.

- `container_format` - обязательный параметр. Формат контейнера определяет, находится ли образ виртуальной машины в формате файла, который также содержит метаданные о реальной виртуальной машине. Обратите внимание, что строка формата контейнера в настоящее время не используется Glance или другими компонентами OpenStack, поэтому если вы не уверены, можно просто указать в качестве формата контейнера bare.
- `disk_format ` - обязательный параметр. Формат диска образа виртуальной машины — это формат базового образа диска. Поставщики виртуальных устройств используют разные форматы размещения информации, содержащейся в образе диска виртуальной машины.
- `region` - регион в котором необходимо создать ресурс.
- `visibility` - видимость образа. Может быть "public" или "private". 
- `properties` - свойства образа. Подробнее про доступные [свойства]()

Если скачиваемый образ сжат с помощью алгоритмов gzip, bzip2 или xz. Необходим аргумент `decompress`, который перед загрузкой в OpenStack выполнит распаковку. 

[Дополнительная информация по используемым аргументам для провайдера OpenStack в Terraform](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs/resources/images_image_v2)

## Packer

