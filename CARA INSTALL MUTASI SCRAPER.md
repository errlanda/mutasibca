CARA INSTALL DI LINUX

Khusus BCA ini script punya Abdul Muttaqin
Dikembangkan Erlanda Cahya K

gunakan root dengan
sudo -i

atur
sudo visudo

kalau bisa sudo usermod -aG sudo your_username

sudo apt-get update
sudo apt-get install -y libnss3

sudo apt-get install -y libatk-bridge2.0-0 libx11-xcb1 libxcomposite1 libxdamage1 libxi6 libxtst6 libnss3 libxrandr2 libasound2 libpangocairo-1.0-0 libatk1.0-0 libgtk-3-0





tambahkan
userkamu ALL=(ALL) NOPASSWD:ALL
control o, enter, control x

sudo apt install unzip

unzip mutasi-scraper-main.zip -d ~/mutasi-scraper



install seperti biasa di root atau manapun
install pakai package asli



npm install --save mutasi-scraper



cek
di dalam folder "wajib digunakan"

setelah diinstall ubah package.jsnya yang ada di dalam node_modules/mutasi-scraper/
tambahkan folder dan file examples/example.js pada node_modules/mutasi-scraper/

masukkan FILE BCA.class.ts
pada node_modules/mutasi-scraper/src/bank/BCA.class.ts




jalankan dengan

sudo npm run example
atau
sudo npx tsx examples/example.js

ini isi file example.js

////////////////////////////////////////////////////////////////////////////////////////////////////////

import { ScrapBCA } from "mutasi-scraper";
import { BCAParser } from "../src/helper/utils/Parser";
import axios from "axios"; // Import axios
"use strict";

// Fungsi penundaan kustom
function delay(time) {
  return new Promise(resolve => setTimeout(resolve, time));
}

const user = "xxxxxxxxx"; // usernamenya akun bank
const pass = "xxxxxxx"; // passwordnya akun bank
const norek = "xxxxxxx"; // pasti nomor rekeningnya

async function runScraper() {
  const scraper = new ScrapBCA(user, pass, norek, {
    headless: true,  // ganti false kalau dibutuhkan
    args: [
      "--log-level=3",
      "--no-default-browser-check",
      "--disable-infobars",
      "--disable-web-security",
      "--disable-site-isolation-trials", 
"--no-sandbox"
    ],
  });

  const today = new Date();
  const yesterday = new Date(today);
  yesterday.setDate(today.getDate() - 8); //dibuat minus berapa hari

  const tglawal = yesterday.getDate(); // Tanggal kemarin
  let blnawal = yesterday.getMonth() + 1; // Bulan kemarin (perlu ditambah 1 karena Januari dimulai dari 0)
  const tglakhir = today.getDate(); // Tanggal hari ini
  const blnakhir = today.getMonth() + 1; // Bulan saat ini (perlu ditambah 1 karena Januari dimulai dari 0)

  // Jika kemarin adalah bulan sebelumnya
  if (tglawal > today.getDate()) {
    blnawal = today.getMonth(); // Atur bulan awal ke bulan sebelumnya
    if (blnawal === 0) { // Jika bulan awal menjadi Januari (0), atur ke Desember (12)
      blnawal = 12;
    }
  }

// di sini bisa ditambahkan pengaturan

  try {
    await delay(2000); // Jeda 2 detik setelah login
    await scraper.loginToBCA();
    await delay(10000); // Jeda 10 detik setelah login
    const mutasinya = await scraper.selectAccountAndSetDates(tglawal, blnawal, tglakhir, blnakhir);
    await delay(5000); // Jeda 5 detik setelah memilih tanggal

    // Ambil HTML dari halaman
    const htmlContent = await mutasinya.content();
    await delay(2000);

    // Proses HTML dengan BCAParser
    const selectors = {
      accountNoField: 'font:contains("Nomor Rekening")',
      nameField: 'font:contains("Nama")',
      periodeField: 'font:contains("Periode")',
      mataUangField: 'font:contains("Mata Uang")',
      transactionsTable: 'table[border="1"]',
      settlementTable: 'table[border="0"][width="70%"]',
    };

    const bcaParser = new BCAParser(htmlContent, selectors);
    const result = bcaParser.parse();
    console.log(result);

    // Kirim hasil parsing ke endpoint URL bot
    const mutasiMasuk = result.mutasi.filter(item => item.mutasi === 'CR');

    for (const item of mutasiMasuk) {
      let referenceId2 = item.nominal.replace(/,/g, ''); // Menghilangkan koma dari nominal
      referenceId2 = referenceId2.split('.')[0]; // Menghapus titik dan angka setelah titik




      // Kirim ke endpoint pertama INI HANYA CONTOH JANGAN DIIKUTIN
      await axios.post('https://wa.erland.biz.id/mutasi', { reference_id2 : referenceId2 })
        .then(response => {
          console.log('Data berhasil dikirim ke wa BOT:', response.data);
        })
        .catch(error => {
          console.error('Error mengirim data:', error);
        });

      // Kirim ke endpoint kedua
      await axios.post('https://hazline.com/endpoint/', { number: referenceId2 })
        .then(response => {
          console.log('Data berhasil dikirim ke hazline:', response.data);
        })
        .catch(error => {
          console.error('Error mengirim data ke hazline:', error);
        });



    }

    await delay(5000); // Jeda 5 detik sebelum logout
    // Logout dan tutup sesi setelah parsing selesai
    await scraper.logoutAndClose();
    console.log("Task completed successfully.");

    runScraper(); // Restart the process
  } catch (error) {
    console.error("Error: ", error);
    

    if (error.message.includes("Anda dapat melakukan login kembali setelah 5 menit") || 
    error.message.includes("You can re-login after 5 minutes")) {
  await scraper.logoutAndClose();
  console.log("Re-login required. Waiting for 5 minutes before retrying.");

  
  await delay(300000); // Jeda 5 menit (300.000 milidetik)
} else {
  await scraper.logoutAndClose();
}


    await scraper.logoutAndClose();
    runScraper(); // Restart the process after an error
  }
}

runScraper();



//////////////////////////////////////////

lanjut ini adalah package.js


///////////////////////////////////////////
{
  "name": "mutasi-scraper",
  "version": "2.2.28",
  "type": "module",
  "main": "src/index.ts",
  "directories": {
    "example": "examples",
    "lib": "lib"
  },
  "scripts": {
    "test": "jest",
    "start": "npx tsx examples/example.js",
    "example": "npx tsx examples/example.js",
    "jsdoc": "jsdoc -c jsdoc.json -r -R README.md src -d docs",
    "cron": "node cron-job.js"
  },
  "author": "Abdul Muttaqin",
  "license": "MIT",
  "description": "Scrap all settlement from indonesian banks",
  "dependencies": {
    "@traceo-sdk/node": "^0.34.1",
    "axios": "^1.6.3",
    "cheerio": "^1.0.0-rc.12",
    "date-format": "^4.0.10",
    "express": "^4.19.2",
    "fingerprint-injector": "^2.1.50",
    "karma-chrome-launcher": "^3.2.0",
    "moment": "^2.29.4",
    "mutasi-scraper": "^2.2.28",
    "node-cron": "^3.0.0",
    "puppeteer": "^21.6.1",
    "puppeteer-extra": "^3.3.6",
    "puppeteer-extra-plugin-adblocker": "^2.13.6",
    "puppeteer-extra-plugin-stealth": "^2.11.2",
    "puppeteer-recaptcha-bypass": "^1.0.1",
    "tsx": "^4.15.5",
    "zustand": "^4.5.2"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/fdciabdul/mutasi-scraper.git"
  },
  "keywords": [
    "mutasi",
    "mutasi",
    "scraper"
  ],
  "bugs": {
    "url": "https://github.com/fdciabdul/mutasi-scraper/issues"
  },
  "homepage": "https://github.com/fdciabdul/mutasi-scraper#readme",
  "devDependencies": {
    "@types/karma-chrome-launcher": "^3.1.4",
    "@types/node": "^20.12.7",
    "docdash": "^2.0.2",
    "jsdoc": "^4.0.2",
    "jsdoc-to-markdown": "^8.0.0"
  }
}
