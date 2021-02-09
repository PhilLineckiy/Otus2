# Otus2
The lesson 2: Дисковая подсистема. Работа с mdadm.

Добавить в Vagrantfile еще дисков.
Собрать R0/R5/R10 на выбор.
Сломать/починить raid.
Прописать собранный рейд в конфигурационный файл, чтобы рейд собирался при загрузке.
Создать GPT раздел и 5 партиций.
Vagrantfile, который сразу собирает систему с подключенным рейдом.

1. Работа с Vagrant.

  '''
  vagrant -v - версия Vagrant
  '''
  
Vagrant 2.2.14

  vagrant box list

- centos-7-7      (virtualbox, 0) - используемый box
- centos/7        (virtualbox, 2004.01)
- ubuntu/trusty64 (virtualbox, 20190514.0.0)

Создан Vagrantfile(приложен к OtusHW2 с добавленными дисками), значит можем поднимать нашу виртуальную машину.

  vagrant up

2. Работа с mdadm, сборка RAID

Собираем из дисков RAID. Выбран RAID 5.Опция -l какого уровня RAID создавать,опция - n указывает на кол-во устройств в RAID:

  mdadm --create --verbose /dev/md0 -l 5 -n 5 /dev/sd{b,c,d,e,f}
  
Смотрим состояние рэйда после сборки:
  cat /proc/mdstat

3. Работа с mdadm, поломать/починить RAID5

Ломаем:

  sudo mdadm /dev/md0 --fail /dev/sde
  
Проверяем что диск сломан:

  cat /proc/mdstat
  
Уберем его, чтобы заменить на новый:

  sudo mdadm /dev/md0 --remove /dev/sde - убираем
  sudo mdadm /dev/md0 --add /dev/sde - добавляем
  
Проверяем что процесс rebuild-а окончен успешно:

  cat /proc/mdstat
 
4. Прописываем собранный рейд в конфигурационный файл, чтобы рейд собирался при загрузке.

Для того, чтобы быть уверенным что ОС запомнила какой RAID массив требуется создать и какие компоненты в него входят создадим файл mdadm.conf. Сначала убедимся, что информация верна:

  sudo mdadm --detail --scan --verbose

Теперь создадим mdadm.conf:

  echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
  mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf

Созданный файл:

  cat /etc/mdadm/mdadm.conf
5. Создание GPT раздела и 5 партиций

Создаем раздел GPT на RAID:

  parted -s /dev/md0 mklabel gpt

Создаем партиции:

  parted /dev/md0 mkpart primary ext4 0% 20%
  
  parted /dev/md0 mkpart primary ext4 20% 40%
  
  parted /dev/md0 mkpart primary ext4 40% 60%
  
  parted /dev/md0 mkpart primary ext4 60% 80%
  
  parted /dev/md0 mkpart primary ext4 80% 100%
  
Создаем на этих партициях фс:

  for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done

Монтируем их по каталогам:

  mkdir -p /raid/part{1,2,3,4,5}

for i in $(seq 1 5); do mount /dev/md0p$i /raid/part$i; done

6. Vagrantfile, который сразу собирает систему с подключенным рейдом приложен в репозитории
