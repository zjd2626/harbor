## Guild for update Clair offline data
### Summary

In some case when user install harbor in an environment without internet access. Then Clair will not be able to fetch the latest vulnerability database. In this circumstance user need manfully update the Clair database.

This document is a step by step instruction on update Clair vulnerability database in Harbor v1.2.

### Preparation

A. User need to install Clair 2.0.1 ( if you have a harbor1.2 instance with internet access will also works.)

B. Check the Clair already update the vulnerability to the latest.

    a. 'docker ps' to list the Clair container. Get the Clair container id .

    b. Check the log of the Clair container.

    c. If you are using harbor you can find the latest Clair log under /var/log/harbor/2017--xx-xx/clair.log

    d. You will find some logs like follow:
    ```
    Jul 3 20:40:45 172.18.0.1 clair[3516]: {"Event":"finished fetching","Level":"info","Location":"updater.go:227","Time":"2017-07-04 03:40:45.890364","updater name":"rhel"}
    Jul 3 20:40:46 172.18.0.1 clair[3516]: {"Event":"finished fetching","Level":"info","Location":"updater.go:227","Time":"2017-07-04 03:40:46.768924","updater name":"alpine"}
    Jul 3 20:40:47 172.18.0.1 clair[3516]: {"Event":"finished fetching","Level":"info","Location":"updater.go:227","Time":"2017-07-04 03:40:47.190982","updater name":"oracle"}
    Jul 3 20:41:07 172.18.0.1 clair[3516]: {"Event":"Debian buster is not mapped to any version number (eg. Jessie-\u003e8). Please update me.","Level":"warning","Location":"debian.go:128","Time":"2017-07-04 03:41:07.833720"}
    Jul 3 20:41:07 172.18.0.1 clair[3516]: {"Event":"finished fetching","Level":"info","Location":"updater.go:227","Time":"2017-07-04 03:41:07.833975","updater name":"debian"}
    Jul 4 00:26:17 172.18.0.1 clair[3516]: {"Event":"finished fetching","Level":"info","Location":"updater.go:227","Time":"2017-07-04 07:26:17.596986","updater name":"ubuntu"}
    Jul 4 00:26:18 172.18.0.1 clair[3516]: {"Event":"adding metadata to vulnerabilities","Level":"info","Location":"updater.go:253","Time":"2017-07-04 07:26:18.060810"}
    Jul 4 00:38:05 172.18.0.1 clair[3516]: {"Event":"update finished","Level":"info","Location":"updater.go:198","Time":"2017-07-04 07:38:05.251580"}
    ```
    e. The update finished indicate that Clair has finished an vulnerability update round. You need to check that logs above it to make sure all the endpoints are update correctly.

### Data dump

A. Login into the host where Clair db is running.

B. Dump the Clair vulnerability database by the follow command
* $>docker exec clair-db /bin/bash -c  "pg_dump -U postgres -a -t feature -t keyvalue -t namespace -t schema_migrations -t vulnerability -t vulnerability_fixedin_feature" > vulnerability.sql
* $>docker exec clair-db /bin/bash -c "pg_dump -U postgres -c -s" > clear.sql


### Back Up Clair DB
A. Before update the offline data, user are strongly suggested to backup their Clair db.
* docker exec clair-db /bin/bash -c  "pg_dump -U postgres -c" > all.sql

### Update Clair DB
A. Copy the vulnerability.sql and clear.sql to the host where clair-db container which you want to update is running on.


C. $>docker exec -i clair-db psql -U postgres < clear.sql

D. $>docker exec -i clair-db psql –U postgres < vulnerability.sql

### Rescan
After update the offline data, user need to trigger the "rescan all" functionality to scan all the images and Harbor reflect the new changes automatically after the scan finished.(Otherwise the vulnerability detail will not show up) 
