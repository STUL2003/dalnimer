# dalnimer

_______________________________________________

Димгностика системы:
```bash
lsusb

ls -la /dev/ttyUSB* #проверить последовательные порты


dmesg | grep -i "usb\|tty\|cp210" #проверить драйверы в ядре

# Проверить загруженные модули
lsmod | grep cp210
# Вывод: Должны быть cp210x и usbserial
```

Диагностика порта:
```bash
# Проверить настройки порта
sudo stty -F /dev/ttyUSB0 -a

sudo stty -F /dev/ttyUSB0 38400 cs8 -parenb -cstopb cread clocal -crtscts #скорость 38400, 8 бит данных, нет четности, 1 стоп-бит

sudo stty -F /dev/ttyUSB0 -a | grep -E "speed|cs8"
```

Проверка отправки (НЕ РАБОТАЕТ):
```bash
printf '\x55\xF8\x00\x00\xF8\xAA' | sudo tee /dev/ttyUSB0

# Проверить буфер порта после отправки
sudo timeout 1s cat /dev/ttyUSB0 | hexdump -C
```

Подбор скорости:
``` bash
#!/bin/bash
echo "Тест скоростей"

for speed in 9600 19200 38400 57600 115200; do
  echo "=== Скорость $speed ==="
  sudo stty -F /dev/ttyUSB0 $speed raw
  printf '\x55\xF8\x00\x00\xF8\xAA' > /dev/ttyUSB0
  sleep 0.3
  response=$(sudo timeout 0.5s cat /dev/ttyUSB0 | wc -c)
  
  if [ $response -gt 0 ]; then
    echo "ЕСТЬ ОТВЕТ: $response байт"
    sudo timeout 0.5s cat /dev/ttyUSB0 | hexdump -C | head -5
  else
    echo "НЕТ ОТВЕТА"
  fi
  echo
done
```

Еще один способ мониторинга:
```bash
#1-ый терминал
sudo stty -F /dev/ttyUSB0 38400 raw
sudo cat /dev/ttyUSB0 | hexdump -C

#2-ой терминал
sudo printf '\x55\xF8\x00\x00\xF8\xAA' > /dev/ttyUSB0  # Статус
sudo printf '\x55\xF2\x00\x00\xF2\xAA' > /dev/ttyUSB0  # Измерение
sudo printf '\x55\xF0\x00\x00\xF0\xAA' > /dev/ttyUSB0  # Самотест
```

Еще некоторые команды:
```bash
# Почистить буферы порта
sudo stty -F /dev/ttyUSB0 38400
sudo echo -n > /dev/ttyUSB0  # Очистка вывода
sudo cat /dev/ttyUSB0 > /dev/null & 

# Сбросить USB порт
sudo bash -c 'echo 0 > /sys/bus/usb/devices/1-2/authorized'
sudo bash -c 'echo 1 > /sys/bus/usb/devices/1-2/authorized'

# Перезагрузить драйвер
sudo modprobe -r cp210x
sudo modprobe cp210x
```
