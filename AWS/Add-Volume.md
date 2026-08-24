## Check before mount

1. `lsblk` if the ebs volume is attached to the ec2
1.1 `lsblk -f` to display the type of disk, ex: `ext4`
2. `df -h` displays only **mounted** volumes

## ADD Volume

1. Let newly created volume supposed to be in the **same AZ(Availability Zone)**.
2. 
   ```bash
   lsblk  
   sudo mkfs -t ext4 /dev/nvme1n1  
   sudo mkdir /data #can be anything  
   sudo mount /dev/nvme1n1 /data
   ```

- everything disk is listed under `/dev` in linux

## Unmount Volume

```bash
sudo umount -l /data
```
- `-l`: lazy unmount until everything is used.