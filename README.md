# 🎵 Spotify OLED Display

## 📺 Tampilan OLED

```
┌────────────────────────────────┐
│▶  Shape of You >>>scroll>>>    │  ← Judul (auto-scroll jika panjang)
│   Ed Sheeran           80%     │  ← Artist + Volume
│────────────────────────────────│
│                                │
│  I'm in love with your         │  ← Lirik saat ini (word-wrap 2 baris)
│  body                          │
│                                │
│    Oh I, oh I, oh I...         │  ← Preview lirik berikutnya
│────────────────────────────────│
│██████████████░░░░░  2:15       │  ← Progress bar + waktu
└────────────────────────────────┘
  0px                        127px
```

---

## 🗺️ Alur Sistem

```
┌─────────────────────────────────────────────────┐
│              SPOTIFY API  (Cloud)               │
│   Lagu aktif · Progress · Album · Volume        │
└──────────────────────┬──────────────────────────┘
                       │ HTTPS (spotipy OAuth)
                       ▼
┌─────────────────────────────────────────────────┐
│            main.py  (PC / Laptop)               │
│                                                 │
│  SpotifyClient ──► LyricsEngine ──► TrackInfo  │
│                                        │        │
│                      ┌─────────────────┤        │
│                      ▼                 ▼        │
│              DiscordPresence      ESP32Serial   │
└──────────────────────┬─────────────────┬────────┘
                       │ IPC pipe         │ USB Serial
                       ▼                 │ JSON @ 115200 baud
           ┌───────────────────┐         ▼
           │  Discord Desktop  │  ┌──────────────────────┐
           │  Rich Presence    │  │    ESP32-S3           │
           └───────────────────┘  │  parseJson()          │
                                  │  drawMain()           │
                                  └──────────┬────────────┘
                                             │ I2C 400kHz
                                             │ SDA=GPIO8
                                             │ SCL=GPIO9
                                             ▼
                                  ┌──────────────────────┐
                                  │  OLED SH1106 128×64  │
                                  │  Judul · Lirik ·     │
                                  │  Progress · Volume   │
                                  └──────────────────────┘
```

---

## 🧰 Yang Dibutuhkan

### Hardware

| Komponen | Jumlah | Keterangan |
|----------|--------|------------|
| ESP32-S3 Dev Board | 1 | Board apapun dengan USB CDC built-in |
| OLED SH1106 128×64 | 1 | Interface I2C, 3.3V. Kompatibel juga dengan SH1107 |
| Kabel Jumper | 4 | Female-to-female |
| Kabel USB | 1 | USB-C/Micro ke PC untuk Serial |

### Software

| Tools | Kegunaan |
|-------|----------|
| Python 3.8+ | Menjalankan `main.py` di PC |
| Arduino IDE 2.x | Upload sketch ke ESP32-S3 |
| Discord Desktop | Diperlukan untuk Rich Presence (bukan versi web) |
| Akun Spotify | Versi free/premium, keduanya bisa |

---

## 🔌 Wiring

### OLED SH1106 → ESP32-S3

| Pin OLED | Pin ESP32-S3 | Warna Kabel (saran) |
|----------|-------------|---------------------|
| VCC | **3.3V** ⚠️ jangan 5V | Merah |
| GND | GND | Hitam |
| SCL | **GPIO 9** | Kuning |
| SDA | **GPIO 8** | Biru |

> ⚠️ **Penting:** OLED SH1106 bekerja di 3.3V. Menghubungkan ke 5V bisa merusaknya secara permanen.

### Diagram Fisik

```
   OLED SH1106                      ESP32-S3
   ┌──────────┐                   ┌────────────────┐
   │          │                   │                │
   │  VCC  ●──┼──── 3.3V ─────────┼─● 3V3          │
   │          │                   │                │
   │  GND  ●──┼──── GND  ─────────┼─● GND          │
   │          │                   │                │
   │  SCL  ●──┼──── I2C CLK ──────┼─● GPIO 9       │
   │          │                   │                │
   │  SDA  ●──┼──── I2C DAT ──────┼─● GPIO 8       │
   │          │                   │                │
   └──────────┘                   │  USB ──────────┼──► PC
                                  └────────────────┘

   I2C Speed  : 400kHz (Fast Mode)
   I2C Address: 0x3C (default) atau 0x3D
```

> Jika OLED menggunakan pin berbeda, ubah `PIN_SDA` dan `PIN_SCL` di file `.ino`.

---

## 📁 Struktur Proyek

```
spotify-oled/
├── main.py                        # Script PC utama
├── requirements.txt               # Dependensi Python
├── .env                           # Kredensial (JANGAN di-commit!)
├── .env.example                   # Template konfigurasi
├── .gitignore
├── .spotify_cache                 # Token Spotify (auto-generated)
└── spotify_oled_esp32s3/
    └── spotify_oled_esp32s3.ino   # Sketch Arduino ESP32-S3
```

---

## ⚙️ Setup & Instalasi

### 1. Clone Repo

```bash
git clone https://github.com/username/spotify-oled.git
cd spotify-oled
```

### 2. Buat Spotify App

1. Buka [Spotify Developer Dashboard](https://developer.spotify.com/dashboard)
2. Klik **Create App**
3. Isi nama bebas, tambahkan Redirect URI: `http://127.0.0.1:8888/callback`
4. Salin **Client ID** dan **Client Secret**

### 3. Buat Discord Application

1. Buka [Discord Developer Portal](https://discord.com/developers/applications)
2. Klik **New Application**, beri nama (contoh: `Spotify OLED`)
3. Salin **Application ID** → ini adalah `DISCORD_CLIENT_ID`
4. *(Opsional)* Tab **Rich Presence → Art Assets**: upload gambar `play`, `pause`, `spotify_logo`

### 4. Konfigurasi `.env`

```bash
cp .env.example .env
# Edit .env dan isi semua nilai
```

### 5. Install Python Dependencies

```bash
pip install -r requirements.txt
```

### 6. Setup Arduino IDE

1. Tambahkan board ESP32 via **Board Manager**:
   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```
2. Install library via **Library Manager**:
   - **U8g2** by olikraus
   - **ArduinoJson** by Benoît Blanchon

3. Setting board:

   | Setting | Nilai |
   |---------|-------|
   | Board | `ESP32S3 Dev Module` |
   | USB Mode | `CDC and JTAG` |
   | Upload Speed | `921600` |

4. Buka `spotify_oled_esp32s3/spotify_oled_esp32s3.ino` → klik **Upload**

> 💡 Jika gagal upload, tahan tombol **BOOT** saat klik Upload, lepas setelah `Connecting...` muncul.

### 7. Jalankan

```bash
python main.py
```

Pertama kali: browser otomatis terbuka untuk login Spotify → klik **Allow** → token disimpan di `.spotify_cache`.

---

## 📄 Kode

<details>
<summary><b>📂 main.py</b> — klik untuk lihat</summary>

```python
"""
Spotify Discord ESP32 Client — dengan Lirik
============================================
Mengambil info lagu dari Spotify, menampilkannya di Discord Rich Presence,
mengirimkan data + lirik synced ke ESP32 via Serial.
"""

import json
import os
import re
import sys
import time
import logging
import threading
from dataclasses import dataclass
from typing import Optional

import serial
import serial.tools.list_ports
import spotipy
import syncedlyrics
from dotenv import load_dotenv
from pypresence import Presence, InvalidID, PipeClosed
from spotipy.oauth2 import SpotifyOAuth

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
    datefmt="%H:%M:%S",
)
log = logging.getLogger(__name__)

load_dotenv()

SPOTIFY_CLIENT_ID     = os.getenv("SPOTIFY_CLIENT_ID", "")
SPOTIFY_CLIENT_SECRET = os.getenv("SPOTIFY_CLIENT_SECRET", "")
SPOTIFY_REDIRECT_URI  = os.getenv("SPOTIFY_REDIRECT_URI", "http://127.0.0.1:8888/callback")
DISCORD_CLIENT_ID     = os.getenv("DISCORD_CLIENT_ID", "")
ESP32_PORT            = os.getenv("ESP32_PORT", "COM4")
ESP32_BAUD            = int(os.getenv("ESP32_BAUD", "115200"))
UPDATE_INTERVAL       = int(os.getenv("UPDATE_INTERVAL", "1"))

SPOTIFY_SCOPE = (
    "user-read-playback-state "
    "user-read-currently-playing "
    "user-read-playback-position"
)


@dataclass
class TrackInfo:
    title: str
    artist: str
    album: str
    album_art_url: str
    is_playing: bool
    progress_ms: int
    duration_ms: int
    volume: int
    track_url: str
    lyric: str      = ""
    next_lyric: str = ""

    def to_json(self) -> str:
        return json.dumps({
            "title":       self.title,
            "artist":      self.artist,
            "album":       self.album,
            "is_playing":  self.is_playing,
            "progress_ms": self.progress_ms,
            "duration_ms": self.duration_ms,
            "volume":      self.volume,
            "lyric":       self.lyric,
            "next_lyric":  self.next_lyric,
        }, ensure_ascii=False)

    def progress_str(self) -> str:
        cur_s = self.progress_ms // 1000
        tot_s = self.duration_ms // 1000
        return f"{cur_s // 60}:{cur_s % 60:02d} / {tot_s // 60}:{tot_s % 60:02d}"


class LyricsEngine:
    def __init__(self) -> None:
        self._cache_key: str                = ""
        self._lyrics: list[tuple[int, str]] = []
        self._lock = threading.Lock()

    @staticmethod
    def _parse_lrc(lrc_text: str) -> list[tuple[int, str]]:
        pattern = re.compile(r"\[(\d+):(\d+\.\d+)\](.*)")
        lines: list[tuple[int, str]] = []
        for line in lrc_text.splitlines():
            m = pattern.match(line.strip())
            if m:
                ms   = int((int(m.group(1)) * 60 + float(m.group(2))) * 1000)
                text = m.group(3).strip()
                if text:
                    lines.append((ms, text))
        return sorted(lines, key=lambda x: x[0])

    def load(self, title: str, artist: str) -> None:
        key = f"{title}|{artist}"
        with self._lock:
            if key == self._cache_key:
                return
            self._cache_key = key
            self._lyrics    = []
        log.info(f"[Lyrics] Mencari: {title} — {artist}")
        try:
            lrc = syncedlyrics.search(f"{title} {artist}")
            if lrc:
                parsed = self._parse_lrc(lrc)
                with self._lock:
                    self._lyrics = parsed
                log.info(f"[Lyrics] {len(parsed)} baris ditemukan.")
            else:
                log.info("[Lyrics] Tidak ditemukan.")
        except Exception as exc:
            log.warning(f"[Lyrics] Error: {exc}")

    def get_current(self, progress_ms: int) -> tuple[str, str]:
        with self._lock:
            lyrics = self._lyrics
        current = ""
        nxt     = ""
        for i, (ts, text) in enumerate(lyrics):
            if ts <= progress_ms:
                current = text
                nxt     = lyrics[i + 1][1] if i + 1 < len(lyrics) else ""
            else:
                break
        return current, nxt

    def clear(self) -> None:
        with self._lock:
            self._cache_key = ""
            self._lyrics    = []


class SpotifyClient:
    def __init__(self) -> None:
        if not SPOTIFY_CLIENT_ID or not SPOTIFY_CLIENT_SECRET:
            log.error("SPOTIFY_CLIENT_ID atau SPOTIFY_CLIENT_SECRET belum diisi di .env!")
            sys.exit(1)
        auth = SpotifyOAuth(
            client_id=SPOTIFY_CLIENT_ID,
            client_secret=SPOTIFY_CLIENT_SECRET,
            redirect_uri=SPOTIFY_REDIRECT_URI,
            scope=SPOTIFY_SCOPE,
            open_browser=True,
            cache_path=".spotify_cache",
        )
        self.sp = spotipy.Spotify(auth_manager=auth)
        log.info("Spotify client berhasil dibuat.")

    def get_current_track(self) -> Optional[TrackInfo]:
        try:
            playback = self.sp.current_playback()
        except Exception as exc:
            log.warning(f"Gagal mengambil data Spotify: {exc}")
            return None
        if not playback or not playback.get("item"):
            return None
        item     = playback["item"]
        artists  = ", ".join(a["name"] for a in item.get("artists", []))
        album    = item.get("album", {})
        images   = album.get("images", [])
        art_url  = images[0]["url"] if images else ""
        ext_urls = item.get("external_urls", {})
        device   = playback.get("device", {})
        volume   = device.get("volume_percent", 0) if device else 0
        return TrackInfo(
            title         = item.get("name", "Unknown"),
            artist        = artists or "Unknown Artist",
            album         = album.get("name", "Unknown Album"),
            album_art_url = art_url,
            is_playing    = playback.get("is_playing", False),
            progress_ms   = playback.get("progress_ms", 0),
            duration_ms   = item.get("duration_ms", 0),
            volume        = volume,
            track_url     = ext_urls.get("spotify", ""),
        )


class DiscordPresence:
    def __init__(self) -> None:
        self.rpc: Optional[Presence] = None
        self._connected  = False
        self._last_track: Optional[str] = None
        if not DISCORD_CLIENT_ID:
            log.warning("DISCORD_CLIENT_ID kosong. Discord Rich Presence dinonaktifkan.")
            return
        self._connect()

    def _connect(self) -> None:
        try:
            self.rpc = Presence(DISCORD_CLIENT_ID)
            self.rpc.connect()
            self._connected = True
            log.info("Discord Rich Presence terhubung.")
        except (InvalidID, ConnectionRefusedError, FileNotFoundError):
            log.warning("Discord tidak bisa dijangkau.")
            self._connected = False
        except Exception as exc:
            log.warning(f"Discord error: {exc}")
            self._connected = False

    def update(self, track: Optional[TrackInfo]) -> None:
        if not self._connected or self.rpc is None:
            return
        try:
            if track is None:
                self.rpc.clear()
                self._last_track = None
                return
            track_key = f"{track.title}|{track.artist}|{track.is_playing}"
            if track_key == self._last_track:
                return
            kwargs: dict = {
                "details":     track.title[:128],
                "state":       f"oleh {track.artist}"[:128],
                "large_image": track.album_art_url or "spotify_logo",
                "large_text":  track.album,
                "small_image": "play" if track.is_playing else "pause",
                "small_text":  "Sedang diputar" if track.is_playing else "Dijeda",
            }
            if track.track_url:
                kwargs["buttons"] = [{"label": "Buka di Spotify", "url": track.track_url}]
            self.rpc.update(**kwargs)
            self._last_track = track_key
            log.info(f"Discord: {track.title} ({'▶' if track.is_playing else '⏸'})")
        except PipeClosed:
            log.warning("Discord terputus. Reconnect...")
            self._connected = False
            self._connect()
        except Exception as exc:
            log.warning(f"Gagal update Discord: {exc}")

    def close(self) -> None:
        if self._connected and self.rpc:
            try:
                self.rpc.clear()
                self.rpc.close()
            except Exception:
                pass


class ESP32Serial:
    def __init__(self) -> None:
        self.ser: Optional[serial.Serial] = None
        if not ESP32_PORT:
            return
        self._connect()

    def _connect(self) -> None:
        try:
            self.ser = serial.Serial(port=ESP32_PORT, baudrate=ESP32_BAUD, timeout=1)
            time.sleep(2)
            log.info(f"ESP32 terhubung di {ESP32_PORT} ({ESP32_BAUD} baud).")
        except serial.SerialException as exc:
            log.warning(f"Gagal buka port {ESP32_PORT}: {exc}")
            ports = list(serial.tools.list_ports.comports())
            if ports:
                log.info("Port tersedia: " + ", ".join(p.device for p in ports))
            self.ser = None

    def send(self, track: Optional[TrackInfo]) -> None:
        if self.ser is None or not self.ser.is_open:
            return
        try:
            payload = track.to_json() if track else json.dumps({
                "title": "Tidak ada lagu", "artist": "", "album": "",
                "is_playing": False, "progress_ms": 0, "duration_ms": 0,
                "volume": 0, "lyric": "", "next_lyric": "",
            })
            self.ser.write((payload + "\n").encode("utf-8"))
        except serial.SerialException as exc:
            log.warning(f"Gagal kirim ke ESP32: {exc}")

    def close(self) -> None:
        if self.ser and self.ser.is_open:
            self.ser.close()


def main() -> None:
    log.info("=" * 50)
    log.info("  Spotify Discord ESP32 Client  [+Lyrics]")
    log.info("=" * 50)

    spotify = SpotifyClient()
    discord = DiscordPresence()
    esp32   = ESP32Serial()
    lyrics  = LyricsEngine()

    log.info(f"Update interval: {UPDATE_INTERVAL} detik | Tekan Ctrl+C untuk berhenti.\n")

    last_track_key = ""

    try:
        while True:
            track = spotify.get_current_track()
            if track:
                track_key = f"{track.title}|{track.artist}"
                if track_key != last_track_key:
                    last_track_key = track_key
                    threading.Thread(
                        target=lyrics.load,
                        args=(track.title, track.artist),
                        daemon=True,
                    ).start()
                if track.is_playing:
                    track.lyric, track.next_lyric = lyrics.get_current(track.progress_ms)
                else:
                    track.lyric = track.next_lyric = ""
                log.info(
                    f"{'▶' if track.is_playing else '⏸'} "
                    f"{track.title} - {track.artist} "
                    f"[{track.progress_str()}] Vol:{track.volume}%"
                )
                if track.lyric:
                    log.info(f"🎵 {track.lyric}")
            else:
                log.info("Tidak ada lagu.")
                last_track_key = ""
                lyrics.clear()

            discord.update(track)
            esp32.send(track)
            time.sleep(UPDATE_INTERVAL)

    except KeyboardInterrupt:
        log.info("\nDihentikan.")
    finally:
        discord.close()
        esp32.close()


if __name__ == "__main__":
    main()
```

</details>

<details>
<summary><b>📂 spotify_oled_esp32s3.ino</b> — klik untuk lihat</summary>

```cpp
/*
 ================================================================
   Spotify + Lirik Display — ESP32-S3 + OLED SH1106 (128x64)
   Library: U8g2 (olikraus) + ArduinoJson (Blanchon)
   Wiring : VCC->3.3V | GND->GND | SCL->GPIO9 | SDA->GPIO8
 ================================================================
*/

#include <Arduino.h>
#include <Wire.h>
#include <U8g2lib.h>
#include <ArduinoJson.h>

#define PIN_SDA  8
#define PIN_SCL  9

// Ganti ke U8G2_SH1107_... jika pakai SH1107
U8G2_SH1106_128X64_NONAME_F_HW_I2C
    u8g2(U8G2_R0, U8X8_PIN_NONE, PIN_SCL, PIN_SDA);

struct SpotifyData {
    String title       = "";
    String artist      = "";
    String album       = "";
    bool   is_playing  = false;
    long   progress_ms = 0;
    long   duration_ms = 1;
    int    volume      = 0;
    String lyric       = "";
    String next_lyric  = "";
};

SpotifyData track;

String   serialBuf        = "";
String   prevLyric        = "";
String   prevTitle        = "";
bool     titleNeedsScroll = false;
int      titleScrollX     = 128;
uint32_t lastMillis       = 0;
uint32_t lastScroll       = 0;
uint32_t lastFrame        = 0;
uint32_t lastSerial       = 0;

#define SCROLL_INTERVAL  55
#define FRAME_INTERVAL   33
#define SERIAL_TIMEOUT   5000

String trunc(const String& s, uint8_t n) {
    if ((int)s.length() <= n) return s;
    return s.substring(0, n - 1) + "~";
}

void drawProgressBar(long prog, long dur) {
    const int X = 0, Y = 60, W = 110, H = 3;
    u8g2.drawRFrame(X, Y, W, H, 1);
    if (dur > 0) {
        int f = (int)((float)prog / dur * W);
        if (f > 0) u8g2.drawRBox(X, Y, f, H, 1);
    }
    char buf[8];
    long cur = prog / 1000;
    snprintf(buf, sizeof(buf), "%ld:%02ld", cur / 60, cur % 60);
    u8g2.setFont(u8g2_font_4x6_tf);
    u8g2.drawStr(112, 64, buf);
}

void drawSplash() {
    u8g2.clearBuffer();
    u8g2.setFont(u8g2_font_7x13B_tf);
    u8g2.drawStr(14, 22, "Spotify OLED");
    u8g2.setFont(u8g2_font_5x7_tf);
    u8g2.drawStr(22, 36, "ESP32-S3  Ready");
    u8g2.drawStr(8, 52, "Menunggu koneksi PC...");
    u8g2.sendBuffer();
}

void drawIdle() {
    u8g2.clearBuffer();
    u8g2.setFont(u8g2_font_6x10_tf);
    u8g2.drawStr(14, 26, "Tidak ada lagu");
    u8g2.setFont(u8g2_font_5x7_tf);
    u8g2.drawStr(10, 40, "Putar lagu di Spotify");
    u8g2.drawLine(56, 50, 56, 57);
    u8g2.drawLine(59, 48, 59, 55);
    u8g2.drawLine(56, 50, 59, 48);
    u8g2.drawDisc(54, 57, 2);
    u8g2.drawDisc(57, 55, 2);
    u8g2.sendBuffer();
}

void drawDisconnect() {
    u8g2.clearBuffer();
    u8g2.setFont(u8g2_font_5x7_tf);
    u8g2.drawStr(20, 24, "PC tidak terhubung");
    u8g2.drawStr(16, 38, "Jalankan main.py");
    u8g2.sendBuffer();
}

void drawMain() {
    u8g2.clearBuffer();

    if (track.is_playing) {
        u8g2.drawTriangle(0, 2, 0, 10, 6, 6);
    } else {
        u8g2.drawBox(0, 2, 2, 8);
        u8g2.drawBox(4, 2, 2, 8);
    }

    u8g2.setFont(u8g2_font_7x13B_tf);
    if (titleNeedsScroll) {
        u8g2.setClipWindow(9, 0, 127, 13);
        u8g2.drawStr(titleScrollX, 12, track.title.c_str());
        u8g2.setMaxClipWindow();
    } else {
        u8g2.drawStr(9, 12, track.title.c_str());
    }

    u8g2.setFont(u8g2_font_5x7_tf);
    u8g2.drawStr(0, 22, trunc(track.artist, 20).c_str());
    char volBuf[6];
    snprintf(volBuf, sizeof(volBuf), "%d%%", track.volume);
    u8g2.drawStr(104, 22, volBuf);

    u8g2.drawHLine(0, 24, 128);

    if (track.lyric.length() > 0) {
        u8g2.setFont(u8g2_font_6x10_tf);
        const int MAX_CH = 21;
        String l1, l2;
        if ((int)track.lyric.length() <= MAX_CH) {
            l1 = track.lyric; l2 = "";
        } else {
            int split = track.lyric.lastIndexOf(' ', MAX_CH);
            if (split <= 0) split = MAX_CH;
            l1 = track.lyric.substring(0, split);
            l2 = track.lyric.substring(split + 1);
            if ((int)l2.length() > MAX_CH)
                l2 = l2.substring(0, MAX_CH - 1) + "~";
        }
        int baseY = (l2.length() == 0) ? 40 : 35;
        u8g2.drawStr(0, baseY, l1.c_str());
        if (l2.length() > 0)
            u8g2.drawStr(0, baseY + 11, l2.c_str());
        if (track.next_lyric.length() > 0) {
            u8g2.setFont(u8g2_font_4x6_tf);
            u8g2.drawStr(0, 56, ("  " + trunc(track.next_lyric, 32)).c_str());
        }
    } else {
        u8g2.setFont(u8g2_font_5x7_tf);
        u8g2.drawStr(24, 42, "~ instrumental ~");
    }

    u8g2.drawHLine(0, 58, 128);
    drawProgressBar(track.progress_ms, track.duration_ms);
    u8g2.sendBuffer();
}

void parseJson(const String& raw) {
    StaticJsonDocument<512> doc;
    DeserializationError err = deserializeJson(doc, raw);
    if (err) { Serial.printf("[JSON] Err: %s\n", err.c_str()); return; }

    track.title       = doc["title"]       | "";
    track.artist      = doc["artist"]      | "";
    track.album       = doc["album"]       | "";
    track.is_playing  = doc["is_playing"]  | false;
    track.progress_ms = doc["progress_ms"] | 0;
    track.duration_ms = doc["duration_ms"] | 1;
    track.volume      = doc["volume"]      | 0;

    String newLyric   = doc["lyric"]       | "";
    if (newLyric != prevLyric) prevLyric = newLyric;
    track.lyric       = newLyric;
    track.next_lyric  = doc["next_lyric"]  | "";

    if (track.title != prevTitle) {
        prevTitle    = track.title;
        titleScrollX = 128;
        u8g2.setFont(u8g2_font_7x13B_tf);
        titleNeedsScroll = (u8g2.getStrWidth(track.title.c_str()) > 118);
    }
    lastSerial = millis();
}

void setup() {
    Serial.begin(115200);
    Wire.begin(PIN_SDA, PIN_SCL);
    Wire.setClock(400000);
    u8g2.begin();
    u8g2.setContrast(220);
    drawSplash();
    delay(2000);
    lastMillis = lastSerial = millis();
}

void loop() {
    uint32_t ms = millis();

    while (Serial.available()) {
        char c = (char)Serial.read();
        if (c == '\n') {
            serialBuf.trim();
            if (serialBuf.length() > 2) parseJson(serialBuf);
            serialBuf = "";
        } else {
            serialBuf += c;
            if (serialBuf.length() > 700) serialBuf = "";
        }
    }

    if (track.is_playing) {
        track.progress_ms += (long)(ms - lastMillis);
        if (track.progress_ms > track.duration_ms)
            track.progress_ms = track.duration_ms;
    }
    lastMillis = ms;

    if (ms - lastScroll >= SCROLL_INTERVAL) {
        lastScroll = ms;
        if (titleNeedsScroll) {
            titleScrollX -= 2;
            u8g2.setFont(u8g2_font_7x13B_tf);
            int w = u8g2.getStrWidth(track.title.c_str());
            if (titleScrollX < -(w + 20)) titleScrollX = 128;
        }
    }

    if (ms - lastFrame >= FRAME_INTERVAL) {
        lastFrame = ms;
        bool dc = (ms - lastSerial > SERIAL_TIMEOUT);
        if (dc)                                           drawDisconnect();
        else if (!track.is_playing && track.title == "")  drawIdle();
        else                                              drawMain();
    }
}
```

</details>

<details>
<summary><b>📂 requirements.txt</b></summary>

```
spotipy
pypresence
pyserial
python-dotenv
syncedlyrics
```

</details>

<details>
<summary><b>📂 .env.example</b></summary>

```env
# Spotify — https://developer.spotify.com/dashboard
SPOTIFY_CLIENT_ID=your_client_id_here
SPOTIFY_CLIENT_SECRET=your_client_secret_here
SPOTIFY_REDIRECT_URI=http://127.0.0.1:8888/callback

# Discord — https://discord.com/developers/applications
DISCORD_CLIENT_ID=your_application_id_here

# ESP32 Serial Port
# Windows : COM3, COM4, ...
# Linux   : /dev/ttyUSB0, /dev/ttyACM0, ...
ESP32_PORT=COM4
ESP32_BAUD=115200

# Interval polling Spotify (detik)
UPDATE_INTERVAL=1
```

</details>

<details>
<summary><b>📂 .gitignore</b></summary>

```gitignore
# Kredensial — JANGAN pernah di-commit
.env
.spotify_cache

# Python
__pycache__/
*.pyc
*.pyo
*.egg-info/
dist/
build/
.venv/
venv/

# Arduino / IDE
*.suo
*.user
build/
.pio/
```

</details>

---

## 🔗 Format JSON Serial (PC → ESP32-S3)

Dikirim setiap `UPDATE_INTERVAL` detik via USB Serial, diakhiri `\n`:

```json
{
  "title":       "Shape of You",
  "artist":      "Ed Sheeran",
  "album":       "÷ (Divide)",
  "is_playing":  true,
  "progress_ms": 32000,
  "duration_ms": 234000,
  "volume":      80,
  "lyric":       "I'm in love with your body",
  "next_lyric":  "Oh I, oh I, oh I..."
}
```

---

## 🏗️ Arsitektur

### `main.py` — Class Diagram

```
main.py
├── TrackInfo (dataclass)
│   ├── title, artist, album, album_art_url, track_url
│   ├── is_playing, progress_ms, duration_ms, volume
│   ├── lyric, next_lyric          ← field lirik
│   └── to_json() → str            ← serialisasi untuk ESP32
│
├── LyricsEngine
│   ├── load(title, artist)        ← background thread, cache per lagu
│   ├── get_current(progress_ms)   ← (lirik_sekarang, lirik_berikutnya)
│   └── _parse_lrc(text)           ← parse .lrc → [(ms, teks)]
│
├── SpotifyClient
│   └── get_current_track()        ← return TrackInfo | None
│
├── DiscordPresence
│   ├── update(track)              ← update Rich Presence
│   └── close()
│
├── ESP32Serial
│   ├── send(track)                ← kirim JSON + \n via Serial
│   └── close()
│
└── main()                         ← loop utama
```

### `spotify_oled_esp32s3.ino` — State Machine Layar

```
loop()
 ├── Baca Serial char-by-char → parseJson() saat \n
 │
 ├── Interpolasi progress_ms lokal (antar kiriman Serial)
 │
 ├── Scroll judul setiap 55ms
 │
 └── Render frame setiap 33ms (~30 fps)
       ├── timeout > 5s       → drawDisconnect()
       ├── no title & paused  → drawIdle()
       └── normal             → drawMain()
             ├── ▶/⏸ icon
             ├── Judul (clip + scroll otomatis)
             ├── Artist + Volume%
             ├── ── garis pemisah ──
             ├── Lirik (word-wrap 2 baris, font 6x10)
             ├── Preview lirik berikutnya (font 4x6)
             ├── ── garis bawah ──
             └── Progress bar rounded + mm:ss
```

---

## 🔧 Troubleshooting

| Masalah | Solusi |
|---------|--------|
| 🖥️ OLED blank/gelap | Cek wiring SDA/SCL. Coba I2C address `0x3D` (ubah di constructor U8g2). Pastikan VCC ke **3.3V** |
| 🔌 Serial tidak konek | Cek `ESP32_PORT` di `.env`. Windows: Device Manager → Ports. Linux: `ls /dev/tty*` |
| 🎤 Lirik tidak muncul | `syncedlyrics` bergantung database online. Lagu tidak populer / lokal mungkin tidak tersedia |
| 🎮 Discord tidak update | Pastikan **Discord Desktop** berjalan (bukan versi web). `DISCORD_CLIENT_ID` harus Application ID |
| 🔑 Login Spotify gagal | Redirect URI di `.env` harus **sama persis** dengan yang di Spotify Dashboard |
| 📺 OLED tampil disconnect | `main.py` belum jalan atau `ESP32_PORT` salah. Timeout 5 detik tanpa data Serial |
| ⬆️ Upload ESP32 gagal | Tahan tombol **BOOT** saat Upload, lepas setelah `Connecting...` |

---

## 📦 Dependensi

| Package | Fungsi |
|---------|--------|
| `spotipy` | Spotify Web API client + OAuth |
| `pypresence` | Discord Rich Presence via IPC pipe |
| `pyserial` | Komunikasi USB Serial ke ESP32 |
| `python-dotenv` | Load konfigurasi dari file `.env` |
| `syncedlyrics` | Ambil synced lyrics format `.lrc` |

---

## 📝 Lisensi

MIT License — bebas digunakan, dimodifikasi, dan didistribusikan.

---

<div align="center">

Dibuat dengan ❤️ &nbsp;·&nbsp; ESP32-S3 + OLED SH1106 + Spotify API + Discord Rich Presence

</div>
