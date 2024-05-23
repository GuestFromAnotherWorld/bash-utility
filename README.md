# bash-utility
A simple command line bash utility using getopt and getopts

#!/bin/bash

# Функция для вывода справки
function show_help {
    echo "Использование: $0 [опции]"
    echo "Опции:"
    echo "  -u, --users         Выводит перечень пользователей и их домашних директорий"
    echo "  -p, --processes     Выводит перечень запущенных процессов"
    echo "  -h, --help          Выводит эту справку"
    echo "  -l PATH, --log PATH Записывает вывод в файл по заданному пути"
    echo "  -e PATH, --errors PATH Записывает ошибки в файл по заданному пути"
    exit 0
}

# Функция для вывода пользователей
function list_users {
    awk -F: '{ print $1 ": " $6 }' /etc/passwd | sort
}

# Функция для вывода процессов (добавлен пример с ошибкой)
function list_processes {
    ps -e --sort pid
    ls /nonexistent_directory  # Эта команда вызовет ошибку
}

# Инициализация переменных для логов и ошибок
LOG_PATH=""
ERROR_PATH=""
ACTION=""

# Обработка аргументов командной строки с использованием getopt
ARGS=$(getopt -o uphl:e: --long users,processes,help,log:,errors: -n "$0" -- "$@")
if [ $? -ne 0 ]; then
    show_help
    exit 1
fi

eval set -- "$ARGS"

# Разбор аргументов
while true; do
    case "$1" in
        -u | --users)
            ACTION="users"
            shift
            ;;
        -p | --processes)
            ACTION="processes"
            shift
            ;;
        -h | --help)
            show_help
            shift
            ;;
        -l | --log)
            LOG_PATH="$2"
            shift 2
            ;;
        -e | --errors)
            ERROR_PATH="$2"
            shift 2
            ;;
        --)
            shift
            break
            ;;
        *)
            show_help
            ;;
    esac
done

# Проверка доступности путей для логов и ошибок
if [ -n "$LOG_PATH" ]; then
    touch "$LOG_PATH" 2>/dev/null
    if [ $? -ne 0 ]; then
        echo "Невозможно записать в файл лога: $LOG_PATH" >&2
        exit 1
    fi
fi

if [ -n "$ERROR_PATH" ]; then
    touch "$ERROR_PATH" 2>/dev/null
    if [ $? -ne 0 ]; then
        echo "Невозможно записать в файл ошибок: $ERROR_PATH" >&2
        exit 1
    fi
fi

# Выполнение действия и вывод результатов
case $ACTION in
    users)
        if [ -n "$LOG_PATH" ]; then
            list_users > "$LOG_PATH"
        else
            list_users
        fi
        ;;
    processes)
        if [ -n "$LOG_PATH" ] && [ -n "$ERROR_PATH" ]; then
            list_processes > "$LOG_PATH" 2> "$ERROR_PATH"
        elif [ -n "$LOG_PATH" ]; then
            list_processes > "$LOG_PATH" 2>&1
        elif [ -n "$ERROR_PATH" ]; then
            list_processes 2> "$ERROR_PATH"
        else
            list_processes
        fi
        ;;
    *)
        show_help
        ;;
esac 

