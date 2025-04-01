![mongoDB](https://upload.wikimedia.org/wikipedia/en/5/5a/MongoDB_Fores-Green.svg)

## **Ubuntu VPS-এ MongoDB Setup, User Management, এবং Firewall Configuration (Step-by-Step)**

---


এই গাইডে আমি বিস্তারিতভাবে Ubuntu VPS-এ **MongoDB ইনস্টল, ইউজার তৈরি, ডাটাবেস তৈরি, ইউজার ও ডাটাবেস ম্যানেজমেন্ট, Firewall কনফিগারেশন, IP Access Control এবং Automatic Backup** সম্পর্কে বলব।

---

## **Step 1: MongoDB Install করা (Locally)**

MongoDB ইনস্টল করার জন্য নিচের কমান্ডগুলো রান করুন:

### **1. MongoDB-এর Official GPG Key Add করুন**

MongoDB-এর প্যাকেজ ভেরিফাই করার জন্য এর GPG key যুক্ত করতে হবে।

```bash
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | sudo gpg --dearmor -o /usr/share/keyrings/mongodb-server-7.0.gpg
```

### **2. MongoDB-এর Official Repository যোগ করুন**

```bash
echo "deb [signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg] https://repo.mongodb.org/apt/ubuntu $(lsb_release -cs)/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```

### **3. System Update করুন**

```bash
sudo apt update
```

### **যদি তুমি oracular (Ubuntu 24.04 LTS) ব্যবহার করে থাকো তাহলে এটি এখনো অফিসিয়ালভাবে MongoDB 7.0 সমর্থন করছে না, তুমি আগের Ubuntu 22.04 (jammy) কোডনেম ব্যবহার করতে পারো:**

```bash
echo "deb [signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```

#### **System Update করুন**

```bash
sudo apt update
```

### **4. MongoDB Install করুন**

```bash
sudo apt install -y mongodb-org
```

### **5. MongoDB Service চালু করুন**

```bash
sudo systemctl start mongod
```

### **6. MongoDB সার্ভিস চালু থাকবে নিশ্চিত করুন**

```bash
sudo systemctl enable mongod
```

### **7. MongoDB চালু হয়েছে কিনা চেক করুন**

```bash
sudo systemctl status mongod
```

যদি সব ঠিক থাকে, তাহলে নিচের মতো মেসেজ দেখাবে:

```
● mongod.service - MongoDB Database Server
     Loaded: loaded (/lib/systemd/system/mongod.service; enabled; vendor preset: enabled)
     Active: active (running) since...
```

এই স্ট্যাটাস পেতে **q চাপলে** টার্মিনাল থেকে বেরিয়ে আসবে।

---

## **Step 2: MongoDB Shell-এ ঢোকা**

MongoDB Shell চালু করতে:

```bash
mongosh
```

এতে MongoDB-এর ইন্টারেক্টিভ কনসোল চালু হবে।

---

## **Step 3: নতুন Database তৈরি করা**

MongoDB তে নতুন ডাটাবেস তৈরি করতে:

```javascript
use myDatabase
```

এটি `myDatabase` নামে একটি নতুন ডাটাবেস তৈরি করবে। তবে এটি তখনই সংরক্ষিত হবে যখন আপনি এতে ডাটা যুক্ত করবেন।

ডাটাবেসে একটি কালেকশন (টেবিলের মতো) তৈরি করুন:

```javascript
db.myCollection.insertOne({ name: "Test Data" });
```

ডাটাবেস লিস্ট দেখতে:

```javascript
show dbs
```

---

## **Step 4: নতুন User তৈরি করা ও ডাটাবেসের সাথে সংযুক্ত করা**

```javascript
use admin
db.createUser({
  user: "myUser",
  pwd: "myPassword",
  roles: [{ role: "readWrite", db: "myDatabase" }]
});
```

**ইউজার তৈরি হওয়ার পর চেক করুন:**

```javascript
show users
```

---

## **Step 5: ইউজারের পাসওয়ার্ড পরিবর্তন করা**

```javascript
use admin
db.updateUser("myUser", { pwd: "newSecurePassword" });
```

---

## **Step 6: User মুছে ফেলা (Delete)**

```javascript
use admin
db.dropUser("myUser")
```

---

## **Step 7: Database Rename করা (MongoDB-তে সরাসরি Rename অপশন নেই)**

MongoDB-তে ডাটাবেস Rename করার কোনো সরাসরি কমান্ড নেই। তবে, **নতুন ডাটাবেস তৈরি করে পুরানো ডাটাবেস থেকে ডাটা কপি করে, পরে পুরানোটি ডিলিট করতে হয়।**

**Database Rename করার পদ্ধতি:**

```bash
mongodump --db oldDatabase --out /backup/
mongorestore --db newDatabase /backup/oldDatabase/
```

---

## **Step 8: Database Delete করা**

MongoDB-তে কোনো ডাটাবেস **ডিলিট করার আগে সেটিতে ঢুকতে হবে**:

```javascript
use myDatabase
db.dropDatabase()
```

ডাটাবেস ডিলিট হয়ে গেলে `show dbs` কমান্ড দিলে আর এটি দেখা যাবে না।

---

## **Step 9: MongoDB Restart, Stop ও Reload করা**

```bash
# MongoDB Restart
sudo systemctl restart mongod

# MongoDB Stop
sudo systemctl stop mongod

# MongoDB Start
sudo systemctl start mongod
```

---

## **সবকিছু ঠিকভাবে কাজ করছে কিনা চেক করা**

```bash
mongosh
```

তারপর লগইন করে চেক করুন:

```javascript
use myDatabase
show collections
db.myCollection.find()
```

---

## **Step 10: নির্দিষ্ট IP বা সকল IP থেকে Access দেওয়া (Firewall + Bind IP)**

MongoDB ডিফল্টভাবে শুধু লোকালহোস্ট (`127.0.0.1`) থেকে কানেক্ট হতে পারে। VPS-এর বাইরের কোনো সার্ভার থেকে অ্যাক্সেস দিতে হলে **bindIP পরিবর্তন ও firewall কনফিগার করতে হবে**।

### **1. MongoDB Config পরিবর্তন করা (Bind IP সেট করা)**

ফাইল ওপেন করুন:

```bash
sudo nano /etc/mongod.conf
```

নিচের অংশটি খুঁজুন:

```yaml
# network interfaces
net:
  bindIp: 127.0.0.1
```

**নির্দিষ্ট IP থেকে Access অনুমতি দিতে (যেমন: `192.168.1.100`)**

```yaml
net:
  bindIp: 127.0.0.1,192.168.1.100
```

**সকল IP থেকে Access দিতে (⚠️ সিকিউরিটির জন্য ঝুঁকিপূর্ণ)**

```yaml
net:
  bindIp: 0.0.0.0
```

পরিবর্তন সংরক্ষণ করতে:
👉 **CTRL + X → Y → ENTER**

### **2. Firewall Rule যোগ করা**

**নির্দিষ্ট IP অনুমতি দিতে (192.168.1.100)**

```bash
sudo ufw allow from 192.168.1.100 to any port 27017
```

**সকল IP কে Access দিতে (⚠️ নিরাপদ নয়)**

```bash
sudo ufw allow 27017/tcp
```

---

## **Step 11: Firewall Enable ও Status চেক করা**

```bash
sudo ufw enable
sudo ufw status
```

MongoDB Firewall rule চেক করতে:

```bash
sudo ufw status numbered
```

---

## **সংক্ষেপে গুরুত্বপূর্ণ কমান্ডগুলো**

| কাজ                 | কমান্ড                                                                          |
| ------------------- | ------------------------------------------------------------------------------- |
| MongoDB Install     | `sudo apt install -y mongodb-org`                                               |
| MongoDB Start       | `sudo systemctl start mongod`                                                   |
| MongoDB Status      | `sudo systemctl status mongod`                                                  |
| MongoDB Shell Open  | `mongosh`                                                                       |
| নতুন Database তৈরি  | `use myDatabase`                                                                |
| নতুন User তৈরি      | `db.createUser({...})`                                                          |
| User Password Reset | `db.updateUser("user", {pwd: "newPass"})`                                       |
| Firewall Enable     | `sudo ufw enable`                                                               |
| নির্দিষ্ট IP Allow  | `sudo ufw allow from 192.168.1.100 to any port 27017`                           |
| সকল IP Allow        | `sudo ufw allow 27017/tcp`                                                      |
| Database Rename     | `mongodump --db oldDB --out /backup/ && mongorestore --db newDB /backup/oldDB/` |
| Database Delete     | `use myDatabase; db.dropDatabase()`                                             |

---

এভাবে আপনি **MongoDB সেটআপ, ইউজার ম্যানেজমেন্ট, ডাটাবেস ম্যানেজমেন্ট, এবং Firewall কনফিগারেশন** করতে পারবেন! 🚀

<br />
<br />
<br />
<br />
<br />
<br />

# **MongoDB automatic backup** সেটআপ করবেন কীভাবে এবং **প্রতিদিন ২ বার** ব্যাকআপ নেওয়ার ব্যবস্থা করা।

---

## **Step 1: VPS-এ লগইন করুন**

প্রথমে **VPS-এর টার্মিনাল বা SSH** তে লগইন করুন:

```bash
ssh your_user@your_vps_ip
```

এখানে:

- `your_user` = আপনার VPS-এর ইউজারনেম
- `your_vps_ip` = আপনার VPS-এর IP Address

---

## **Step 2: ব্যাকআপ সংরক্ষণের জন্য ফোল্ডার তৈরি করুন**

MongoDB-এর ব্যাকআপগুলো সংরক্ষণ করার জন্য একটি ফোল্ডার তৈরি করুন:

```bash
mkdir -p ~/mongodb_backups
```

এটি আপনার **হোম ডিরেক্টরিতে** `mongodb_backups` নামে একটি ফোল্ডার তৈরি করবে।

---

## **Step 3: ব্যাকআপ স্ক্রিপ্ট তৈরি করুন**

এখন `backup.sh` নামে একটি স্ক্রিপ্ট তৈরি করতে হবে। এই ফাইলটি ব্যাকআপ নেওয়ার কাজ করবে।

1. **nano editor** দিয়ে স্ক্রিপ্ট তৈরি করুন:

   ```bash
   nano ~/backup.sh
   ```

2. ফাইলের মধ্যে নিচের কোড **copy-paste** করুন:

   ```bash
   #!/bin/bash

   # ব্যাকআপের জন্য তারিখ ও সময় যুক্ত করা
   TIMESTAMP=$(date +"%F-%H-%M-%S")

   # MongoDB সেটআপ তথ্য
   DB_NAME="your_database_name"   # আপনার ডাটাবেসের নাম দিন
   BACKUP_DIR="/home/your_user/mongodb_backups"   # ব্যাকআপ সংরক্ষণের লোকেশন
   MONGO_HOST="localhost"
   MONGO_PORT="27017"

   # ব্যাকআপ ডিরেক্টরি যদি না থাকে, তাহলে তৈরি করো
   mkdir -p "$BACKUP_DIR"

   # MongoDB ডাটাবেস ব্যাকআপ নেওয়া
   mongodump --host $MONGO_HOST --port $MONGO_PORT --db $DB_NAME --out "$BACKUP_DIR/$DB_NAME-$TIMESTAMP"

   # পুরনো ৭ দিনের ব্যাকআপ ডিলিট করা
   find "$BACKUP_DIR" -type d -mtime +7 -exec rm -rf {} \;

   echo "MongoDB backup completed at $TIMESTAMP"
   ```

   ⚠️ **যা পরিবর্তন করবেন:**

   - `your_database_name` = আপনার **MongoDB database name**
   - `your_user` = আপনার VPS-এর **username**

3. **ফাইলটি সংরক্ষণ করুন:**
   - **Ctrl + X** চাপুন
   - তারপর **Y** চাপুন এবং **Enter** দিন
4. **যদি সম্পূর্ণ Database সংরক্ষণ করতে চান:**

   ```bash
   #!/bin/bash
   echo "Backup script started at $(date)" >> /root/erro_file_store/mongodb_backup.log
   # ব্যাকআপের জন্য তারিখ ও সময় যুক্ত করা
   TIMESTAMP=$(date +"%F-%H-%M-%S")

    # MongoDB সেটআপ তথ্য
    BACKUP_DIR="/home/root/mongodb_backups"   # ব্যাকআপ সংরক্ষণের লোকেশন
    MONGO_HOST="localhost"
    MONGO_PORT="27017"

    # ব্যাকআপ ডিরেক্টরি যদি না থাকে, তাহলে তৈরি করো
    mkdir -p "$BACKUP_DIR"

    # MongoDB সকল ডাটাবেসের ব্যাকআপ নেওয়া
    mongodump --host $MONGO_HOST --port $MONGO_PORT --out "$BACKUP_DIR/mongodb-backup-$>

    # Google Drive এ ব্যাকআপ আপলোড করা (rclone ব্যবহার)
    rclone copy "$BACKUP_DIR/mongodb-backup-$TIMESTAMP" gdrive:/mongodb_backups/ --log->

    # পুরনো ৭ দিনের ব্যাকআপ ডিলিট করা
    find "$BACKUP_DIR" -type d -mtime +7 -exec rm -rf {} \;

    echo "MongoDB backup completed at $TIMESTAMP and uploaded to Google Drive"

   ```


---

## **Step 4: স্ক্রিপ্টটি এক্সিকিউটেবল করুন**
এই স্ক্রিপ্টটি রান করার জন্য **এক্সিকিউটেবল (Executable)** করতে হবে:
```bash
chmod +x ~/backup.sh
````

---

## **Step 5: Cron Job সেটআপ করুন (প্রতিদিন ২ বার ব্যাকআপ নেওয়া)**

এখন, আমরা **ক্রনজব (cron job)** সেট করব যাতে **প্রতিদিন ২ বার (সকাল ৮টা এবং রাত ৮টা)** ব্যাকআপ নেয়।

1. ক্রনজব এডিটর খুলুন:

   ```bash
   crontab -e
   ```

2. নিচের লাইনটি যোগ করুন:

   ```bash
   0 8 * * * /home/your_user/backup.sh >> /home/your_user/backup.log 2>&1
   0 20 * * * /home/your_user/backup.sh >> /home/your_user/backup.log 2>&1
   ```

   **ব্যাখ্যা:**

   - `0 8 * * *` → **প্রতিদিন সকাল ৮টায় ব্যাকআপ নিবে**
   - `0 20 * * *` → **প্রতিদিন রাত ৮টায় ব্যাকআপ নিবে**
   - `backup.log` → ব্যাকআপের **লগ (log) ফাইল** তৈরি হবে, যেখানে ব্যাকআপ নেওয়ার রেকর্ড থাকবে।

3. **Ctrl + X** চাপুন, তারপর **Y** চাপুন এবং **Enter** দিন।

---

## **Step 6: ম্যানুয়ালি ব্যাকআপ পরীক্ষা করুন**

ক্রনজব কাজ করছে কিনা তা নিশ্চিত করতে **ম্যানুয়ালি** স্ক্রিপ্ট চালান:

```bash
~/backup.sh
```

তারপর, চেক করুন ব্যাকআপ ফোল্ডারে ফাইল তৈরি হয়েছে কিনা:

```bash
ls -lah ~/mongodb_backups
```

যদি সব ঠিক থাকে, তাহলে দেখবেন একটি নতুন ব্যাকআপ ফোল্ডার তৈরি হয়েছে।

---

## **Step 7: ব্যাকআপ ফাইল ডাউনলোড / রিস্টোর করার পদ্ধতি**

### **✅ ব্যাকআপ ফাইল ডাউনলোড করা (VPS থেকে নিজের কম্পিউটারে)**

আপনি **scp** কমান্ড দিয়ে ব্যাকআপ ফাইল ডাউনলোড করতে পারেন:

```bash
scp -r your_user@your_vps_ip:/home/your_user/mongodb_backups ./
```

এটি **আপনার লোকাল পিসিতে** `mongodb_backups` ফোল্ডারটি ডাউনলোড করবে।

---

### **✅ MongoDB ব্যাকআপ রিস্টোর করা (Restore Database from Backup)**

ব্যাকআপ থেকে **ডাটাবেস রিস্টোর** করতে চাইলে এই কমান্ড ব্যবহার করুন:

```bash
mongorestore --db your_database_name /home/your_user/mongodb_backups/backup_folder_name/
```

এখানে `backup_folder_name` পরিবর্তন করে আপনার নির্দিষ্ট ব্যাকআপ ফোল্ডারের নাম দিন।

---

## **NOTE**

এখন আপনার **MongoDB automatic backup system** **সফলভাবে সেটআপ** করা হয়ে গেছে! প্রতিদিন **সকাল ৮টা ও রাত ৮টা**-তে VPS **অটোমেটিক MongoDB ব্যাকআপ** নিয়ে সংরক্ষণ করবে এবং পুরনো **৭ দিনের ব্যাকআপ মুছে ফেলবে।** ⤵

<br />
<br />
<br />
<br />
<br />

# **Google Drive-এ MongoDB Backup অটোমেটিক আপলোড**

আপনার VPS-এ MongoDB ব্যাকআপ গুগল ড্রাইভে **অটোমেটিক** আপলোড করতে **rclone** ব্যবহার করা হবে।

---

### **Step 1: VPS-এ rclone ইন্সটল করুন**

VPS-এর টার্মিনালে নিচের কমান্ড দিন:

```bash
curl https://rclone.org/install.sh | sudo bash
```

এটি **rclone** ইন্সটল করবে।

---

### **Step 2: rclone কনফিগার করুন (Google Drive একাউন্ট যুক্ত করুন)**

```bash
rclone config
```

এরপর ধাপে ধাপে অনুসরণ করুন:

1. নতুন কনফিগার তৈরি করুন:

   - `n` চাপুন (New remote)
   - নাম দিন: `gdrive`

2. Google Drive নির্বাচন করুন:

   - `Storage type` চাবে, সেখানে `drive` টাইপ করুন

3. **Client ID & Secret এন্টার করা লাগবে না**, শুধু **Enter** চাপুন

4. **Authentication Method**:

   - **Full access দিতে** `1` চাপুন

5. **Google Drive Authorization:**
   - VPS-এর **ব্রাউজার না থাকলে**, আপনাকে একটা **লিংক দেবে**
   - **লিংক ওপেন করে** আপনার **Google Account** দিয়ে লগইন করুন
   - **Authentication কোড কপি করে** VPS-এর টার্মিনালে পেস্ট করুন

---

Link টা এমন হবে

```
http://127.0.0.1:53682/?state=VQ99iErCu78A8OCLfTHXjg&code=4/0AQSTgQEhhRZ8gaPYFtM3teE_71FhKXF0iPn6DEUyiUn5h_sHtfLtUiVQyY1Iy0MwhyWxXw&scope=https://www.googleapis.com/auth/drive
```

---

6. সেটিংস সেভ করুন:
   - `y` চাপুন (Yes)

---

### **Step 3: ব্যাকআপ স্ক্রিপ্টে Google Drive আপলোড যুক্ত করুন**

`backup.sh` ফাইল খুলুন:

```bash
nano ~/backup.sh
```

নিচের লাইনটি স্ক্রিপ্টের **শেষে** যুক্ত করুন:

```bash
rclone copy "$BACKUP_DIR" gdrive:MongoDB_Backups
```

এটি আপনার ব্যাকআপ ফাইল **Google Drive-এর "MongoDB_Backups" ফোল্ডারে** আপলোড করবে।

---

### **Step 4: ম্যানুয়ালি আপলোড পরীক্ষা করুন**

```bash
./backup.sh
```

তারপর Google Drive-এ **MongoDB_Backups** ফোল্ডারে গিয়ে দেখুন ফাইল এসেছে কিনা।

---

### **Step 5: Cron Job আপডেট করুন (Auto Upload)**

```bash
crontab -e
```

নিচের লাইনটি যোগ করুন:

```bash
0 8 * * * /home/your_user/backup.sh >> /home/your_user/backup.log 2>&1
0 20 * * * /home/your_user/backup.sh >> /home/your_user/backup.log 2>&1
```

এতে **প্রতিদিন ২ বার** ব্যাকআপ নিয়ে **Google Drive-এ আপলোড** হবে।

---

<br />
<br />
<br />
<br />

# **AWS S3-তে MongoDB Backup আপলোড করা**

AWS S3-তে MongoDB ব্যাকআপ আপলোড করার জন্য **AWS CLI** ব্যবহার করবো।

---

### **Step 1: AWS CLI ইন্সটল করুন**

VPS-এর টার্মিনালে রান করুন:

```bash
sudo apt update && sudo apt install awscli -y
```

---

### **Step 2: AWS Credentials সেটআপ করুন**

```bash
aws configure
```

এখন **AWS Access Key & Secret Key দিন** (আপনার AWS IAM Console থেকে নিতে হবে)।

- Access Key: `your_aws_access_key`
- Secret Key: `your_aws_secret_key`
- Region: `ap-southeast-1` (বাংলাদেশের জন্য Singapore)
- Output Format: `json`

---

### **Step 3: S3-তে ব্যাকআপ আপলোড যুক্ত করুন**

`backup.sh` স্ক্রিপ্ট আপডেট করুন:

```bash
nano ~/backup.sh
```

নিচের লাইনটি স্ক্রিপ্টের **শেষে** যুক্ত করুন:

```bash
aws s3 cp --recursive "$BACKUP_DIR" s3://your-bucket-name/mongodb_backups/
```

⚠️ **your-bucket-name** পরিবর্তন করে **আপনার S3 bucket name দিন**।

---

### **Step 4: ম্যানুয়ালি আপলোড পরীক্ষা করুন**

```bash
./backup.sh
```

তারপর **AWS S3 console**-এ গিয়ে চেক করুন, ব্যাকআপ ফাইল আপলোড হয়েছে কিনা।

---

### **Step 5: Cron Job আপডেট করুন (Auto Upload to S3)**

```bash
crontab -e
```

নিচের লাইনটি যোগ করুন:

```bash
0 8 * * * /home/your_user/backup.sh >> /home/your_user/backup.log 2>&1
0 20 * * * /home/your_user/backup.sh >> /home/your_user/backup.log 2>&1
```

এতে **প্রতিদিন ২ বার** ব্যাকআপ নিয়ে **AWS S3-তে আপলোড** হবে।

---

## **Final Result:**

✅ MongoDB-এর ব্যাকআপ **VPS-এ সংরক্ষণ** হবে  
✅ **Google Drive / AWS S3**-তে **Auto Upload** হবে  
✅ পুরনো **৭ দিনের ব্যাকআপ অটোমেটিক ডিলিট** হবে

---
