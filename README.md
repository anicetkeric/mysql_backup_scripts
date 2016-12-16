# mysql_backup_scripts
Bash script to backup MySQL databases for Linux
    
    
# Instructions


1. Use the script **mysql_dump.sh** 
2. Execute script - sudo ./mysql_dump.sh   
3. Setup a CRON job (see CRON below) 

## Show script mysql_dump.sh





***
     "dump" is the name of the MySql user
     "password" is the dump user's password

## Execute script
Make the file executable

    chmod +x mysql_dump.sh

and start with

     ./mysql_dump.sh
     
## CRON
The cron is simple, just schedule it once per day for example.

    $crontab -e
    0 0 * * * /home/root/mysql_dump.sh
