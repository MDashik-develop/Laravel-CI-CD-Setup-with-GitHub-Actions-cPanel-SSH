আপনার GitHub রিপোজিটরির README.md ফাইলের জন্য একটি সুন্দর এবং প্রফেশনাল ডিজাইন
নিচে দেওয়া হলো। আপনি সরাসরি এটি কপি করে আপনার ফাইলে বসিয়ে দিতে পারেন।

🚀 Laravel CI/CD Setup with GitHub Actions & cPanel (SSH)

এই গাইডটি আপনাকে শেখাবে কীভাবে GitHub Actions ব্যবহার করে আপনার Laravel
প্রোজেক্ট অটোমেটিক cPanel/VPS-এ ডিপ্লয় করবেন। এর ফলে আপনি যখনই কোড পুশ করবেন,
সেটি অটোমেটিক সার্ভারে আপডেট হয়ে যাবে।

📋 ধাপ ১: সার্ভারে SSH Key জেনারেট করা

প্রথমে আপনার সার্ভারে (টার্মিনাল বা জেলশেল ব্যবহার করে) একটি SSH কি তৈরি করতে
হবে।

# SSH কি জেনারেট করতে
ssh-keygen -t ed25519 -C "your_email@example.com"

এন্টার প্রেস করে ডিফল্ট অপশনগুলো সিলেক্ট করুন।

🔑 ধাপ ২: GitHub-এ Secrets সেটআপ করা

আপনার রিপোজিটরির Settings > Secrets and variables > Actions-এ যান এবং নিচের ৪টি
New repository secret অ্যাড করুন:

| Secret Name           | Value (উদাহরণ)                           |
| :-------------------- | :--------------------------------------- |
| **SSH\_HOST**         | `your-server-ip` অথবা `yourdomain.com`   |
| **SSH\_USER**         | `demosashik`                             |
| **SSH\_PORT**         | `22` (cPanel-এর ক্ষেত্রে ভিন্ন হতে পারে) |
| **SSH\_PRIVATE\_KEY** | `cat ~/.ssh/id_ed25519` এর আউটপুট        |

সতর্কতা: Private Key-টি শুরু থেকে শেষ পর্যন্ত (বিগিন এবং এন্ড লাইনসহ) সম্পূর্ণ
কপি করবেন।

🛠️ ধাপ ৩: সার্ভারে Public Key পারমিশন দেওয়া

সার্ভারে লগইন থাকা অবস্থায় রান করুন যাতে GitHub আপনার সার্ভারে ঢোকার অনুমতি পায়:

cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh

📁 ধাপ ৪: প্রথমবার প্রোজেক্ট ক্লোন করা

আপনার সার্ভারের নির্দিষ্ট ডিরেক্টরিতে গিয়ে প্রথমবার রিপোজিটরিটি ক্লোন করে নিন:

cd /home/demosashik/practice.ashik.top
git clone git@github.com:MDashik-develop/practice-cicd.git

🤖 ধাপ ৫: GitHub Workflow ফাইল তৈরি

আপনার প্রোজেক্টের রুটে .github/workflows/deploy.yml ফাইলটি তৈরি করুন এবং নিচের
কোডটি পেস্ট করুন:

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
গিয়ে দেখুন সব স্টেপ সবুজ (Success) দেখাচ্ছে কি না।

⚠️ সাধারণ কিছু প্রশ্ন (FAQ)

১. Node.js deprecation warning কেন আসছে? এটি GitHub Actions-এর ইন্টারনাল
ওয়ার্নিং, আপনার লারাভেল প্রোজেক্টের কোনো সমস্যা নয়। এটি ইগনোর করা যায়।

২. কোড পুশ করার পর সার্ভারে আপডেট হচ্ছে না কেন? আপনার YAML ফাইলে cd এর পর দেওয়া
ফোল্ডার পাথটি আপনার সার্ভারের পাথের সাথে মিলছে কি না চেক করুন।

Author: MD Ashik
Topic: Laravel CI/CD with cPanel SSH (Bangla Guide)

আপনি এই টাইটেলটি দিতে পারেন:

"Laravel CI/CD with cPanel SSH - Bangla Guide" অথবা একটু ক্যাচি করতে চাইলে: "🚀
অটোমেটিক লারাভেল ডিপ্লয়মেন্ট: GitHub Actions & cPanel (Bangla)"
