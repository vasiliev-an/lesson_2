# **Дальнейшие действия на уже собранном RAID 5**

### **Создание конфигурационного файла mdadm.conf**


Убедимся, что информация о RAID верна:

```
sudo mdadm --detail --scan --verbose
```

Переходим в режим суперпользователя и вводим пароль:

```
su
```

Создаём папку mdadm:

```
sudo mkdir /etc/mdadm
```

Создаём файл mdadm.conf

```
echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
```

### **Сломать/починить RAID**

"Зафейлим" одно из блочных устройств:

```
mdadm /dev/md0 --fail /dev/sdf
```

Смотрим статус RAID:

```
cat /proc/mdstat
mdadm -D /dev/md0
```

Удалим “сломанный” диск из массива:

```
mdadm /dev/md0 --remove /dev/sdf
```

Добавляем новый диск в RAID:

```
mdadm /dev/md0 --add /dev/sdf
```

### **Создаём GPT раздел, пять партиций и смонтируем их на диск**

Создаем раздел GPT на RAID:

```
parted -s /dev/md0 mklabel gpt
```

Создаем партиции:

```
parted /dev/md0 mkpart primary ext4 0% 20%
parted /dev/md0 mkpart primary ext4 20% 40%
parted /dev/md0 mkpart primary ext4 40% 60%
parted /dev/md0 mkpart primary ext4 60% 80%
parted /dev/md0 mkpart primary ext4 80% 100%
```

Создаём на этих партициях ФС:

```
for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
```

Монтируем их по каталогам

```
mkdir -p /raid/part{1,2,3,4,5}
for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done
```