আপনার স্ক্রিনশট দেখে বুঝলাম সমস্যাটি কোথায়। আপনি টেক্সটগুলো সরাসরি পেস্ট করেছেন,
কিন্তু Markdown Formatting ব্যবহার করেননি। GitHub-এ ডিজাইন সুন্দর করার জন্য
নির্দিষ্ট কিছু কোড (যেমন #, ```, -) ব্যবহার করতে হয়।

নিচের বক্সে আমি আপনার জন্য একদম রেডিমেড কোড লিখে দিচ্ছি। আপনি আপনার README.md
ফাইলটি এডিট মোডে ওপেন করে আগের সব মুছে ফেলুন এবং নিচের এই কোডটি কপি করে পেস্ট
করে সেভ দিন।

# 🚀 Laravel CI/CD Setup with GitHub Actions & cPanel (SSH)

এই গাইডটি আপনাকে শেখাবে কীভাবে **GitHub Actions** ব্যবহার করে আপনার **Laravel** প্রোজেক্ট অটোমেটিক **cPanel/VPS**-এ ডিপ্লয় করবেন। এর ফলে আপনি যখনই কোড পুশ করবেন, সেটি অটোমেটিক সার্ভারে আপডেট হয়ে যাবে।

---

### 📋 ধাপ ১: সার্ভারে SSH Key জেনারেট করা
প্রথমে আপনার সার্ভারে (টার্মিনাল বা জেলশেল ব্যবহার করে) একটি SSH কি তৈরি করতে হবে।

```bash
# SSH কি জেনারেট করতে
ssh-keygen -t ed25519 -C "your_email@example.com"

এন্টার প্রেস করে ডিফল্ট অপশনগুলো সিলেক্ট করুন।

🔑 ধাপ ২: GitHub-এ Secrets সেটআপ করা

আপনার রিপোজিটরির Settings > Secrets and variables > Actions-এ যান এবং নিচের ৪টি
New repository secret অ্যাড করুন:

| Secret Name           | Value (কিভাবে পাবেন)                            |
| :-------------------- | :---------------------------------------------- |
| **SSH\_HOST**         | আপনার সার্ভারের IP অথবা ডোমেইন                  |
| **SSH\_USER**         | সার্ভারের ইউজারনেম (যেমন: `demosashik`)         |
| **SSH\_PORT**         | আপনার SSH পোর্ট (ডিফল্ট `22`)                   |
| **SSH\_PRIVATE\_KEY** | টার্মিনালে `cat ~/.ssh/id_ed25519` লিখে যা আসবে |

সতর্কতা: Private Key-টি -----BEGIN OPENSSH PRIVATE KEY----- থেকে শুরু করে শেষ
পর্যন্ত সবটুকু কপি করবেন।

🛠️ ধাপ ৩: সার্ভারে Public Key পারমিশন দেওয়া

সার্ভারে এই কমান্ডটি রান করুন যাতে GitHub আপনার হয়ে সার্ভারে কমান্ড চালাতে পারে:

cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh

📁 ধাপ ৪: প্রথমবার প্রোজেক্ট ক্লোন করা

আপনার সার্ভারের নির্দিষ্ট ডিরেক্টরিতে গিয়ে প্রথমবার রিপোজিটরিটি ক্লোন করে নিন:

cd /home/demosashik/practice.ashik.top
git clone git@github.com:MDashik-develop/practice-cicd.git

🤖 ধাপ ৫: GitHub Workflow ফাইল তৈরি

আপনার প্রোজেক্টের রুটে .github/workflows/deploy.yml নামে ফাইলটি তৈরি করুন এবং
নিচের কোডটি দিন:

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

            # আপনার সঠিক প্রোজেক্ট ফোল্ডারে ঢোকা
            cd /home/demosashik/practice.ashik.top/practice-cicd

            echo "Pulling latest code from GitHub..."
            git pull origin main

            echo "Deployment finished successfully!"
          EOF

✅ ধাপ ৬: টেস্ট করা

এখন আপনার লোকাল কম্পিউটার থেকে কোডটি git push করুন। GitHub-এর Actions ট্যাবে
গিয়ে দেখুন সব স্টেপ সবুজ (Success) দেখাচ্ছে কি না। 🚀

Author: MD Ashik
Project: Laravel CI/CD with cPanel SSH (Bangla Guide)


### কেন এই ডিজাইনটি সুন্দর দেখাবে?
১. **Headings:** `#` ব্যবহার করার ফলে বড় হেডলাইন তৈরি হবে।
২. **Code Blocks:** ` ```bash ` এবং ` ```yaml ` ব্যবহার করার ফলে কোডগুলো কালো ব্যাকগ্রাউন্ডে সুন্দর করে হাইলাইট হবে।
৩. **Table:** সিক্রেটস গুলো টেবিল আকারে থাকবে, যা পড়তে সহজ।
৪. **Separators:** `---` দিয়ে আমি প্রতিটি ধাপকে আলাদা করে দিয়েছি।
৫. **Emojis:** রকেট, চাবি এবং ফোল্ডার ইমোজিগুলো পড়ার অভিজ্ঞতা আরও ভালো করবে।

**এখন জাস্ট এই সম্পূর্ণ টেক্সটটুকু কপি করে আপনার README ফাইলে পেস্ট করে সেভ দিন!** দেখুন একদম প্রফেশনাল লাগবে।
