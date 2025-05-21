# Детальное руководство по сборке ядра Linux

## 1. Подготовка исходников

### Получение исходного кода
Есть два основных способа получить исходники ядра Linux:

1. **Официальный архив с kernel.org**:
```bash
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.5.tar.xz
tar xvf linux-6.5.tar.xz
cd linux-6.5
```

2. **Через git (для разработки)**:
```bash
git clone git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
cd linux
git checkout v6.5  # Для конкретной версии
```

### Выбор версии ядра
- **Stable**: Стабильные выпуски (рекомендуется для production)
- **LTS**: Долгосрочная поддержка (5+ лет обновлений)
- **Mainline**: Последняя разработка (может быть нестабильной)

## 2. Конфигурация ядра

### Основные цели конфигурации

#### `make defconfig`
**Назначение**: Создаёт базовую конфигурацию для архитектуры
**Использование**:
```bash
make ARCH=x86_64 defconfig
```
**Примечание**: Для ARM используйте `ARCH=arm` и часто требуется указать:
```bash
make ARCH=arm multi_v7_defconfig
```

#### `make localmodconfig`
**Назначение**: Создаёт конфиг только с используемыми модулями
**Пример**:
```bash
lsmod > /tmp/my-modules.txt
make LSMOD=/tmp/my-modules.txt localmodconfig
```
**Преимущества**:
- Уменьшает время сборки
- Сокращает размер ядра
**Недостатки**:
- Может пропустить необходимые для загрузки модули

#### `make oldconfig` vs `make olddefconfig`
**oldconfig**:
- Интерактивно спрашивает про новые параметры
```bash
make oldconfig
```

**olddefconfig**:
- Автоматически принимает значения по умолчанию
```bash
make olddefconfig
```

#### Графические конфигураторы
1. `make menuconfig` (ncurses/TUI)
2. `make nconfig` (улучшенный TUI)
3. `make xconfig` (Qt)
4. `make gconfig` (GTK)

**Совет**: Для серверов используйте `menuconfig`, для рабочих станций - `xconfig/gconfig`

## 3. Сборка ядра

### Основные цели сборки

#### Полная сборка
```bash
make -j$(nproc)
```
Эквивалентно:
```bash
make all
```

#### Отдельные компоненты
```bash
make bzImage      # Основное ядро (x86)
make modules      # Только модули
make dtbs         # Device Tree Blobs (ARM)
```

### Оптимизация сборки

1. **Параллельная сборка**:
```bash
make -j$(nproc)   # Использует все ядра CPU
```

2. **Кэширование компиляции** (ccache):
```bash
sudo apt install ccache
export CC="ccache gcc"
make -j$(nproc)
```

3. **Уменьшение размера**:
```bash
make LOCALVERSION="" KERNELRELEASE=custom -j$(nproc)
```

## 4. Установка ядра

### Установка модулей
```bash
sudo make modules_install
```

### Установка ядра
```bash
sudo make install
```
Это:
1. Копирует ядро в /boot
2. Создаёт initramfs
3. Обновляет загрузчик

### Обновление загрузчика

#### Для GRUB:
```bash
sudo update-grub
```

#### Для systemd-boot:
```bash
sudo bootctl update
```

## 5. Отладка и устранение ошибок

### Типичные проблемы

1. **Отсутствующие зависимости**:
```bash
sudo apt install build-essential libncurses-dev bison flex libssl-dev libelf-dev
```

2. **Ошибки конфигурации**:
```bash
make distclean
make defconfig
```

3. **Проблемы с модулями**:
```bash
dmesg | grep -i error
journalctl -k -b -1  # Для предыдущей загрузки
```

### Анализ размера ядра
```bash
ls -lh vmlinux  # Несжатое ядро
ls -lh arch/x86/boot/bzImage  # Сжатое ядро
```

## 6. Дополнительные настройки

### Безопасность
```bash
make menuconfig
```
Рекомендуемые опции:
- CONFIG_STRICT_DEVMEM=y
- CONFIG_SECURITY=y
- CONFIG_SECURITY_SELINUX=y

### Сборка для встраиваемых систем
```bash
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j$(nproc)
```

### Подпись модулей
```bash
openssl req -new -x509 -newkey rsa:2048 -keyout key.priv -outform DER -out key.der -nodes -days 36500 -subj "/CN=My Key/"
```

Добавить в конфиг:
```
CONFIG_MODULE_SIG=y
CONFIG_MODULE_SIG_ALL=y
CONFIG_MODULE_SIG_KEY="key.der"
```

## Заключение

### Рекомендуемый workflow
1. `make defconfig`
2. `make menuconfig` (опционально)
3. `make -j$(nproc)`
4. `sudo make modules_install`
5. `sudo make install`
6. `sudo update-grub`

### Полезные ссылки
- [Официальная документация](https://www.kernel.org/doc/html/latest/)
- [Kernel Build System](https://www.kernel.org/doc/html/latest/kbuild/index.html)
- [Kernel Parameters](https://www.kernel.org/doc/html/latest/admin-guide/kernel-parameters.html)

Это руководство покрывает все основные аспекты сборки ядра Linux. Для специфических задач (например, real-time ядро или особые аппаратные платформы) могут потребоваться дополнительные настройки.
