## OpenClaw AI Gateway - Windows WSL Ubuntu Kurulum Rehberi



📋 İçindekiler

Ön Gereksinimler

1. WSL2 Ubuntu Kurulumu

2. OpenClaw Ön Gereksinimleri

3. OpenClaw Kurulumu

4. İlk Çalıştırma ve Yapılandırma

5. Telegram Entegrasyonu

6. Ücretsiz ChatGPT ile Kullanım

7. Sorun Giderme

8. Güvenlik Önlemleri

9. Günlük Kullanım Komutları

---

🔧 Ön Gereksinimler
| Gereksinim      | Açıklama                        |
| --------------- | ------------------------------- |
| Windows 10 / 11 | Build 19041 veya üzeri          |
| WSL2            | Windows Subsystem for Linux v2  |
| İnternet        | Model API erişimi için          |
| ChatGPT hesabı  | Ücretsiz veya Plus              |
| Telegram hesabı | Bot bağlantısı için (opsiyonel) |

---

# 1. WSL2 Ubuntu Kurulumu
1.1 Windows'ta WSL2'yi Etkinleştirme
PowerShell'i Yönetici olarak açın ve şu komutları çalıştırın:

```
# WSL özelliğini etkinleştir
``` dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart ```

# Sanal makine platformunu etkinleştir
``` dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart ```

# Bilgisayarı yeniden başlat
Restart-Computer
```

1.2 WSL2'yi Varsayılan Sürüm Yapma
Yeniden başlattıktan sonra PowerShell'i tekrar yönetici olarak açın:

```
# WSL2'yi varsayılan sürüm yap
wsl --set-default-version 2

# Linux kernel güncellemesi (gerekirse)
# https://aka.ms/wsl2kernel adresinden indirip kurun
```


1.3 Ubuntu Kurulumu

```
# Microsoft Store'dan veya komut satırından Ubuntu 22.04 LTS kurun
wsl --install -d Ubuntu-22.04

# Kurulum tamamlandıktan sonra başlat
wsl -d Ubuntu-22.04
```

1.4 Ubuntu İlk Kurulum
İlk açılışta sizden kullanıcı adı ve şifre istenecek. Örnek:

```
Enter new UNIX username: khenzar
New password: ******
Retype new password: ******
```

ÖNEMLİ: Bu kullanıcı adını ve şifreyi not edin, her sudo komutunda lazım olacak.

---

# 2. OpenClaw Ön Gereksinimleri
WSL Ubuntu terminalinde aşağıdaki komutları sırayla çalıştırın:

2.1 Sistem Güncellemeleri ve Temel Araçlar


Paket listesini güncelle
```
sudo apt update && sudo apt upgrade -y
```

Temel araçları kur
```
sudo apt install -y curl wget git build-essential
```

Node.js kaynağını ekle (Node.js 22.x için)
```
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
```

Node.js ve npm'i kur
```
sudo apt install -y nodejs
```

Node.js sürümünü kontrol et
```
node --version  # v22.x.x olmalı
```

Screen kur (arka planda çalıştırmak için)
```
sudo apt install -y screen
```

jq kur (JSON işlemleri için - opsiyonel)
```
sudo apt install -y jq
```

2.2 npm Ayarları

npm'i güncelle
```
sudo npm install -g npm@latest
```

Permissions sorunu yaşamamak için (opsiyonel)
```
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

---

# 3. OpenClaw Kurulumu
3.1 OpenClaw CLI Kurulumu


OpenClaw'u global olarak kur
```
npm install -g openclaw@latest
```

Kurulumu kontrol et
```
openclaw --version
```

3.2 OpenClaw Onboarding (İlk Yapılandırma)

Onboarding'i başlat
```
openclaw onboard
```

Onboarding sırasında karşınıza çıkacak seçenekler:

1. Güvenlik uyarısı: Yes ile devam edin

2. Auth provider: API key seçin (şimdilik, sonra değiştireceğiz)

3. API key: Geçici bir şey yazın (12345 gibi, sonra düzelteceğiz)

4. Bind address: loopback seçin (güvenlik için)

5. Port: Varsayılan 18789'u kullanın

6. Install daemon: Yes seçin

7. Workspace path: Varsayılanı kabul edin (Enter)

8. Skills: No seçin (güvenlik için)

9. Hooks: Skip for now seçin

10. Web search: Skip for now seçin

Not: Onboarding sonunda hata alabilirsiniz (systemd ile ilgili). Bu normaldir, sonraki adımda düzelteceğiz.

---

# 4. İlk Çalıştırma ve Yapılandırma
4.1 Gateway'i Screen ile Çalıştırma (systemd alternatifi)

Yeni screen oturumu oluştur
```
screen -S openclaw
```

Gateway'i başlat
```
openclaw gateway --port 18789
```

Screen'den ayrılmak için: CTRL+A, D
Tekrar bağlanmak için: 
```
screen -r openclaw
```

4.2 Gateway Sağlık Kontrolü
```
# Gateway durumunu kontrol et
openclaw health
openclaw status
```

4.3 Auth Profilini ChatGPT ile Değiştirme
```
# OpenAI Codex auth profili ekle
openclaw auth add-profile openai-codex:default --provider openai-codex --mode oauth
```
Bu komut sizi tarayıcıda OpenAI login sayfasına yönlendirecek. ChatGPT hesabınızla giriş yapın ve yetki verin.

4.4 Model Yapılandırması
```
# models.json dosyasını düzenle
nano ~/.openclaw/agents/main/agent/models.json
```

Dosyaya şu içeriği yapıştırın:
```
{
  "providers": {
    "openai-codex": {
      "baseUrl": "https://chatgpt.com/backend-api",
      "api": "openai-codex-responses",
      "models": [
        {
          "id": "gpt-5.2-codex",
          "name": "GPT-5.2 Codex",
          "capabilities": ["chat", "codex"],
          "context": 272000,
          "maxTokens": 128000
        },
        {
          "id": "gpt-5.1-codex-max",
          "name": "GPT-5.1 Codex Max",
          "capabilities": ["chat", "codex"],
          "context": 272000,
          "maxTokens": 128000
        },
        {
          "id": "gpt-5.1-codex-mini",
          "name": "GPT-5.1 Codex Mini",
          "capabilities": ["chat", "codex"],
          "context": 272000,
          "maxTokens": 128000
        }
      ]
    }
  },
  "defaultProvider": "openai-codex",
  "defaultModel": "gpt-5.2-codex"
}
```

Kaydedin: CTRL+X, Y, Enter

4.5 Ana Konfigürasyonu Düzenleme
```
# Ana config dosyasını düzenle
nano ~/.openclaw/openclaw.json
```
Dosyada "primary" satırını bulup şöyle değiştirin:

```
        "primary": "openai-codex/gpt-5.2-codex"
```


4.6 Gateway'i Yeniden Başlatma

```
# Screen'e bağlan
screen -r openclaw

# CTRL+C ile durdur
# Tekrar başlat:
openclaw gateway --port 18789

# Screen'den ayrıl: CTRL+A, D
```

---

# 5. Telegram Entegrasyonu
5.1 Telegram Botu Oluşturma
1. Telegram'da @BotFather ile konuşun

2. /newbot komutunu gönderin

3. Bot için bir isim verin (ör: Benim Asistanım)

4. Bot için bir kullanıcı adı verin (ör: benim_asistan_bot)

5. Size verilen token'ı not edin


5.2 Telegram Botunu OpenClaw'a Bağlama
```
# Telegram kanalını yapılandır
openclaw configure

# Sihirbazda şunları seçin:
# - Telegram'ı seçin
# - Bot token'ını girin
# - DM policy: "allowlist" seçin
```

5.3 Gateway'i Tekrar Başlatma
```
screen -r openclaw
# CTRL+C
openclaw gateway --port 18789
# CTRL+A, D
```

5.4 Telegram'da Botu Test Etme
Telegram'da botunuza mesaj gönderin. İlk mesajda yanıt almazsanız:

```
# Bekleyen pairing isteklerini listele
openclaw pairing list telegram

# İsteği onayla (XXX yerine gelen kodu yazın)
openclaw pairing approve telegram XXXXXXXX
```

---

# 6. Ücretsiz ChatGPT ile Kullanım
OpenClaw, ChatGPT hesabınızın ücretsiz kotası ile çalışır. Ek ücret ödemeniz gerekmez.

6.1 Kullanılabilecek Modeller

Model				Açıklama
gpt-5.2-codex			En güncel ve kararlı model
gpt-5.1-codex-max		Yüksek kapasiteli, uzun sohbetler için
gpt-5.1-codex-mini		Hafif ve hızlı, basit görevler için

6.2 Model Değiştirme (İhtiyaç halinde)
```
# Model değiştir
nano ~/.openclaw/agents/main/agent/models.json
# defaultModel değerini değiştirin

# Ana config'de de değiştirin
nano ~/.openclaw/openclaw.json
# primary değerini değiştirin

# Gateway'i restart edin
screen -r openclaw
# CTRL+C
openclaw gateway --port 18789
# CTRL+A, D
```

---

# 7. Sorun Giderme
7.1 Sık Karşılaşılan Hatalar
Hata					Çözüm
Gateway service install failed		Screen kullanın (Adım 4.1)
"model is not supported"		Model adını gpt-5.2-codex yapın
Bot yanıt vermiyor			Pairing isteğini onaylayın
Web UI bağlanmıyor			Gateway'in çalıştığından emin olun


7.2 Log İzleme
```
# Logları canlı izle
openclaw logs --follow
```


7.3 Gateway'i Sıfırlama
```
# Tüm konfigürasyonu yedekle
cp -r ~/.openclaw ~/.openclaw.yedek

# Gateway'i durdur (screen'de CTRL+C)
# Screen'i sonlandır
screen -X -S openclaw quit

# Yeniden başlat
screen -S openclaw
openclaw gateway --port 18789
```

---

# 8. Güvenlik Önlemleri
8.1 Zorunlu Güvenlik Ayarları

```
# Gateway'i sadece localhost'ta dinle (kontrol)
openclaw config set gateway.bind loopback

# Token authentication ekle
openclaw config set gateway.auth.mode token
openclaw config set gateway.auth.token "$(openssl rand -hex 32)"

# Dosya izinlerini kısıtla
chmod 600 ~/.openclaw/openclaw.json
chmod 700 ~/.openclaw/workspace
```

8.2 Güvenlik Denetimi
```
# Güvenlik denetimi yap
openclaw security audit --deep
```

8.3 Telegram Güvenliği
```
# Sadece belirli kullanıcıların yazmasına izin ver
openclaw config set channels.telegram.allowFrom '[123456789]'  # Telegram user ID'niz
```

---

# 9. Günlük Kullanım Komutları
Gateway Yönetimi

```
# Gateway'i başlat (screen içinde)
screen -S openclaw
openclaw gateway --port 18789
# CTRL+A, D ile ayrıl

# Gateway durumunu kontrol et
openclaw status

# Gateway'i durdur (screen'e bağlan, CTRL+C)
screen -r openclaw
```

Channel Yönetimi
```
# Channel durumunu kontrol et
openclaw channels status --probe

# Pairing isteklerini listele
openclaw pairing list telegram

# Pairing onayla
openclaw pairing approve telegram KOD
```

Test Mesajı Gönderme
```
# Telegram'da test
openclaw message send --target TELEGRAM_USER_ID --message "test"
```

Log İzleme
```
# Logları canlı izle
openclaw logs --follow
```

Yedekleme
```
# Konfigürasyonu yedekle
cp -r ~/.openclaw ~/openclaw-yedek-$(date +%Y%m%d)
```


Kurulum Tamamlandı
Şimdi

✅ OpenClaw AI Gateway çalışıyor

✅ Telegram botunuz aktif

✅ ChatGPT hesabınızla ücretsiz kullanım

✅ Güvenlik önlemleri alındı

Botunuza Telegram'dan mesaj gönderip kişiliğini ayarlayabilirsiniz.

