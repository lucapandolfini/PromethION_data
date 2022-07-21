# Change bonding mode for round-robin transfer load balancing
This doubles transfer speed while maintaining interface redundancy:

Edit ```/etc/network/interfaces``` and change to ```bond-mode 0```

____

# File synchronization commands for the PromethION

## Mount the SMB directory on the local directory

```bash
mount -t cifs -o username=nanopore_user //1.2.3.4/nanopore_store /home/prom/NANOPORE_STORE/
```

Alias: **sudo mount_store**

passwd   => 1st: Local
         => 2nd: Network Folder

____
 

## Synchronise data (copy on remote volume)
Use two-pass synch (rclone first, then rsync as a redundancy)
```bash
if grep nanopore /etc/mtab; 
then
  echo "DIRECTORY MOUNTED";
else
  echo "DIRECTORY UNMOUNTED. Please run mount_store first:"
  mount -t cifs -o username=nanopore_user //1.2.3.4/nanopore_store \
  /home/prom/NANOPORE_STORE/
fi
rclone copy --progress --no-traverse --size-only \
--exclude-from=/home/prom/rclone_blacklist --checkers 12 --transfers 12 \
--low-level-retries 10 --retries 5 --skip-links /data/ /home/prom/NANOPORE_STORE/
rsync -rptgoDzvhu --progress --exclude=.* \
--exclude-from=/home/prom/rsync_blacklist /data/ /home/prom/NANOPORE_STORE/
```

Alias: **sudo transfer_data**

____

## Detach SMB directory

```bash
umount /home/prom/NANOPORE_STORE
```

Alias: **sudo umount_store**

____

## KEEP SYNCHRONISED

(Useful to run it while the sequencing is in progress)

```
while true; do transfer_data; sleep 2h; done
```

Alias: **sudo keep_sync**

____

## Copy Fast5 files for basecalling

```bash
set -e
ssh namesurname@1.2.3.4 "mkdir /work/lpandolfini/$1"
rclone copy --sftp-host 1.2.3.4 --sftp-user namesurname --sftp-ask-password --checkers 12 --transfers 12 --low-level-retries 10 --retries 5 --include *.fast5 --progress /data/$1 :sftp:/work/lpandolfini/$1/
```

Alias: **energon_push <20220707_LSK_DIV0>** (absolute target name, works from any directory)

____

## Copy Basecalled folder

```bash
ssh namesurname@1.2.3.4 "rm -rf /work/lpandolfini/$1/*.log /work/lpandolfini/$1/guppy_basecaller-core-dump-db"
rclone copy --sftp-host 1.2.3.4 --sftp-user namesurname --sftp-ask-password --checkers 12 --transfers 12 --low-level-retries 10 --retries 5 --progress :sftp:/work/lpandolfini/$1 .
pigz -p 20 sequencing_summary.txt
```

Alias: **mkdir destination_dir && cd destination_dir&& energon_pull <name_basecalled_folder>**

____

## Mount a read-only user

```bash
sudo mount -t cifs -o username=namesurname,domain=IIT.local //1.2.3.4/nanopore_store win_share
```
