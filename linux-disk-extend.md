You’re already using ext4 on /dev/vda1 👍
So you don’t need to change filesystem — just expand it to full 250GB.

Right now:

Disk: vda → 250G
Partition: vda1 → 50G
Filesystem: ext4 ✅
✅ Do this (safe & correct way)
1. Install growpart (if not installed)
apt update
apt install cloud-guest-utils -y
2. Expand partition to full disk
growpart /dev/vda 1

👉 This will increase vda1 from 50G → 250G

3. Resize ext4 filesystem
resize2fs /dev/vda1

👉 This expands ext4 to use full partition

4. Verify
df -h

Expected:

/dev/vda1   ~250G
🔥 Important Notes
✅ No reboot needed
✅ No data loss (safe operation)
✅ Docker overlay will auto use new space
❗ If growpart says "NOCHANGE"

Then run:

partprobe

and retry:

growpart /dev/vda 1
💡 Summary (very simple)

You already have:

ext4 ✔
250GB disk ✔

You just need:

Expand partition → growpart
Expand filesystem → resize2fs