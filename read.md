বুঝতে পারছি – তুমি আসলে পুরো ছবিটা একসাথে connect করতে চাইছো — অর্থাৎ **user–group–permission** কিভাবে একে অপরের সাথে লিঙ্কড থাকে।
চলো step by step ডট কানেক্ট করি 🔗

---

## 1️⃣ User তৈরি করলে কি হয়?

👉 তুমি যখন `sony` নামে user বানালে:

* একটা **user account** তৈরি হলো → `sony (UID=1001)`
* সাথে সাথে একটা **primary group** তৈরি হলো → `sony (GID=1001)`
* মানে `sony` user by default `sony` group-এর member।
* এই primary group শুধু user-এর default ownership set করতে কাজে লাগে (যেমন নতুন ফাইল বানালে, সেই ফাইলের group হবে `sony`)।

🔹 এজন্য তুমি দেখছো:

```
sony:x:1001:
```

---

## 2️⃣ Secondary groups

Primary group শুধু default ownership-এর জন্য। কিন্তু **special কাজ করার জন্য user কে secondary group এ ঢুকাতে হয়**।

উদাহরণ:

* তুমি `sony` কে `sudo` group এ দিলে → সে এখন admin permission পেতে পারে।
* তুমি `sony` কে `docker` group এ দিলে → সে এখন docker চালাতে পারবে `sudo` ছাড়া।

🔹 এজন্য `/etc/group` এ এভাবে দেখাচ্ছে:

```
sudo:x:27:sony,vagrant
docker:x:999:vagrant,sony
```

মানে `sony` → **secondary member** of `sudo` আর `docker`।

---

## 3️⃣ Root group vs Sudo group

* **root user (UID=0, GID=0)** → Linux-এর superuser। সে সবকিছু করতে পারে। তার primary group হলো `root`।
* **sudo group** → সাধারণ user দেরকে অস্থায়ীভাবে root permission দেওয়ার shortcut।

  * `sony` sudo group এ থাকলে → `sony` root না, কিন্তু `sudo command` দিলে root-এর মতো কাজ করতে পারবে।
  * root আলাদা user, sudo group হলো অন্য user-দেরকে root এর privilege দেওয়া।

👉 এজন্য root আলাদা থাকে, আর sudo গ্রুপ root এর shortcut access।

---

## 4️⃣ Software-specific groups

সব software group বানায় না। কিছু software কেন group বানায়?
কারণ তারা **system resources** এ access control করতে চায়।

* **Docker** → root ছাড়াই socket access দরকার। তাই `docker` group বানায়।
* **KVM / libvirt** → virtualization এর জন্য `kvm`, `libvirt` group বানায়।
* **Systemd services** (`systemd-journal`, `systemd-resolve`) → নিজেদের process গুলোকে আলাদা group এ রাখে যাতে limited access control হয়।

অন্য software যেগুলোর আলাদা permission দরকার নেই (যেমন `vim`, `curl`) তারা group বানায় না।

---

## 5️⃣ এখন সব ডট connect করলে ছবিটা এরকম 🔗

* **User add করলে** → একই নামে **primary group** add হয়।
* **User কে secondary group এ দিলে** → user extra permission পায় (যেমন `sudo`, `docker`)।
* **Root** → সবকিছুর মালিক (superuser)।
* **Sudo group** → root এর কাজ করার shortcut দেয় normal user কে।
* **System groups** → system services এবং কিছু সফটওয়্যার নিজেদের কাজের জন্য group বানায়।
* **সব group = শুধুই permission management এর container**।

---

## 🔹 Practical mapping from তোমার `/etc/group`

* `sony:x:1001:` → **Primary group** (default for sony)
* `sudo:x:27:sony,vagrant` → **Secondary group** → admin power
* `docker:x:999:vagrant,sony` → **Secondary group** → docker চালাতে দেয়
* `systemd-journal`, `syslog`, `messagebus` → **System groups** → services এর জন্য
* `root:x:0:` → **Root group** (UID=0, GID=0, সবকিছু করতে পারে)

---

👉 এখন একটা example scenario:

* `sony` একটা ফাইল বানালে → ফাইলের owner হবে `(sony, sony)`
* `sony` চাইলে system update করতে → `sudo apt update` (কারণ sudo group এ আছে)
* `sony` চাইলে docker চালাতে → `docker run …` (কারণ docker group এ আছে)

---

এখন বলো — তুমি কি চাইছো আমি diagram style (ASCII বা table আকারে) একটা full map এঁকে দিই user–group–permission relationship?
