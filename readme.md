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

    host replication replicator 10.7.22.18/32 md5

Reload the PostgreSQL configuration to apply the changes:

    SELECT pg_reload_conf();

### Creating a Publication (On Primary Server)

    CREATE PUBLICATION my_publication FOR ALL TABLES;

### Setting Up the Subscriber Server
On the subscriber server, you’ll create a subscription to receive data from the publication created on the primary server.

    sudo -u postgres psql

    \c replica_database

    CREATE SUBSCRIPTION my_subscription
    CONNECTION 'host=10.7.22.17 port=5432 dbname=mydb user=replicator password=replicator_password'
    PUBLICATION my_publication;

###  Managing Subscriptions
Disable a subscription:

    ALTER SUBSCRIPTION my_subscription DISABLE;

### Re-enable a subscription:

    ALTER SUBSCRIPTION my_subscription ENABLE;

### Monitoring Replication Status
Check the replication status on the subscriber server:

    SELECT * FROM pg_stat_subscription;
    



