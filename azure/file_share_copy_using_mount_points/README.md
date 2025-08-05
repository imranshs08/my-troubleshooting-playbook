
````markdown
# Copy Files Between Two Azure File Shares in the Same Storage Account using (RHEL VM)

This guide explains how to copy data from File Share A (`file-share-a`) to File Share B (`file-share-b`) within the same Azure Storage Account by mounting both shares on a RHEL VM.
````

## Prerequisites

- A RHEL VM with root access.
- `cifs-utils` package installed.
- Access to the Azure Storage Account keys.
- Two existing Azure File Shares:  
  - **Source:** `file-share-a`  
  - **Destination:** `file-share-b`  

## Step 1: Install Required Packages

```bash
sudo yum install -y cifs-utils
````

## Step 2: Get Storage Account Key

Login to Azure CLI and fetch the storage account key:

```bash
az login
az storage account keys list \
  --resource-group <YourResourceGroup> \
  --account-name <YourStorageAccountName> \
  --query "[0].value" -o tsv
```

Copy the key for use in the next step.


## Step 3: Create Mount Directories

```bash
sudo mkdir -p /mnt/file-share-a
sudo mkdir -p /mnt/file-share-b
```


## Step 4: Mount the File Shares

Replace `<YourStorageAccountName>` and `<StorageAccountKey>` with your values.

```bash
sudo mount -t cifs //<YourStorageAccountName>.file.core.windows.net/file-share-a /mnt/file-share-a \
   -o vers=3.0,username=<YourStorageAccountName>,password=<StorageAccountKey>,dir_mode=0777,file_mode=0777,serverino

sudo mount -t cifs //<YourStorageAccountName>.file.core.windows.net/file-share-b /mnt/file-share-b \
   -o vers=3.0,username=<YourStorageAccountName>,password=<StorageAccountKey>,dir_mode=0777,file_mode=0777,serverino
```


## Step 5: Verify Mounts

```bash
df -h | grep file.core.windows.net
```

You should see both `file-share-a` and `file-share-b` mounted.

---

## Step 6: Copy Data from Share A to Share B

For **preserving timestamps and permissions**:

```bash
sudo rsync -av /mnt/file-share-a/ /mnt/file-share-b/
```

Or use a simpler copy command:

```bash
sudo cp -rvi /mnt/file-share-a/. /mnt/file-share-b/
```

---

## Step 7: Validate the Copy

```bash
ls -l /mnt/file-share-b
```

Check that all files from `file-share-a` are present in `file-share-b`.

---

## Step 8: Unmount After Copy

```bash
sudo umount /mnt/file-share-a
sudo umount /mnt/file-share-b
```

---

## Notes

* If this is a **one-time copy**, unmount after completion.
* If you want the shares to auto-mount after reboot, add entries to `/etc/fstab`.

---

✅ You have successfully copied all files from **file-share-a** to **file-share-b** on a RHEL VM.


