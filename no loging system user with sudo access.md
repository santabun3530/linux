নিশ্চিত! নিচে তোমার জন্য পুরো বিষয়টা নিয়ে একটা সুন্দরভাবে গুছানো **`README.md`** ফাইল বানিয়ে দিলাম, যা তুমি ডকুমেন্টেশন হিসেবে বা নিজের প্রোজেক্টে ব্যবহার করতে পারো।

---

## 📄 `README.md`

````markdown
# 🛠️ Scheduled Service Restart using Non-Login System User with Sudo Access

This setup allows a non-login system user (e.g., `sony`) to run a script that restarts a systemd service (`aa.server`) every 10 minutes using `cron` — without requiring a password.

---

## 📌 Objective

- Use a system (non-login) user to run a script.
- The script runs every 10 minutes via `cron`.
- The script restarts a systemd service using `sudo` **without** prompting for a password.

---

## 🧾 Step-by-Step Guide

### 1. ✅ Create a system user (if not already created)

```bash
sudo useradd -r -s /sbin/nologin sony
````

* `-r`: Creates a system user
* `-s /sbin/nologin`: Prevents shell login

---

### 2. ✅ Allow `sony` to run systemctl restart via `sudo` (no password)

Edit the sudoers file safely:

```bash
sudo visudo
```

Add the following line at the bottom:

```bash
sony ALL=(ALL) NOPASSWD: /bin/systemctl restart aa.server
```

> You can confirm the path to `systemctl` by running:
>
> ```bash
> which systemctl
> ```

---

### 3. ✅ Create the restart script

Create a file called `restart_service.sh` and add the following:

```bash
#!/bin/bash

sudo /bin/systemctl restart aa.server
```

Make it executable:

```bash
chmod +x /path/to/restart_service.sh
```

---

### 4. ✅ Schedule it with cron (as `sony` user)

Edit the crontab for `sony`:

```bash
sudo crontab -u sony -e
```

Add this line to run the script every 10 minutes:

```cron
*/10 * * * * /path/to/restart_service.sh
```

---

## 🔐 Security Notes

* Only the specific command (`systemctl restart aa.server`) is allowed with `sudo`.
* The user has **no shell access**, preventing manual login.
* This setup minimizes risk while allowing automated control over services.

---

## ✅ Summary

| Feature               | Implemented                 |
| --------------------- | --------------------------- |
| Non-login user        | ✅ `-s /sbin/nologin`        |
| Scheduled restart     | ✅ via `cron`                |
| Root-level permission | ✅ via limited `sudo` access |
| Secure and automated  | ✅                           |

---

## 🧠 Tip

You can apply this method to any service/script that needs `sudo`, while keeping your system secure from unnecessary user access.

```

---

## 📦 Bonus

তুমি চাইলে আমি এটা `.zip` আকারে ডেলিভার করতে পারি অথবা তোমার পছন্দমতো format এ সাজিয়ে দিতেও পারি — markdown, PDF, বা shell script bundle করে।

বললেই করে দিচ্ছি। ✅
```
