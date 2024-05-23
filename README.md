# bash-utility
A simple command line bash utility using getopt and getopts

//show_help:
//This function prints help on how to use the script and terminates its execution.

function show_help {
    echo "usage: $0 [option]"
    echo "Опции:"
    echo "  -u, --users         Lists users and their home directories"
    echo "  -p, --processes     Displays a list of running processes"
    echo "  -h, --help          Displays this help"
    echo "  -l PATH, --log PATH Writes output to a file at the given path"
    echo "  -e PATH, --errors PATH Writes errors to a file at the specified path"
    exit 0
}

.........................................................................................................................................................

//list_users:
//This function displays a list of users and their home directories, sorted alphabetically.
//Uses the awk command to process the /etc/passwd file. This file contains information about system users.
//awk -F: Sets the colon (:) as the field separator.
//{ print $1 ": " $6 } prints the first field (username) and the sixth field (home directory), separated by a colon.
//The output of awk is passed to the sort command, which sorts the lines in alphabetical order.

function list_users {
    awk -F: '{ print $1 ": " $6 }' /etc/passwd | sort
}

..........................................................................................................................................................

//list_processes:
//This function lists running processes sorted by PID and intentionally raises an error to demonstrate error handling.
//Uses the command ps -e --sort pid to list all running processes, sorted by their PID.
//ls /nonexistent_directory intentionally throws an error because the specified directory does not exist. This is done to demonstrate error handling.

function list_processes {
    ps -e --sort pid
    ls /nonexistent_directory  # Error
}

............................................................................................................................................................

//Initializing variables for logs and errors
//Variables are initialized to store log and error paths, as well as a variable for the selected action.

LOG_PATH=""
ERROR_PATH=""
ACTION=""

............................................................................................................................................................

//Handling command line arguments using getopt
//This section of the script processes command line arguments using getopt.
//getopt processes command line arguments, determining which options were passed.
//If getopt encounters an error, the show_help function is called and the script exits with code 1.
//eval set -- "$ARGS" splits the arguments into separate words so they can be processed in a while loop.

ARGS=$(getopt -o uphl:e: --long users,processes,help,log:,errors: -n "$0" -- "$@")
if [ $? -ne 0 ]; then
    show_help
    exit 1
fi

eval set -- "$ARGS"

.............................................................................................................................................................

//Parsing arguments
//The while loop loops through all the arguments.
//For each argument the corresponding action is performed:
//-u | --users: Sets the action to users.
//-p | --processes: Sets the action to processes.
//-h | --help: Calls the show_help function to display help.
//-l | --log: Sets the path for the log file.
//-e | --errors: Sets the path for the error file.
//--: Completes argument parsing.

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

..............................................................................................................................................................

//Checking the availability of paths for logs and errors
//Checks if the paths for logs and errors are set.
//If the path is set, tries to create the file using touch.
//If touch returns a non-zero exit code, displays an error message and exits the script with code 1.

if [ -n "$LOG_PATH" ]; then
    touch "$LOG_PATH" 2>/dev/null
    if [ $? -ne 0 ]; then
        echo "Cannot write to log file: $LOG_PATH" >&2
        exit 1
    fi
fi

if [ -n "$ERROR_PATH" ]; then
    touch "$ERROR_PATH" 2>/dev/null
    if [ $? -ne 0 ]; then
        echo "Cannot write to error file: $ERROR_PATH" >&2
        exit 1
    fi
fi

.....................................................................................................................................................................

//Performing an action and displaying the results
//Depending on the value of the ACTION variable, the corresponding function (list_users or list_processes) is executed.
//If a log file path is set, the function output is redirected to that file.
//If a path for an error file is set, errors are redirected to that file.
//If both paths are set, output and errors are redirected accordingly.
//If the action is not recognized, show_help is called.

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
