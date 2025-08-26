# Upgrading Postgresql databases from cluster 13 to cluster 17
## Install PostgreSQL 17 alongside 13

    echo "deb http://apt.postgresql.org/pub/repos/apt bookworm-pgdg main" | \
    sudo tee /etc/apt/sources.list.d/pgdg.list

    sudo apt update
    sudo apt install postgresql-17

Ensure the two clusters are available and active

    pg_lsclusters

Expect output:

    Ver Cluster Port Status Owner     Data directory              Log file
    13  main    5434 online <unknown> /var/lib/postgresql/13/main /var/log/postgresql/postgresql-13-main.log
    15  main    5432 down   postgres  /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
    17  main    5433 online postgres  /var/lib/postgresql/17/main /var/log/postgresql/postgresql-17-main.log    

### Change authentication to trust on postgresql@13-main/pg_hba.conf to avoid password prompt. Remember to change it back once done.


    # Database administrative login by Unix domain socket
    local   all             postgres                                trust


### Stop both clusters

    sudo systemctl stop postgresql@13-main
    sudo systemctl stop postgresql@17-main
 

### Navigate to directory where user postgres has read and write permissions

    cd /var/lib/postgresql

### Run the upgrade using the command below.

    sudo -u postgres pg_upgrade \
    -b /usr/lib/postgresql/13/bin \
    -B /usr/lib/postgresql/17/bin \
    -d /var/lib/postgresql/13/main \
    -D /var/lib/postgresql/17/main \
    -U postgres

Expected output snippet.

    Performing Consistency Checks
    -----------------------------
    Checking cluster versions                                     ok
    Checking database user is the install user                    ok
    Checking database connection settings                         ok
    Checking for prepared transactions                            ok
    Checking for contrib/isn with bigint-passing mismatch         ok
    Checking data type usage                                      ok
    Checking for user-defined encoding conversions                ok
    Checking for user-defined postfix operators                   ok
    Checking for incompatible polymorphic functions               ok
    Checking for not-null constraint inconsistencies              ok
    Creating dump of global objects                               ok
    Creating dump of database schemas                             
                                                              ok
    Checking for presence of required libraries                   ok
    Checking database user is the install user                    ok
    Checking for prepared transactions                            ok
    Checking for new cluster tablespace directories               ok

    If pg_upgrade fails after this point, you must re-initdb the
    new cluster before continuing.

    Performing Upgrade
    ------------------
    Setting locale and encoding for new cluster                   ok
    Analyzing all rows in the new cluster                         ok
    Freezing all rows in the new cluster                          ok
    Deleting files from new pg_xact                               ok
    Copying old pg_xact to new server                             ok
    Setting oldest XID for new cluster                            ok
    Setting next transaction ID and epoch for new cluster         ok
    Deleting files from new pg_multixact/offsets                  ok
    Restoring global objects in the new cluster                   ok
    Restoring database schemas in the new cluster                 
                                                                ok
    Copying user relation files                                   
                                                                ok
    Setting next OID for new cluster                              ok
    Sync data directory to disk                                   ok
    Creating script to delete old cluster                         ok
    Checking for extension updates                                ok

    Upgrade Complete