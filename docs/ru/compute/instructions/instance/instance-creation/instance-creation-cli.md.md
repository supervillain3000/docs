## Подготовительные шаги

1. Выполните [установку]() и пройдите [аутентификацию]() OpenStack CLI.

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
	```shell
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

