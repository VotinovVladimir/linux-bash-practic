# Пояснения к проекту

## Report: Part 1. Проба пера

### Цель задания

Написать простой bash-скрипт, принимающий один текстовый параметр и проверяющий корректность его ввода. Если введённый параметр является числом, вывести сообщение об ошибке.

Реализован следующий bash-скрипт:

```bash
#!/bin/bash

# Проверка количества аргументов
if [ $# -ne 1 ]; then
    echo "ОШИБКА: необходимо указать ровно один текстовый параметр."
    exit 1
fi

param="$1"

# Проверка: параметр не должен быть числом
if [[ "$param" =~ ^[0-9]+$ ]]; then
    echo "ОШИБКА: параметр не должен быть числом."
    exit 1
fi

# Вывод параметра
echo "Параметр: $param"

exit 0
```

## Part 2. Исследование системы

### Цель задания

Разработать bash-скрипт, который собирает системную информацию и выводит её в удобочитаемом виде. Дополнительно предлагается сохранить собранную информацию в файл, если пользователь подтвердит своё желание.

Реализованы три отдельных bash-файла, каждый из которых выполняет свою задачу:

1. **main.sh** — основной файл, объединяющий остальные модули и управляющий процессом.
2. **system\_info.sh** — модуль сбора системной информации.
3. **print\_info.sh** — модуль вывода информации на экран.
4. **save\_info.sh** — модуль записи информации в файл.

**main.sh**

```bash
#!/bin/bash
# main.sh

source "$(dirname "$0")/system_info.sh"
source "$(dirname "$0")/print_info.sh"
source "$(dirname "$0")/save_info.sh"

get_system_info
print_system_info
save_system_info

exit 0
```

**system\_info.sh**

```bash
get_system_info() {
    HOSTNAME=$(hostname)
    TIMEZONE="$(cat /etc/timezone) UTC $(date +"%:::z")"
    USER=$(whoami)
    OS=$(cat /etc/issue | awk '{printf $1" "$2}')
    DATE=$(date +"%d %B %Y %T")
    UPTIME=$(uptime -p)
    UPTIME_SEC=$(cat /proc/uptime | awk '{print $1}')
    IP=$(networkctl status | grep "Address" | awk {'print $2'})
    MASK=$(ip -4 addr show | grep inet | awk '{print "255.255.255.0"}' | head -1)
    GATEWAY=$(ip r | grep -e "default" | awk '{print $3}' | head -1)
    RAM_TOTAL=$(free -mt | grep "Mem" | awk '{printf "%.3f GB", $2/1024}')
    RAM_USED=$(free -mt | grep "Mem" | awk '{printf "%.3f GB", $3/1024}')
    RAM_FREE=$(free -mt | grep "Mem" | awk '{printf "%.3f GB", $4/1024}')
    SPACE_ROOT=$(df / | grep "/" | awk '{printf "%.2f MB", $2/1024}')
    SPACE_ROOT_USED=$(df / | grep "/" | awk '{printf "%.2f MB", $3/1024}')
    SPACE_ROOT_FREE=$(df / | grep "/" | awk '{printf "%.2f MB", $4/1024}')

```

**print_info.sh**

Файл, отвечающий за вывод информации на экран:

```bash
print_system_info() {
    echo "HOSTNAME = $HOSTNAME"
    echo "TIMEZONE = $TIMEZONE"
    echo "USER = $USER"
    echo "OS = $OS"
    echo "DATE = $DATE"
    echo "UPTIME = $UPTIME"
    echo "UPTIME_SEC = $UPTIME_SEC"
    echo "IP = $IP"
    echo "MASK = $MASK"
    echo "GATEWAY = $GATEWAY"
    echo "RAM_TOTAL = $RAM_TOTAL"
    echo "RAM_USED = $RAM_USED"
    echo "RAM_FREE = $RAM_FREE"
    echo "SPACE_ROOT = $SPACE_ROOT"
    echo "SPACE_ROOT_USED = $SPACE_ROOT_USED"
    echo "SPACE_ROOT_FREE = $SPACE_ROOT_FREE"
}
```

**save_info.sh**

Файл, позволяющий сохранять собранные данные в файл:

```bash
save_system_info() {
    read -p "Do you want to save this information to a file? (Y/N): " answer
    if [[ $answer == Y || $answer == y ]]; then
        filename=$(date "+%d_%m_%y_%H_%M_%S.status")

        echo "HOSTNAME = $HOSTNAME" > "$filename"
        echo "TIMEZONE = $TIMEZONE" >> "$filename"
        echo "USER = $USER" >> "$filename"
        echo "OS = $OS" >> "$filename"
        echo "DATE = $DATE" >> "$filename"
        echo "UPTIME = $UPTIME" >> "$filename"
        echo "UPTIME_SEC = $UPTIME_SEC" >> "$filename"
        echo "IP = $IP" >> "$filename"
        echo "MASK = $MASK" >> "$filename"
        echo "GATEWAY = $GATEWAY" >> "$filename"
        echo "RAM_TOTAL = $RAM_TOTAL" >> "$filename"
        echo "RAM_USED = $RAM_USED" >> "$filename"
        echo "RAM_FREE = $RAM_FREE" >> "$filename"
        echo "SPACE_ROOT = $SPACE_ROOT" >> "$filename"
        echo "SPACE_ROOT_USED = $SPACE_ROOT_USED" >> "$filename"
        echo "SPACE_ROOT_FREE = $SPACE_ROOT_FREE" >> "$filename"

        echo "Information saved to $filename"
    else
        echo "Information not saved"
    fi
}
```

## Part 3. Визуальное оформление вывода для скрипта исследования системы

### Цель задания

Необходимо преобразовать существующий bash-скрипт из второй части таким образом, чтобы обеспечить визуализацию данных путём изменения цветов текста и фона заголовков и значений. Для реализации использованы четыре обязательных параметра, определяющие соответствующие цвета.

Для реализации визуализации создана новая структура скриптов:

1. **Color.sh** - вспомогательная библиотека для преобразования чисел в коды цветов.
2. **validate_input.sh** - модуль для проверки правильности ввода параметров.
3. **print_info.sh** - утилита для красивого вывода данных с применением цветов.
4. **main.sh** - главный файл, объединяющий все предыдущие компоненты.

**color.sh**

Библиотека для получения цветовой схемы:

```bash
get_color() {
  case $1 in
    1) echo "7" ;;
    2) echo "1" ;;
    3) echo "2" ;;
    4) echo "4" ;;
    5) echo "5" ;;
    6) echo "0" ;;
    *) echo "" ;;
  esac
}
```

**validate_input.sh**

Модуль для проверки валидности введенных параметров:

```bash
validate_input() {
  if [ $# -ne 4 ]; then
    echo "Error: You must provide exactly 4 numeric parameters (1-6)."
    exit 1
  fi

  BG_NAME=$(get_color $1)
  FG_NAME=$(get_color $2)
  BG_VALUE=$(get_color $3)
  FG_VALUE=$(get_color $4)

  if [[ -z $BG_NAME || -z $FG_NAME || -z $BG_VALUE || -z $FG_VALUE ]]; then
    echo "Error: Parameters must be numeric values between 1 and 6."
    exit 1
  fi

  if [[ $BG_NAME == $FG_NAME ]]; then
    echo "Error: Background and font color for value names must not match."
    exit 1
  elif [[ $BG_VALUE == $FG_VALUE ]]; then
    echo "Error: Background and font color for values must not match."
    exit 1
  fi
}
```

**print_info.sh**

Файл для оформления вывода информации:

```bash
print_info() {
  local name=$1
  local value=$2
  echo -e "\033[4${BG_NAME};3${FG_NAME}m${name}\033[0m = \033[4${BG_VALUE};3${FG_VALUE}m${value}\033[0m"
}
```

**main.sh**

Главный файл, объединяющий все модули:

```bash
#!/bin/bash

source "$(dirname "$0")/color.sh"
source "$(dirname "$0")/validate_input.sh"
source "$(dirname "$0")/../02/system_info.sh"
source "$(dirname "$0")/print_info.sh"

validate_input "$@"
get_system_info

print_info "HOSTNAME" "$HOSTNAME"
print_info "TIMEZONE" "$TIMEZONE"
print_info "USER" "$USER"
print_info "OS" "$OS"
print_info "DATE" "$DATE"
print_info "UPTIME" "$UPTIME"
print_info "UPTIME_SEC" "$UPTIME_SEC"
print_info "IP" "$IP"
print_info "MASK" "$MASK"
print_info "GATEWAY" "$GATEWAY"
print_info "RAM_TOTAL" "$RAM_TOTAL"
print_info "RAM_USED" "$RAM_USED"
print_info "RAM_FREE" "$RAM_FREE"
print_info "SPACE_ROOT" "$SPACE_ROOT"
print_info "SPACE_ROOT_USED" "$SPACE_ROOT_USED"
print_info "SPACE_ROOT_FREE" "$SPACE_ROOT_FREE"

exit 0
```

## Part 4. Конфигурирование визуального оформления вывода для скрипта исследования системы

### Цель задания

Преобразовать существующий bash-скрипт из третьей части таким образом, чтобы параметры, влияющие на визуальное оформление, задавались через конфигурационный файл, а не через аргументы командной строки. Пользователь должен иметь возможность настроить внешний вид вывода информации удобным способом.

Новая реализация основана на чтении настроек из отдельного конфигурационного файла `colors.conf`, который хранит значения четырех параметров, управляющих цветом фона и шрифта для двух столбцов. Эти параметры могут быть пропущены, тогда используются значения по умолчанию.

**colors_name.sh**

Модуль для сопоставления номеров цветов их названиям:

```bash
get_color_name() {
  case $1 in
    0) echo "black" ;;
    1) echo "red" ;;
    2) echo "green" ;;
    4) echo "blue" ;;
    5) echo "purple" ;;
    7) echo "white" ;;
  esac
}
```

**colors.conf**

Примеры конфигурации:

```conf
column1_background=2
column1_font_color=4
column2_background=5
column2_font_color=1
```

**main.sh**

Основной файл, собирающий всю необходимую функциональность вместе:

```bash
#!/bin/bash

source "$(dirname "$0")/colors_name.sh"
source "$(dirname "$0")/validate_input.sh"
source "$(dirname "$0")/../03/color.sh"
source "$(dirname "$0")/../03/print_info.sh"
source "$(dirname "$0")/../02/system_info.sh"

CONF_FILE="$(dirname "$0")/colors.conf"

DEFAULT_BG1=6
DEFAULT_FG1=1
DEFAULT_BG2=2
DEFAULT_FG2=4

if [[ -f "$CONF_FILE" ]]; then
  source "$CONF_FILE"
fi

BG_NAME=$(get_color ${column1_background:-$DEFAULT_BG1})
FG_NAME=$(get_color ${column1_font_color:-$DEFAULT_FG1})
BG_VALUE=$(get_color ${column2_background:-$DEFAULT_BG2})
FG_VALUE=$(get_color ${column2_font_color:-$DEFAULT_FG2})

validate_input
get_system_info

print_info "HOSTNAME" "$HOSTNAME"
print_info "TIMEZONE" "$TIMEZONE"
print_info "USER" "$USER"
print_info "OS" "$OS"
print_info "DATE" "$DATE"
print_info "UPTIME" "$UPTIME"
print_info "UPTIME_SEC" "$UPTIME_SEC"
print_info "IP" "$IP"
print_info "MASK" "$MASK"
print_info "GATEWAY" "$GATEWAY"
print_info "RAM_TOTAL" "$RAM_TOTAL"
print_info "RAM_USED" "$RAM_USED"
print_info "RAM_FREE" "$RAM_FREE"
print_info "SPACE_ROOT" "$SPACE_ROOT"
print_info "SPACE_ROOT_USED" "$SPACE_ROOT_USED"
print_info "SPACE_ROOT_FREE" "$SPACE_ROOT_FREE"

get_color_name

echo ""
echo "Column 1 background = ${column1_background:-default} ($(get_color_name "$BG_NAME"))"
echo "Column 1 font color = ${column1_font_color:-default} ($(get_color_name "$FG_NAME"))"
echo "Column 2 background = ${column2_background:-default} ($(get_color_name "$BG_VALUE"))"
echo "Column 2 font color = ${column2_font_color:-default} ($(get_color_name "$FG_VALUE"))"

exit 0
```

**validate_input.sh**

Модуль для проверки введенных параметров на коректность:

```bash
validate_input() {
  if [[ -z $BG_NAME || -z $FG_NAME || -z $BG_VALUE || -z $FG_VALUE ]]; then
    echo "Error: Parameters must be numeric values between 1 and 6."
    exit 1
  fi

  if [[ $BG_NAME == $FG_NAME ]]; then
    echo "Error: Background and font color for value names must not match."
    exit 1
  elif [[ $BG_VALUE == $FG_VALUE ]]; then
    echo "Error: Background and font color for values must not match."
    exit 1
  fi
}
```

## Part 5. Исследование файловой системы

### Цель задания

Создать bash-скрипт, который исследует указанную директорию и выводит подробную статистику о её содержимом, включая общее количество папок, топ-5 папок по размеру, типы файлов, топ-10 файлов по размеру и топ-10 исполняемых файлов с вычисленным MD5-хэшем. Важной частью является предоставление подробной статистики и расчёт времени исполнения самого скрипта.

1. **main.sh** - Главный файл, контролирующий выполнение основных операций.
2. **output.sh** - Модуль, ответственный за формирование конечного вывода на экран.
3. **stat.sh** - Библиотека для подсчета статистической информации о директориях и файлах.
4. **utils.sh** - Утилиты для измерения времени выполнения скрипта и обработки параметров.

**output.sh**

Отвечает за красывый вывод результатов на экран:

```bash
display_output() {
  echo "Total number of folders (including all nested ones) = $TOTAL_FOLDERS"
  echo "TOP 5 folders of maximum size arranged in descending order (path and size):"
  echo "$TOP_FOLDERS"
  echo "Total number of files = $TOTAL_FILES"
  echo "Number of:"
  echo "Configuration files (with the .conf extension) = $CONF_FILES"
  echo "Text files = $TEXT_FILES"
  echo "Executable files = $EXEC_FILES"
  echo "Log files (with the .log extension) = $LOG_FILES"
  echo "Archive files = $ARCH_FILES"
  echo "Symbolic links = $SYMLINKS"
  echo "TOP 10 files of maximum size arranged in descending order (path, size and type):"
  echo "$TOP_FILES"
  echo "TOP 10 executable files of the maximum size arranged in descending order (path, size and MD5 hash of file):"
  echo "$TOP_EXEC_FILES"
}
```

**stat.sh**

Сбор всей необходимой статистики о файлах и папках:

```bash
collect_stats() {
  DIR="$1"
  TOTAL_FOLDERS=$(find "$DIR" -type d | wc -l)
  TOP_FOLDERS=$(du -h "$DIR" 2>/dev/null | sort -hr | head -5 | awk '{print NR " - " $2 ", " $1}')
  TOTAL_FILES=$(find "$DIR" -type f | wc -l)
  CONF_FILES=$(find "$DIR" -type f -name "*.conf" | wc -l)
  TEXT_FILES=$(find "$DIR" -type f -name "*.txt" | wc -l)
  EXEC_FILES=$(find "$DIR" -type f -executable | wc -l)
  LOG_FILES=$(find "$DIR" -type f -name "*.log" | wc -l)
  ARCH_FILES=$(find "$DIR" -type f $ -name "*.tar" -o -name "*.gz" -o -name "*.zip" $ | wc -l)
  SYMLINKS=$(find "$DIR" -type l | wc -l)
  TOP_FILES=$(find "$DIR" -type f -printf "%s %p\n" 2>/dev/null | sort -nr | head -n 10 | nl -w2 -s' ' | while read num size file; do
      ext="${file##*.}"
      [[ "$file" == "$ext" ]] && ext="none"
      echo "$num - $file, $(numfmt --to=iec $size), $ext"
  done)
  TOP_EXEC_FILES=$(find "$DIR" -type f -executable -printf "%s %p\n" 2>/dev/null | sort -nr | head -10 | nl -w2 -s' ' | while read num size file; do
      md5=$(md5sum "$file" | awk '{print $1}')
      echo "$num - $file, $size bytes, $md5"
  done)
}
```

**utils.sh**

Функции для расчета времени выполнения и проверок:

```bash
start_timer() {
  START_TIME=$(date +%s.%N)
}

end_timer() {
  END_TIME=$(date +%s.%N)
  EXEC_TIME=$(echo "$END_TIME - $START_TIME" | bc)
  printf -v EXEC_TIME "%.9f" "$EXEC_TIME"
}

print_exec_time() {
  echo "Script execution time (in seconds) = $EXEC_TIME"
}

directory_check() {
  if [[ $# -ne 1 || ! -d "$1" ]]; then
    echo "Usage: <path_to_dir/>"
    exit 1
  fi
}
```

**main.sh**

Управляющий процесс выполнения всех компонентов:

```bash
#!/bin/bash

source "$(dirname "$0")/utils.sh"
source "$(dirname "$0")/stat.sh"
source "$(dirname "$0")/output.sh"

start_timer

directory_check "$1"
DIR=$1

collect_stats "$DIR"

display_output

end_timer
print_exec_time

exit 0
```
