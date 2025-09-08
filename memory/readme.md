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
We will allocate a file of the size that we want called swapfile in our root (/) directory.
The best way of creating a swap file is with the fallocate program.

    sudo fallocate -l 1G /swapfile

Verify that the correct amount of space was reserved by typing:

    ls -lh /swapfile

    Output
    -rw-r--r-- 1 root root 1.0G Aug 23 11:14 /swapfile

## Step 4 – Enabling the Swap File
Lock down the permissions of the file so that only users with root privileges can read the contents. This prevents normal users from being able to access the file, which would have significant security implications.

    sudo chmod 600 /swapfile

Verify the permissions.

    ls -lh /swapfile

    Output
    -rw------- 1 root root 1.0G Aug 23 11:14 /swapfile

Mark the file as swap space.

    sudo mkswap /swapfile

    Output
    Setting up swapspace version 1, size = 1024 MiB (1073737728 bytes)
    no label, UUID=6e965805-2ab9-450f-aed6-577e74089dbf

Enable the swap file.

    sudo swapon /swapfile

Verify that the swap is available.

    sudo swapon --show

    Output
    NAME      TYPE  SIZE USED PRIO
    /swapfile file 1024M   0B   -2

## Step 5 – Making the Swap File Permanent
Our recent changes have enabled the swap file for the current session. 
We can make this permanent by adding the swap file to our `/etc/fstab` file.

Back up the `/etc/fstab` file in case anything goes wrong.

    sudo cp /etc/fstab /etc/fstab.bak

Add the swap file information to the end of your `/etc/fstab`.

    echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab