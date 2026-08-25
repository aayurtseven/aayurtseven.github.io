// ZZ XTREAM — RTDB temizlik scripti
// Ne yapar: devices/<mac>/profiles altındaki e-posta değerlerini bulur,
// users/<uid>/profileIndex içindeki profil ADIYLA değiştirir.
//
// Kullanım:
//   node zz-eposta-temizlik.js          → RAPOR modu (hiçbir şeyi değiştirmez, ne yapacağını listeler)
//   node zz-eposta-temizlik.js fix      → FIX modu (gerçekten düzeltir)
//
// Kimlik: GOOGLE_APPLICATION_CREDENTIALS ortam değişkeni servis anahtarı
// JSON dosyasını göstermeli. (GitHub Actions kullanıyorsan buna gerek yok,
// workflow kendisi ayarlıyor.)
//
// GÜVENLİK: Loglarda e-postalar maskelenir (ab***@gmail.com). Bu dosyayı
// herkese açık repoya koyabilirsin — içinde gizli bilgi YOK; gizli olan
// yalnızca servis anahtarı JSON'udur ve o GitHub Secret'ta kalır.

const admin = require("firebase-admin");
const fs = require("fs");

const keyPath = process.env.GOOGLE_APPLICATION_CREDENTIALS;
if (!keyPath || !fs.existsSync(keyPath)) {
  console.error("Servis anahtarı bulunamadı: GOOGLE_APPLICATION_CREDENTIALS=" + keyPath);
  process.exit(1);
}

admin.initializeApp({
  credential: admin.credential.cert(JSON.parse(fs.readFileSync(keyPath, "utf8"))),
  databaseURL: "https://zziptv-player-control-default-rtdb.europe-west1.firebasedatabase.app",
});

const db = admin.database();
const MODE = process.argv[2] === "fix" ? "fix" : "report";

function mask(v) {
  if (typeof v !== "string") return v;
  return v.replace(/(.{2}).+(@.+)/, "$1***$2");
}

async function profilAdiBul(pid) {
  try {
    const idx = (await db.ref(`users/${pid}/profileIndex`).get()).val() || {};
    if (typeof idx[pid] === "string" && !idx[pid].includes("@")) return idx[pid];
    const aday = Object.values(idx).find((x) => typeof x === "string" && !x.includes("@"));
    if (aday) return aday;
  } catch (e) {}
  return "Profil";
}

(async () => {
  const snap = await db.ref("devices").get();
  const devices = snap.val() || {};
  let tarananProfil = 0;
  let bulunanMail = 0;
  let duzeltilen = 0;

  for (const [mac, dev] of Object.entries(devices)) {
    const profs = dev && dev.profiles;
    if (!profs) continue;

    for (const [pid, val] of Object.entries(profs)) {
      tarananProfil++;
      if (typeof val !== "string" || !val.includes("@")) continue;
      bulunanMail++;

      const yeniAd = await profilAdiBul(pid);
      const path = `devices/${mac}/profiles/${pid}`;

      if (MODE === "fix") {
        await db.ref().update({ [path]: yeniAd });
        duzeltilen++;
      }
      console.log(`${MODE === "fix" ? "DUZELTILDI  " : "DUZELTILECEK"} ${path}  eski=${mask(val)}  yeni=${yeniAd}`);
    }
  }

  console.log("");
  console.log(`OZET: ${tarananProfil} profil tarandi, ${bulunanMail} e-posta bulundu, ${duzeltilen} duzeltildi. Mod: ${MODE}`);
  if (MODE !== "fix" && bulunanMail > 0) {
    console.log("Uygulamak icin: node zz-eposta-temizlik.js fix   (veya Actions'ta mode=fix)");
  }
  process.exit(0);
})().catch((e) => {
  console.error("HATA:", e.message);
  process.exit(1);
});
