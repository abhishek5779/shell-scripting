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

* `dpkh -l` - List the packages concisely.

## Create a script that monitors CPU and memory usage every 5 seconds and logs the results to a file.

```bash

```