

| Значение | Описание | Поддерживаемые значения |
| ---- | ---- | ---- |
| `os_distro` | Общее название дистрибутива операционной системы строчными буквами. | Название дистрибутива, например:<br>`centos`, `debian`, `fedora`, `arch`, `freebsd`, `gentoo`, `netbsd`, `opensuse`, `openbsd`, `ubuntu`, `rhel`, `windows` |
| `os_version` | Версия операционной системы, указанная дистрибьютором. | Корректный номер версии. Например `22.04`. |
| `os_require_quiesce` | Процесс приостановки записи на диск во время создания снимка, чтобы сделать его более согласованным и предотвратить потерю данных. | `yes` или `no`. По умолчанию `no` |
| `os_type` | Тип операционной системы установленной в образе. Например, для образов os_type=windows вместо раздела подкачки Linux создается раздел подкачки на основе FAT32, а длина введенного имени хоста ограничивается менее чем 16 символами. | `linux` или `windows` |
| `hw_qemu_guest_agent` | Это параметр, который указывает, будет ли QEMU guest agent включен или выключен для данной виртуальной машины. | `true` или `false`. По умолчанию `false`. |
| `hw_disk_bus` | Specifies the type of disk controller to attach disk devices to. One of scsi, virtio, uml, xen, ide, or usb |  |
| `hw_scsi_model` | Enables the use of VirtIO SCSI (virtio-scsi) to provide block device access for compute instances; by default, instances use VirtIO Block (virtio-blk). |  |

```
openstack image show <ID образа>
```

```
openstack image set --property <имя и значение тега> <ID образа>
```

```
openstack image unset --property <имя и значение тега> <ID образа>
```
