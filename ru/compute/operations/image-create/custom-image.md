# Подготовить свой образ диска

{% note info %}

Использовать можно только образы, подготовленные по этой инструкции.

Вы также можете ознакомиться с готовыми образами в [{{ marketplace-full-name }}](/marketplace).

{% endnote %}

Вы можете использовать собственный файл с [образом](../../concepts/image.md) [диска](../../concepts/disk.md) [виртуальной машины](../../concepts/vm.md) под управлением операционной системы из семейства Linux. После подготовки образа [загрузите его](upload.md) в {{ compute-name }}.

Если вы подготовили программное обеспечение, которое может быть полезно другим, [предложите](../../../marketplace/operations/create-product.md) его в {{ marketplace-full-name }}.

{% note warning %}

Образы с [UEFI/EFI](https://ru.wikipedia.org/wiki/Extensible_Firmware_Interface)-загрузчиком не совместимы с {{ compute-name }}. Чтобы загрузиться с дисков объемом более 2 ТБ с разметкой [GUID Partition Table (GPT)](https://ru.wikipedia.org/wiki/Таблица_разделов_GUID), используйте загрузчик [GRUB 2](https://www.gnu.org/software/grub/manual/grub/html_node/BIOS-installation.html).

{% endnote %}

## Настройте ОС в соответствии с требованиями {#requirements}

{% include [image-create-requirements](../../../_includes/compute/image-create-requirements.md) %}

### Установите virtio-драйверы {#virtio}

Для успешной загрузки в ОС должны присутствовать драйверы `virtio-blk`, `virtio-net` и `virtio-pci`, а для работы с [файловыми хранилищами](../../concepts/filesystem.md) {{ compute-name }} — `virtiofs`.

Большинство современных дистрибутивов по умолчанию содержит драйверы `virtio`. Драйверы могут быть скомпилированы в виде отдельных файлов `.ko` или входить в состав самого ядра.

По инструкции ниже вы можете проверить, установлены ли драйверы, и при необходимости добавить их в ОС.
1. Узнайте, входят ли драйверы в конфигурацию ядра:

   {% cut "Как узнать" %}

   Выполните команду:

   ```sh
   grep -E -i "VIRTIO_(BLK|NET|PCI|FS)" /boot/config-$(uname -r)
   ```

   Если на экране не появились строки, начинающиеся на `CONFIG_VIRTIO_BLK=`, `CONFIG_VIRTIO_NET=`, `CONFIG_VIRTIO_PCI=` и `CONFIG_VIRTIO_FS=`, нужно заново скомпилировать ядро Linux с virtio-драйверами. В противном случае переходите к следующим шагам.

   {% endcut %}

1. Если на шаге 1 на экране появились строки `CONFIG_VIRTIO_BLK=y`, `CONFIG_VIRTIO_NET=y`, `CONFIG_VIRTIO_PCI=y` и `CONFIG_VIRTIO_FS=y`, проверьте, что драйверы входят в состав ядра:

   {% cut "Как проверить драйверы в составе ядра" %}

   Выполните команду:

   ```sh
   grep -E "virtio(_blk|_net|_pci|fs)" /lib/modules/"$(uname -r)"/modules.builtin
   ```

   * Если на экране появились строки с файлами `virtio_net.ko`, `virtio_blk.ko`, `virtio_pci.ko` и `virtiofs.ko`, драйверы входят в состав ядра, устанавливать их не нужно.
   * Если на экране не появились такие строки, нужно заново скомпилировать ядро Linux с virtio-драйверами.

   {% endcut %}

1. Если на шаге 1 на экране появились строки `CONFIG_VIRTIO_BLK=m`, `CONFIG_VIRTIO_NET=m`, `CONFIG_VIRTIO_PCI=m` и `CONFIG_VIRTIO_FS=m`, проверьте, что драйверы установлены в качестве модулей ядра:

   {% cut "Как проверить модули ядра" %}

   {% list tabs group=operating_system %}

   - CentOS/Fedora {#centos}

     Выполните следующую команду:

     ```sh
     sudo lsinitrd /boot/initramfs-$(uname -r).img | grep -E "virtio(_blk|_net|_pci|fs)"
     ```

     * Если на экране появились строки с файлами `virtio_net.ko.xz`, `virtio_blk.ko.xz`, `virtio_pci.ko.xz` и `virtiofs.ko.xz`, драйверы установлены в качестве модулей ядра.
     * Если на экране не появились такие строки, создайте резервную копию файла `initramfs` и установите драйверы:

       ```sh
       sudo cp /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak
       sudo mkinitrd -f --with=virtio_blk --with=virtio_net --with=virtio_pci --with=virtiofs /boot/initramfs-$(uname -r).img $(uname -r)
       ```

       Если на экране появилась ошибка `Command 'mkinitrd' not found`, установите драйверы с помощью утилиты `dracut`:

       ```sh
       sudo dracut -f --add-drivers "virtio_blk virtio_net virtio_pci virtiofs" /boot/initramfs-$(uname -r).img $(uname -r)
       ```

       После этого перезапустите ОС и проверьте, что драйверы появились в файле `initramfs` и загрузились:

       ```sh
       sudo lsinitrd /boot/initramfs-$(uname -r).img | grep -E "virtio(_blk|_net|_pci|fs)"
       find /lib/modules/"$(uname -r)"/ -name "virtio*" | grep -E "(blk|net|pci|fs)"
       ```

       После каждой из команд на экране должны появиться строки с файлами `virtio_net.ko.xz`, `virtio_blk.ko.xz`, `virtio_pci.ko.xz` и `virtiofs.ko.xz`.

   - Debian/Ubuntu {#ubuntu}

     Выполните следующую команду:

     ```sh
     lsinitramfs /boot/initrd.img-$(uname -r) | grep -E "virtio(_blk|_net|_pci|fs)"
     ```

     * Если на экране появились строки с файлами `virtio_net.ko`, `virtio_blk.ko`, `virtio_pci.ko` и `virtiofs.ko`, драйверы установлены в качестве модулей ядра.
     * Если на экране не появились такие строки, установите драйверы:

       ```sh
       echo -e "virtio_blk\nvirtio_net\nvirtio_pci\nvirtiofs" | sudo tee -a /etc/initramfs-tools/modules
       sudo update-initramfs -u
       ```

       После этого перезапустите ОС и проверьте, что драйверы появились в файле `initrd` и загрузились:

       ```sh
       lsinitramfs /boot/initrd.img-$(uname -r) | grep -E "virtio(_blk|_net|_pci|fs)"
       find /lib/modules/"$(uname -r)"/ -name "virtio*" | grep -E "(blk|net|pci|fs)"
       ```

       После каждой из команд на экране должны появиться строки с файлами `virtio_net.ko`, `virtio_blk.ko`, `virtio_pci.ko` и `virtiofs.ko`.

   {% endlist %}

   {% endcut %}

### Настройте серийную консоль {#serial-console}

[Серийная консоль](../serial-console/index.md) — это способ получить доступ к ВМ вне зависимости от состояния [сети](../../../vpc/concepts/network.md#network) или ОС. Используйте серийную консоль, например, для устранения неисправностей ВМ или при возникновении проблем с доступом через [SSH](../../../glossary/ssh-keygen.md). Подробнее см. в разделе [{#T}](../serial-console/index.md).

Чтобы к ВМ можно было подключаться с помощью серийной консоли, настройте для образа терминал `ttyS0` (порт COM1) в качестве системной консоли:
1. В файле с настройками GRUB `/etc/default/grub` добавьте в значение параметра `GRUB_CMDLINE_LINUX` опцию `console=ttyS0`. Строка с этим параметром должна иметь такой вид:

   ```sh
   GRUB_CMDLINE_LINUX="foo=bar baz console=ttyS0"
   ```

1. Убедитесь, что в файле `/etc/securetty` с перечислением терминалов, через которые пользователь `root` может войти в ОС, есть строка `ttyS0`. Если файла не существует, создайте его.

   {% note info %}

   В некоторых ОС файл `/etc/securetty` не используется, и пользователю `root` по умолчанию разрешен доступ ко всем терминалам. Если вы хотите настраивать доступ с помощью этого файла, нужно добавить в файл `/etc/pam.d/login` указание на модуль `pam_securetty.so`. Подробнее см. `man pam_securetty` и `man pam.d`.

   {% endnote %}

1. Выполните следующие команды:

   {% list tabs group=operating_system %}

   - CentOS/Fedora {#centos}

     ```sh
     sudo stty -F /dev/ttyS0 9600 # Устанавливает рекомендуемую скорость терминала ttyS0 — 9600 бод.
     sudo grub2-mkconfig -o /boot/grub2/grub.cfg # Генерирует конфигурационный файл для GRUB.
     sudo systemctl start getty@ttyS0 # Запускает getty на терминале ttyS0.
     sudo systemctl enable getty@ttyS0 # Указывает, что getty нужно запускать при каждом включении ОС.
     ```

   - Debian/Ubuntu {#ubuntu}

     ```sh
     sudo stty -F /dev/ttyS0 9600 # Устанавливает рекомендуемую скорость терминала ttyS0 — 9600 бод.
     sudo update-grub # Генерирует конфигурационный файл для GRUB.
     sudo systemctl start getty@ttyS0 # Запускает getty на терминале ttyS0.
     sudo systemctl enable getty@ttyS0 # Указывает, что getty нужно запускать при каждом включении ОС.
     ```

   {% endlist %}

1. Перезапустите ОС.

После [создания ВМ из образа](upload.md#create-vm-from-user-image) ее нужно будет дополнительно [настроить для работы с серийной консолью](../serial-console/index.md). 

## Отключите проверку облачной платформы при создании образа в Amazon EC2 {#ec2}

Если вы создаете образ в Amazon EC2 на основе Amazon Machine Image, `cloud-init` проверяет, что ВМ запускается в Amazon EC2. ВМ в {{ compute-full-name }} не пройдут проверку и могут работать некорректно.

Чтобы этого не произошло, отключите проверку метаданных на ВМ. Для этого создайте в директории `/etc/cloud/cloud.cfg.d` конфигурационный файл, например `99-ec2-datasource.cfg`, и добавьте в него код:

```yaml
#cloud-config
datasource:
  Ec2:
    strict_id: false
```

## Установите драйверы для совместимости с GPU {#gpu-drivers}


Если при работе с ВМ потребуется [GPU](../../concepts/gpus.md), во время подготовки образа [установите драйверы NVIDIA](../vm-operate/install-nvidia-drivers.md).

## Создайте файл с образом {#create-image-file}

{% include [available-image-format](../../../_includes/compute/available-image-format.md) %}

Рекомендуется использовать формат `Qcow2` с оптимизированным размером кластера, чтобы ускорить импорт. Сконвертировать образ из других форматов можно с помощью утилиты `qemu-img`:

```bash
qemu-img convert -p -O qcow2 -o cluster_size=2M <имя_вашего_файла_образа> <имя_нового_файла_образа>
```

{% note info %}

Не используйте программы для сжатия или архивирования при подготовке файла с образом. Для увеличения скорости создания ВМ из образа, вы можете его [оптимизировать](../../concepts/image.md#images-optimized-for-deployment).

{% endnote %}