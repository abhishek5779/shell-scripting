# Shell Scripting Challenges

## Write a simple Bash script that prints “Hello DevOps” along with the current date and time.


```bash
#!/bin/bash
echo "Hello World, Current date & time is $(TZ="Asia/Kolkata" date +"%d-%m-%Y %H:%M:%S IST")"
```


## Create a script that checks if a website (e.g., https://www.learnxops.com) is reachable using curl or ping. Print a success or failure message.


```bash
#!/bin/bash
URL="https://www.learnxops.com"
HTTP_CODE=$(curl -ksL -w "%{http_code}" ${URL} -o index.html)
echo $HTTP_CODE

if [ $HTTP_CODE -eq 200 ]
then
    echo "SUCCESS: You are able to reach the webpage ${URL}"
else
    echo "FAILURE: You have received the HTTP CODE $HTTP_CODE"
fi
```

Here in the above curl command,`
* `-k` is used to Ignores SSL certificate validation.
* `-s` is used to Suppresses progress meter and error messages, making the output cleaner.
* `-w` Outputs the HTTP status code of the response after the request completes

Another approach could be like below where we only focus on the header - 

```bash
#!/bin/bash
URL="https://www.learnxops1.com"
HOST="learnxops.com"

if curl -Is $URL --max-time 5 | head -n 1 | grep -q "200\|301\|302"
then
    echo "SUCCESS: ${URL} is reachable via curl"
else
    echo "FAILURE: ${URL} is not reachable via curl, let's try using ping"
    if ping -n 2 -w 5 ${HOST} >/dev/null 2>&1
    then
        echo "SUCCESS: ${HOST} is reacahble via ping!"
    else
        echo "FAILURE: ${HOST} is not reacahble via ping as well"
    fi
fi
```

* `-I` - Sends a HEAD request (faster than full request).
* `--max-time` - Times out if no response within 5 seconds.
* `-q` - Runs in quiet mode, suppressing output and returning only an exit status.
* `-n` - Tells ping to send how many number of requests
* `-w` - Timeout value for a packet


## Write a script that takes a filename as an argument, checks if it exists, and prints the content of the file accordingly.


```bash
#!/bin/bash

if [ $# -eq 0 ]
then
    echo "ERROR : FileName is not provided"
    echo "Usage : ./check_file.sh <file-name>"
    exit 1
fi

fileName="$1"

if [ -f "$fileName" ]
then
    echo "${fileName} file exists. Displaying content"
    cat $fileName
else
    echo "ERROR: $fileName file doesn't exists"
    ls -ltr
fi
```


## Create a script that lists all running processes and writes the output to a file named process_list.txt.


```bash
#!/bin/bash
ps aux >> process_file.txt
echo "Running Process list saved to process_file.txt"
```


## Write a script that installs multiple packages at once (e.g., git, vim, curl). The script should check if each package is already installed before attempting installation.


```bash
#!/bin/bash
packages=("git" "vim" "curl")

#Check whether package exist or not
for pkg in ${packages[@]}
do
    if dpkg -l | grep -qw "$pkg"
    then
        echo "$pkg is already installed"
    else
        echo "$pkg is not installed. Installing ...."
        sudo apt-get install -y $pkg
    fi
done
```

* `dpkg -l` - List the packages concisely.


## Create a script that monitors CPU and memory usage every 5 seconds and logs the results to a file.


```bash
#!/bin/bash
LOG_FILE="resource_usage.log"

echo "Monitoring CPU and Memory usage in log file ${LOG_FILE}"
echo "TIMESTAMP | CPU(%)  | MEMORY(%)" > ${LOG_FILE}

while true
do
    TIMESTAMP=$(TZ="Asia/Kolkata" date +"%d-%m-%Y %H:%M:%S IST")
    CPU=$(top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}')
    MEMORY=$(free | awk '/Mem/ {print "%.2f", $3/$2 * 100}')

    echo "$TIMESTAMP | $CPU  | $MEMORY" >> ${LOG_FILE}
    sleep 5
done
```

* `-bn1` - Runs the top command in batch mode `(-b)` for a single iteration `(-n1)`.


## Write a script that automatically deletes log files older than 7 days from /var/log.


```bash
#!/bin/bash
dir="/var/log"
find ${dir} -type f -name "*.log" -mtime +7 -delete
```

* `+7` -  The +7 means "greater than 7 days". That means command will find and delete log files that were last modified more than 7 days ago.
* `-7` - The -7 means "less than 7 days". That means command will find and delete log files that were modified less than 7 days ago.


## Automate user account creation – Write a script that takes the username as an argument, checks, if the user exists, gives the message “user already exists“ else creates a new user, adds it to a “devops“ group, and sets up a default home directory


```bash
#!/bin/bash
if [ $# -eq 0 ]
then
    echo "ERROR : Username is not provided"
    echo "Usage : ./check_user.sh <user-name>"
    exit 1
fi

USERNAME=$1
GROUP="devops"
if cat /etc/passwd | grep -qw "$USERNAME"
then
    echo "$USERNAME already exists"
else
    echo "Checking group exists or not"
    if ! getent group $GROUP >/dev/null
    then
        echo "Creating Group $GROUP"
        sudo groupadd $GROUP
    fi
    echo "Creating Username ${USERNAME}"
    sudo useradd -m -s /bin/bash -g ${GROUP} ${USERNAME}
    # Set a default password (optional, force change on first login)
    echo "$USERNAME:ChangeMe123" | sudo chpasswd
    sudo passwd --expire "$USERNAME"
    echo "✅ User '$USERNAME' created successfully and added to group '$GROUP'."
    echo "ℹ️ Default password: ChangeMe123 (User must change it on first login)"
fi
```
* `-m` → Creates a home directory.

* `-s /bin/bash` → Sets Bash as the default shell.

* `-g devops` → Adds the user to the devops group.

* Sets a default password (ChangeMe123) and forces a password change on first login.


## Use awk or sed in a script to process a log file and extract only error messages.


```bash
#!/bin/bash
sourceFile="/var/log/syslog"
logFile="error_logs.log"

if [ -f "${sourceFile}" ]
then
    echo "ERROR: Log file ${sourceFile} doesn't exist"
    exit 1
fi

echo "Scanning the log file for errors"
awk '/error|ERROR|Error/ {print}' "${sourceFile}" > "${logFile}"

#sed -n '/error\|ERROR\|Error/p' "${sourceFile}" > "${logFile}"
echo "ERROR logs are extracted to file ${logFile}"
```

* `-n` - With -n, only explicitly requested lines (via commands like p) are printed.
* `p` - Prints the matched lines


##  Set up a cron job that runs a script to back up (zip/tar) a directory daily.

```bash
#!/bin/bash
direct_path="var/log/files"
backup_path="/var/log/backup"
TIMESTAMP=$(date +"%d-%m-%Y")
backup_file="$backup_path/backup_${TIMESTAMP}.tar.gz"

mkdir -p ${backup_path}

tar -czf "${backup_file}" "${direct_path}"
#zip -r backup.zip "${direct_path}"
```

* `-c` - Create a new archive , `-z` - Compress the archive using gzip, `-f` - specify the name of the archive file
* `-p` - In the `mkdir` command, it ensures that parent directories are created and avoid errors if the directory exists

# Steps to schedule a cron job - 

```bash
#Open the crontab file for editing
crontab -e

#Setup Daily cron job and redirect output to a log file
* 0 * * * backup_file.sh >> /var/log/backup.log 2>&1

#Verify the cron job 
crontab -l
```

* `* * * * * script.sh`
    * Here’s what each field represents:
        * **Minute**: (0–59)
        * **Hour**: (0–23)
        * **Day of Month**: (1–31)
        * **Month**: (1–12)
        * **Day of Week**: (0–6, where Sunday = 0)