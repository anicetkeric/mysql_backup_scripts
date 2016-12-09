# mysql_backup_scripts
Bash script to backup MySQL databases for Linux
    
    
# Instructions


1. Use the script **mysql_dump.sh** 
2. Execute script - sudo ./mysql_dump.sh   
3. Setup a CRON job (see CRON below) 

## Show script mysql_dump.sh

    #!/bin/bash 

    #ANICET ERIC KOUAME 

    set -eu

    #Create a MySQL user with restricted reading rights on all databases
    mysql -u root -p -e "CREATE USER 'dump'@'localhost' IDENTIFIED BY 'password';";
    mysql -u root -p -e "GRANT SELECT, SHOW DATABASES, LOCK TABLES, SHOW VIEW ON *. * TO 'dump'@'localhost' IDENTIFIED BY 'password' WITH MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0 ;";

    ## parameter
    USER='dump'
    PASS='password' 
    # Backup storage directory
    DATADIR="/var/backups/mysql"
    # Working directory 
    DATATMP=$DATADIR
    # dump Name
    DATANAME="dump_$(date +%d.%m.%y@%Hh%M)"
    # Compression
    COMPRESSIONCMD="tar -czf" 
    COMPRESSIONEXT=".tar.gz"
    # rotation backups
    RETENTION=30
    # Exclude mysql databases
    EXCLUSIONS='(information_schema|performance_schema|mysql)'
    # Email for errors (0 to disable)
    EMAIL=0


    ## Start script

    ionice -c3 -p$$ &>/dev/null
    renice -n 19 -p $$ &>/dev/null

    function cleanup {
        if [ "`stat --format %s ${DATATMP}/error.log`" != "0" ] && [ "$EMAIL" != "0" ] ; then
            cat ${DATATMP}/error.log | mail -s "Backup MySQL $DATANAME - Log error" ${EMAIL}
        fi
    }
    trap cleanup EXIT

    # A temporary directory
    mkdir -p ${DATATMP}/${DATANAME}
    # Log 
    exec 2> ${DATATMP}/error.log
    databases="$(mysql -u $USER -p$PASS -Bse 'show databases' | grep -v -E $EXCLUSIONS)"

    # For each database found ...
    for database in ${databases[@]} 
    do
        echo "dump : $database"
        mysqldump -u $USER -p$PASS --quick --add-locks --lock-tables --extended-insert $database  > ${DATATMP}/${DATANAME}/${database}.sql
    done 

    # Compress all databases
    cd ${DATATMP}
    ${COMPRESSIONCMD} ${DATANAME}${COMPRESSIONEXT} ${DATANAME}/
    chmod 600 ${DATANAME}${COMPRESSIONEXT}

    # Moves to the directory
    if [ "$DATATMP" != "$DATADIR" ] ; then
        mv ${DATANAME}${COMPRESSIONEXT} ${DATADIR}
    fi

    # symlink to the latest release
    cd ${DATADIR}
    set +eu
    unlink last${COMPRESSIONEXT}
    set -eu
    ln ${DATANAME}${COMPRESSIONEXT} last${COMPRESSIONEXT}

    # Removing the temporary directory 
    rm -rf ${DATATMP}/${DATANAME}

    echo "Suppression des vieux backup : "
    find ${DATADIR} -name "*${COMPRESSIONEXT}" -mtime +${RETENTION} -print -exec rm {} \\;


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
