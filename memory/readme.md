# Adding Swap Memory to Debian System
## Step 1 – Checking the System for Swap Information
Check your system for swap info using the command below

    sudo swapon --show

If you dont get back any output it means there is no swap space available in your system.

The `free` utility can verify this.

    free -h


    Output
                    total        used        free      shared  buff/cache   available
    Mem:           976Mi        75Mi       623Mi       0.0Ki       276Mi       765Mi
    Swap:             0B          0B          0B

## Step 2 – Checking Available Space on the Hard Drive Partition
Check your disk usage to ensure you have enough disk spqce for the swap memory.

    df -h

    Output
    Filesystem      Size  Used Avail Use% Mounted on
    udev            472M     0  472M   0% /dev
    tmpfs            98M  500K   98M   1% /run
    /dev/vda1        25G  1.1G   23G   5% /
    tmpfs           489M     0  489M   0% /dev/shm
    tmpfs           5.0M     0  5.0M   0% /run/lock
    /dev/vda15      124M  5.9M  118M   5% /boot/efi
    tmpfs            98M     0   98M   0% /run/user/0

The device with `/` in the Mounted on column is our disk in this case. 

## Step 3 – Creating a Swap File

