+++
title = "Replicating Sysvol from Windows to Linux"
+++

```bash
#!/bin/bash
if [ $EUID -ne 0 ]; then
   echo "This script must be run as root"
   exit 1
fi
# Dynamically get domain, machine, and PDC Emulator info
REALM=$(samba-tool domain info 127.0.0.1 | grep "Domain" | awk '{print $3}')
MACHINE_SAM=$(samba-tool computer list | grep -i "$(hostname -s)" | grep "\$")
MACHINE_PRINCIPAL="${MACHINE_SAM}@${REALM^^}"
PDC_DC=$(samba-tool fsmo show | grep "PdcEmulationMasterRole owner" | awk -F',' '{print $2}' | awk -F'=' '{print $2}')
SOURCE_SHARE="//${PDC_DC}/sysvol"

# Local Configuration
MOUNT_POINT="/mnt/sysvol_sync"
LOCAL_SYSVOL="/var/lib/samba/sysvol/"
KEYTAB="/etc/krb5.keytab"

# Export only the required machine principal
samba-tool domain exportkeytab "${KEYTAB}" --principal="${MACHINE_SAM}"

# Authenticate - This now uses the account the KDC recognizes as a client
kinit -k -t "${KEYTAB}" "${MACHINE_PRINCIPAL}"

if [ $? -ne 0 ]; then
    echo "Error: Failed to obtain Kerberos ticket."
    exit 1
fi

# Mount the remote PDC sysvol
mkdir -p "${MOUNT_POINT}"
mount -t cifs "${SOURCE_SHARE}" "${MOUNT_POINT}" -o sec=krb5,ro

if [ $? -ne 0 ]; then
    echo "Error: Failed to mount remote SysVol from ${PDC_DC}."
    exit 1
fi

# Sync and Fix ACLs
rsync -XAavz --delete-after --exclude DfsrPrivate "${MOUNT_POINT}/" "${LOCAL_SYSVOL}"
umount "${MOUNT_POINT}"
samba-tool ntacl sysvolreset
```