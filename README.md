# දීමනාපත්‍ර ලේඛනාගාරය — Web App (Firebase Edition)

## මොනවද මේ update එකෙන් වෙනස් වුනේ?

මෙය දැන් **සම්පූර්ණ web application** එකකි. කලින් තිබුනු Node.js server
(sql.js local database) එක ඉවත් කර, ඒ වෙනුවට **Firebase Firestore**
(cloud database) එකකට connect කර ඇත. මින් ලැබෙන වාසි:

- **submit වැඩ නොකරන issue එක fix වී ඇත** — කලින් localhost server එකක්
  run කර නොතිබුනොත් (server run කරන පරිගණකයෙන් නොවේ නම්) submit/save
  වැඩ කරන්නේ නැහැ. දැන් server ekak run karanna oni na — browser eken
  directly Firebase ekata data save wenawa.
- **ඕනෑම තැනකින් access කළ හැක** — computer, mobile, office network,
  home internet — ඕනෑම device එකකින්, ඕනෑම තැනකින්, internet
  connection එකක් තිබුනොත් access කරන්න පුළුවන්.
- **Real-time cloud database** — data save wenne Firebase Firestore
  ekata, so hardware fail unath data safe.

## 1. Deploy කරන විදිය (Firebase Hosting)

1. Node.js install කරගන්න (නොමැති නම්): https://nodejs.org
2. Firebase CLI install කරගන්න:
   ```
   npm install -g firebase-tools
   ```
3. Firebase account එකට login වෙන්න:
   ```
   firebase login
   ```
4. මෙම folder එකට terminal එකෙන් යන්න, ඉන්පසු deploy කරන්න:
   ```
   firebase deploy
   ```
5. Deploy සම්පූර්ණ වූ පසු, terminal එකේ පෙන්වන **Hosting URL** එක
   (උදා: `https://land-permit-registry.web.app`) කවුරුත්ට share කරන්න
   පුළුවන් — ඒකෙන් website එකට access කරගන්න පුළුවන්.

## 2. දත්ත ආරක්ෂාව (Security) — Login සමඟ

මෙම app එකේ දැන් **Firebase Authentication (Email + Password)** login
system එකක් ඇත. Login නොවී කිසිවෙකුට data කියවීමට හෝ වෙනස් කිරීමට
බැහැ. ගිණුම් දෙකකි:

- **Officer** — සටහන් කියවීම, අලුතින් සෑදීම, සංස්කරණය කළ හැක.
- **Admin** — Officer ට කළ හැකි සියල්ල + සටහන් මකා දැමීම + Backup/Restore.

### Setup — එක වතාවක් කරන්න

1. Firebase Console → **Build → Authentication → Get started** → **Sign-in
   method** tab එකෙන් **Email/Password** provider එක **Enable** කරන්න:
   https://console.firebase.google.com/project/land-permit-registry/authentication/providers
2. `firestore.rules` deploy කරන්න (login නොවූ, සහ තවම අනුමත නොවූ, අයට
   block කරන නව rules ක්‍රියාත්මක කිරීමට අනිවාර්යයි):
   ```
   firebase deploy --only firestore:rules
   ```
3. Website එකට ගොස් **"ගිණුමක් සාදන්න"** ටැබ් එකෙන් ඔබගේම කාර්යාල
   ඊමේල් ලිපිනයකින් පළමු ගිණුම සාදන්න. **මෙම ගිණුම default එකේ
   "Officer" + "pending" (අනුමත නොවූ) ලෙසින් සෑදේ** — කවුරුත්ට sign-up
   screen එකෙන්ම Admin තෝරාගත නොහැක.
4. ඔබගේ **පළමු Admin ගිණුම** manual එකක් ලෙස Firestore Console එකෙන්
   setup කරගන්න (මෙය එක වතාවක් පමණක් අවශ්‍යයි):
   - **Build → Firestore Database → Data → users** collection එකට යන්න:
     https://console.firebase.google.com/project/land-permit-registry/firestore/data
   - ඔබ දැන් සෑදූ ගිණුමට අදාළ document එක සොයාගන්න (email එක බලා).
   - `role` field එක **"admin"** ලෙසත්, `status` field එක **"approved"**
     ලෙසත් වෙනස් කරන්න.
   - Website එකට ආපසු ගොස් **logout කර නැවත login** වන්න — දැන් ඔබ Admin
     ලෙස පිවිසේවි.
5. ඉන් පසුව, අනෙක් නිලධාරීන් ඔවුන්ගේම ඊමේල් වලින් **"ගිණුමක් සාදන්න"**
   ටැබ් එකෙන් ලියාපදිංචි විය හැක, නමුත් **ඔවුන්ට login විය හැක්කේ ඔබ
   (Admin) ඔවුන් අනුමත කළ පසුවම පමණි** (පහත බලන්න).

### ගිණුම් අනුමත කිරීම — "කාර්ය මණ්ඩල ප්‍රවේශය" (Staff Access)

- **ඕනෑම කෙනෙකුට "ගිණුමක් සාදන්න" ටැබ් එකෙන් sign up විය හැක, නමුත්
  Admin කෙනෙක් අනුමත කරන තුරු ඔවුන්ට කිසිදු දත්තයක් බැලීමට හෝ වෙනස්
  කිරීමට නොහැක** — sign-up එකෙන් සෑදෙන සෑම ගිණුමක්ම default එකේ
  "Officer" + "pending" ලෙසින් සෑදේ, සහ Firestore security rules
  මගින්ම මෙය server-side එකේදී enforce කෙරේ (client-side JS එක edit
  කිරීමෙන් bypass කළ නොහැක).
- Admin ලෙස login වූ පසු, header එකේ **"👥 කාර්ය මණ්ඩල ප්‍රවේශය"** button
  එක (අනුමතියක් රැඳී ඇත්නම් ගණන සහිත badge එකක් සමඟ) පෙනේ. එය click කර:
  - අනුමැතිය රැඳී ඇති ගිණුම් **"Officer ලෙස අනුමත කරන්න"**,
    **"Admin ලෙස අනුමත කරන්න"**, හෝ **"ප්‍රතික්ෂේප (මකන්න)"** කළ හැක.
  - දැනට අනුමත කාර්ය මණ්ඩලයේ role එක වෙනස් කළ හැක (Officer ⇄ Admin),
    හෝ ඔවුන්ගේ ප්‍රවේශය **අවලංගු** කළ හැක (ඔවුන් නැවත pending තත්ත්වයට
    යයි — නැවත login වීමට Admin කෙනෙක් යළි අනුමත කළ යුතුය).
  - ආරක්ෂාව සඳහා, Admin කෙනෙකුට තමන්ගේම ගිණුම මෙතනින් වෙනස් කළ/මකා දැමිය
    නොහැක.

### මුරපදය අමතක වුවහොත්

Firebase Console → Authentication → Users tab එකෙන් password reset
කරන්න, නැතහොත් login screen එකට "මුරපදය අමතකද?" link එකක් එකතු කරන්න
Claude ට කියන්න.

## 3. Backup / Restore

List panel එකේ වම් පස ඇති **"Cloud Database — Backup / Restore"**
කොටුවෙන් සියලුම දත්ත JSON ගොනුවකට export කරගත හැක, හෝ backup ගොනුවකින්
restore කරගත හැක. මෙය දැන් cloud database එකටම බලපාන නිසා, backup එකක්
regularly ගැනීම තවමත් නිර්දේශිතයි.

## 4. Tech stack

- Frontend: Plain HTML/CSS/JS (`public/index.html`)
- Database: Firebase Firestore (cloud, real-time)
- Hosting: Firebase Hosting
- PDF export: html2canvas + jsPDF (browser-side, unchanged)

## 5. Firebase Console

- Project: https://console.firebase.google.com/project/land-permit-registry/overview
- Firestore data: **Build → Firestore Database → Data** tab එකෙන් සියලුම
  records කෙලින්ම බලන්න/edit කරන්න පුළුවන්.
