## Logical Replication

### PostgreSQL Configuration (On Primary Server)
On the primary server, add the following settings to the postgresql.conf file:


    wal_level = logical
    max_replication_slots = 4
    max_wal_senders = 10

Restart the PostgreSQL server:

    systemctl restart postgresql

###  Creating a Replication User (On Primary Server)

    CREATE USER replicator WITH REPLICATION PASSWORD 'replicator_password';

### Grant Necessary Permissions

    sudo -u postgres psql

    \c primary_database

    GRANT USAGE ON SCHEMA public TO replicator;
    GRANT SELECT ON ALL TABLES IN SCHEMA public TO replicator;

###  Configuring pg_hba.conf (On Primary Server)
In the pg_hba.conf file, add the following entry to allow the subscriber server to connect using the replication user:

    host replication replicator <subscriber_ip>/32 peer

Reload the PostgreSQL configuration to apply the changes:

    sudo systemctl restart postresql@13-main

### Creating a Publication (On Primary Server)

    sudo -u postgres psql
    \c <primary_database>
    CREATE PUBLICATION my_publication FOR ALL TABLES;

### Ensure all tables without a PRIMARY KEY have a REPLICA IDENTITY

    DO $$
    DECLARE
        r RECORD;
    BEGIN
        FOR r IN
            SELECT c.relname AS table_name
            FROM pg_class c
            JOIN pg_namespace n ON n.oid = c.relnamespace
            LEFT JOIN (
                SELECT conrelid
                FROM pg_constraint
                WHERE contype = 'p'
            ) pk ON pk.conrelid = c.oid
            WHERE c.relkind = 'r'
              AND n.nspname = 'public'
              AND pk.conrelid IS NULL
        LOOP
            EXECUTE 'ALTER TABLE ' || quote_ident(r.table_name) || ' REPLICA IDENTITY FULL;';
        END LOOP;
    END;
    $$;

    

### Setting Up the Subscriber Server
Create a database with only the schema of the primary server. Create a dump on the primary server and download it to the local.

    ssh username@primary-server-domain
    pg_dump -U your_username -d your_database -F c -f your_database.dump

On your local pc download the your_database.dump file and create a database with the schema only from the your_database.dump.

    scp username@primary-server-domain /tmp
    pg_restore --schema-only -U postgres_user -d replica_database ~/tmp/your_database.dump

create a ssh tunnel to connect from your local to the database on the primary on the primary port.

    ssh -f -N -L 5433:localhost:5432 username@primary-ip

    #5433 - free port on local
    #5432 - postgres port on primary server
    
On the subscriber server, you’ll create a subscription to receive data from the publication created on the primary server.

    sudo -u postgres psql

    \c replica_database

    CREATE SUBSCRIPTION my_subscription
    CONNECTION 'host=localhost port=5433 dbname=primary_db user=replicator password=replicator_password'
    PUBLICATION my_publication;

###  Managing Subscriptions
Disable a subscription:

    ALTER SUBSCRIPTION my_subscription DISABLE;

### Re-enable a subscription:

    ALTER SUBSCRIPTION my_subscription ENABLE;

### Monitoring Replication Status
Check the replication status on the subscriber server:

    SELECT * FROM pg_stat_subscription;
    



