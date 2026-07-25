# WorkHive — মডুলার আর্কিটেকচার

## 📁 ফাইল স্ট্রাকচার
```
index.html              ← মার্কআপ + স্ক্রিন শেল (এইটাই একমাত্র .html ফাইল)
css/
  base.css               ← বিদ্যমান ডিজাইন সিস্টেম (অপরিবর্তিত)
  post.css               ← নতুন "Post a Job" ফিচারের স্টাইল
js/
  main.js                ← এন্ট্রি পয়েন্ট — সব মডিউল জোড়া লাগায়
  core/
    firebase.js           ← Firebase config + SDK (একমাত্র জায়গা যেখানে এটা আছে)
    state.js               ← গ্লোবাল state (S) + এডমিন-কনফিগ (SITE)
    utils.js, toast.js, dailyLink.js, nav.js
  services/                ← Firestore-এর সাথে কথা বলা প্রতিটা ফাংশন
    auth.js, jobs.js, postJobs.js, submissions.js, wallet.js,
    notifications.js, referral.js, support.js, settings.js, cloudinary.js
  screens/                  ← প্রতিটা ট্যাব/পেজের নিজস্ব ফাইল
    auth.js, home.js, tasks.js, mytasks.js, post.js, wallet.js,
    history.js, notifications.js, referral.js, profile.js, spin.js, support.js
  components/
    taskCard.js, modal.js
firestore.rules           ← Firebase Console → Firestore → Rules-এ পেস্ট করুন
```

**একটা নির্দিষ্ট সেকশন বদলাতে চাইলে** শুধু সেই একটা ফাইল খুলে এডিট করুন — বাকি কিছু ছোঁয়ার দরকার নেই। যেমন: Wallet-এর ডিজাইন বদলাতে চাইলে শুধু `js/screens/wallet.js` + দরকার হলে `css/base.css`-এর `.wallet-*` ক্লাসগুলো।

## 🚀 GitHub Pages-এ লাইভ করা
1. পুরো ফোল্ডার (এই কাঠামো অক্ষুণ্ণ রেখে) আপনার রিপোতে আপলোড/কমিট করুন।
2. Settings → Pages → Branch সিলেক্ট করে Save করুন।
3. `https://<username>.github.io/<repo>/` এ সাইট লাইভ হবে।
4. Firebase Console → Authentication → Settings → Authorized domains-এ আপনার GitHub Pages ডোমেইন যোগ করুন (নইলে লগইন কাজ করবে না)।

## ⚙️ কনফিগ-চালিত এডমিন প্যানেল (নিরাপদ পদ্ধতি, যেটা আপনি বেছে নিয়েছেন)
Firestore-এর `siteSettings/global` ডকুমেন্টে যেকোনো একটা ফিল্ড বদলালেই সব ইউজারের স্ক্রিনে লাইভ আপডেট হয়ে যাবে — কোনো কোড ডিপ্লয় লাগবে না। এই মুহূর্তে যেসব ফিল্ড কাজ করে (আপনার এডমিন প্যানেল UI-তে এগুলোর জন্য ফর্ম বানালেই হবে):

| ফিল্ড | কী নিয়ন্ত্রণ করে |
|---|---|
| `minWithdraw`, `withdrawFee` | Withdraw ফর্মের নিয়ম |
| `refGen1/2/3` | রেফারেল বোনাস অ্যামাউন্ট |
| `spinEnabled` | Lucky Spin চালু/বন্ধ |
| `banner`, `bannerType` | হোমপেজের ব্যানার |
| `supportEmail/Hours/Response` | সাপোর্ট সেকশনের তথ্য |
| `postFeatureEnabled` | Post ফিচার পুরো চালু/বন্ধ |
| `postMinReward` / `postMaxReward` | ইউজার-পোস্টেড জবে রিওয়ার্ডের সীমা |
| `postMinSlots` / `postMaxSlots` | স্লট সংখ্যার সীমা |
| `postPlatformFeePct` | বাজেট প্রিভিউয়ে প্ল্যাটফর্ম ফি % |
| `postApprovalNote` | Post ফর্মের উপরে দেখানো নোট |

নতুন কনফিগ-ফিল্ড যোগ করতে: `js/core/state.js`-এর `SITE` অবজেক্টে ডিফল্ট বসান → `js/services/settings.js`-এ পড়ুন → প্রাসঙ্গিক স্ক্রিনের `applySiteSettingsTo...()`-এ প্রয়োগ করুন।

## 📢 নতুন "Post" ফিচার কীভাবে কাজ করে
ইউজার ফর্ম পূরণ করে → `jobs` কালেকশনে `status:'pending_review'` হিসেবে সেভ হয় → **আপনার এডমিন প্যানেলে এটা দেখাবে**, approve করলে `status:'approved'` হয়ে গেলে Tasks ট্যাবে সবাই দেখতে পাবে। `firestore.rules`-এ এটা এনফোর্স করা আছে — ইউজার নিজে থেকে `status:'approved'` লিখতে পারবে না।

**এডমিন প্যানেলে যা বানাতে হবে (পরবর্তী ধাপ):** pending_review জবের তালিকা দেখানো, Approve/Reject বাটন (reject করলে `rejectReason` ফিল্ড লিখুন — এটা ইউজারের "My Posted Jobs"-এ দেখা যাবে)।

## 🔐 নিরাপত্তা — এখন যা করা হয়েছে, যা এখনো বাকি
- ✅ `firestore.rules` দিয়ে user-posted জব জোর করে `pending_review` রাখা হয়েছে।
- ✅ Submission/withdrawal-এর status বদলানো শুধু admin custom claim দিয়েই সম্ভব।
- ⚠️ **এখনো বাকি (Phase 2, সবচেয়ে জরুরি):** ডেইলি-লিংক টাস্ক অটো-ক্রেডিট আর ৩-জেনারেশন রেফারেল বোনাস এখনো ব্রাউজার থেকে সরাসরি `balance` বাড়ায় (মূল কোডের মতোই)। এগুলো Cloud Functions-এ সরানো দরকার যাতে ইউজার নিজে ব্যালেন্স বাড়াতে না পারে। যতক্ষণ না এটা হয়, ছোট স্কেলে চালান এবং লেনদেন নিয়মিত মনিটর করুন।
- এডমিন কাস্টম ক্লেইম সেট করতে: Firebase CLI বা একটা ওয়ানটাইম Cloud Function দিয়ে `admin: true` বসান — কখনো Firestore ফিল্ড দিয়ে না।
