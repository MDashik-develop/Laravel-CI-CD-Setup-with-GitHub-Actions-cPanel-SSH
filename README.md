
# 🚀 Laravel CI/CD with SSH & GitHub Actions Guide

এই গাইডটি অনুসরণ করে আপনি আপনার Laravel প্রোজেক্টকে GitHub থেকে সরাসরি cPanel বা VPS সার্ভারে অটোমেটিক ডিপ্লয় করতে পারবেন।

---

### **Step 1: 🔍 Check for Existing SSH Keys**

প্রথমে আপনার সার্ভার টার্মিনালে চেক করুন কোনো SSH Key আছে কি না:

```bash
ls -al ~/.ssh
```

যদি `id_rsa.pub` বা `id_ed25519.pub` না থাকে, তবে নতুন কি তৈরি করুন।

---

### **Step 2: 🔑 Generate a New SSH Key**

আপনার GitHub ইমেইল ব্যবহার করে নিচের কমান্ডটি রান করুন:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

*(সবগুলোতে এন্টার চেপে ডিফল্ট অপশন রাখুন)*

এরপর কি-টি অ্যাক্টিভেট করুন:

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

---

### **Step 3: 📝 Add SSH Key to GitHub**

আপনার পাবলিক কি-টি কপি করুন:

```bash
cat ~/.ssh/id_ed25519.pub
```

**এখন যা করবেন:**
1. আপনার **GitHub Settings** যান।
2. **Settings** -> **SSH and GPG keys** এ যান।
3. **New SSH Key** তে ক্লিক করে কপি করা কি-টি পেস্ট করুন এবং সেভ দিন।

---

### **Step 4: 📂 Clone Repo & Manage Folders**

আপনার সার্ভারের নির্দিষ্ট ফোল্ডারে (যেখানে প্রোজেক্ট রাখতে চান) গিয়ে SSH দিয়ে ক্লোন করুন:

```bash
git clone git@github.com:example/practice-cicd.git
```

**⚠️ প্রোজেক্ট ফাইলগুলো মেইন ডিরেক্টরিতে মুভ করা:**
সাধারণত ক্লোন করলে একটি আলাদা ফোল্ডার তৈরি হয়। সব ফাইল মেইন ডিরেক্টরিতে আনতে এবং খালি ফোল্ডার ডিলিট করতে নিচের কমান্ডগুলো দিন:

```bash
# প্রোজেক্ট ফোল্ডারের ভেতরে সব ফাইল মেইন ডিরেক্টরিতে মুভ করুন
mv practice-cicd/* .
mv practice-cicd/.* .

# এখন খালি ফোল্ডারটি ডিলিট করে দিন
rm -rf practice-cicd
```

---

### **Step 5: 🤖 Setup GitHub Actions Workflow**

আপনার প্রোজেক্টের ভেতর `.github/workflows/deploy-laravel.yml` নামে একটি ফাইল তৈরি করুন এবং নিচের কোডটি পেস্ট করুন:

```yaml
name: Laravel CI/CD Deployment

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup SSH
        uses: webfactory/ssh-agent@v0.9.0
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

      - name: Deploy on Server
        run: |
          ssh -o StrictHostKeyChecking=no -p ${{ secrets.SSH_PORT }} \
          ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }} << 'EOF'
            
            # আপনার প্রোজেক্টের সঠিক পাথে যান
            cd /home/example/practice.ashik.top
            
            echo "Pulling latest code..."
            git pull origin main
            
            echo "Deployment successful!"
          EOF
```

---

### **Step 6: 🔐 Set Repository Secrets**

GitHub-এ আপনার প্রোজেক্টের **Settings > Secrets and variables > Actions**-এ গিয়ে নিচের ৪টি সিক্রেট সেট করুন:

| Variable Name | Description |
| :--- | :--- |
| **`SSH_HOST`** | আপনার সার্ভারের IP বা ডোমেইন |
| **`SSH_USER`** | আপনার SSH ইউজারনেম (যেমন: `demosashik`) |
| **`SSH_PORT`** | SSH পোর্ট (ডিফল্ট সাধারণত `22`) |
| **`SSH_PRIVATE_KEY`** | সার্ভারে `cat ~/.ssh/id_ed25519` লিখে প্রাপ্ত প্রাইভেট কি |

---

### **Step 7: ✅ Final Step - Server Permission**

সার্ভারে এই শেষ কমান্ডটি দিতে ভুলবেন না, যাতে GitHub আপনার হয়ে কমান্ড চালানোর পারমিশন পায়:

```bash
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
```

---

🚀 **এখন থেকে আপনি যখনই কোড পুশ করবেন, তা অটোমেটিক সার্ভারে আপডেট হয়ে যাবে!**
