# Kullanım Kılavuzu

Bu depo, GitHub profilinde (`github.com/hyqanxd`) görünen temanın kaynak dosyalarıdır.
Profil sayfasında görünmesi için **repo adı kullanıcı adınla aynı olmalıdır** (`hyqanxd`) ve
`README.md` dosyası kök dizinde bulunmalıdır — bu koşullar sağlandığı için profilinde otomatik görünür.

---

## 1. Dosya haritası

| Dosya | Ne işe yarar |
|---|---|
| `README.md` | Profil sayfasında görünen içerik |
| `assets/header.svg` | Üst banner (kimlik + site/topluluk kartları) |
| `assets/architecture.svg` | "Veri nereden geliyor?" akış diyagramı |
| `assets/repo-frontend.svg` | Frontend proje kartı |
| `assets/repo-backend.svg` | Backend proje kartı |
| `assets/divider.svg` | Bölüm ayırıcı çizgi |
| `assets/metrics.svg` | GitHub istatistik kartı — **workflow üretir**, elle düzenleme |
| `dist/output/github-contribution-snake*.svg` | Katkı grafiği — **workflow üretir**, elle düzenleme |
| `.github/workflows/metrics.yml` | `lowlighter/metrics` iş akışı (günlük) |
| `.github/workflows/snake.yml` | `Platane/snk` iş akışı (günlük) |

> `assets/metrics.svg` ve `dist/output/*.svg` dosyaları ilk çalıştırmaya kadar "henüz üretilmedi"
> yazan yer tutucudur. Workflow çalıştıktan sonra gerçek veriyle değişir.

---

## 2. İlk kurulum (tek seferlik)

### 2.1 Profil README'sini aktif et

Repo adın zaten `hyqanxd` olduğu için otomatik olarak profilde görünür.
Görünmüyorsa: repo sayfasında **Share to profile** butonuna bas.

### 2.2 `METRICS_TOKEN` secret'ını oluştur

> ⚠️ **Fine-grained token ÇALIŞMAZ.** GitHub fine-grained PAT'leri GraphQL kimlik
> doğrulamasında desteklemiyor; `lowlighter/metrics` bu durumda
> *"It seems you're trying to use a fine-grained personal access token"* diyerek
> **exit code 1** ile düşüyor. **Classic** token kullan.

1. GitHub → **Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)**
2. **Generate new token (classic)**
3. **Note:** istediğin bir ad yaz
4. **Expiration:** 90 gün (süre bitince kart boşalır, yenile)
5. **Scopes:** **hiçbir scope seçme** — metrics dokümanına göre scope gerekmiyor
   (gerekirse ilk çalıştırma hatasında hangi scope'un istendiği yazar, o zaman sadece onu ekle)
6. **Generate token** → kopyala

Sonra:

1. Bu repo → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**
2. Name: `METRICS_TOKEN`
3. Secret: az önce kopyaladığın classic token
4. **Add secret**

> Secret yoksa workflow çalışır ama istatistik kartı boş/eksik gelir.
1. Bu repo → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**
2. Name: `METRICS_TOKEN`
3. Secret: az önce kopyaladığın token
4. **Add secret**

> Secret yoksa workflow çalışır ama istatistik kartı boş/eksik gelir.

### 2.3 İlk veriyi üret

Repo → **Actions** sekmesi:

- **Metrics** → **Run workflow** → yeşil buton
- **Generate snake** → **Run workflow** → yeşil buton

İlk çalıştırmalar 2–5 dakika sürebilir (Docker imajı çekiliyor).
Bittiğinde `assets/metrics.svg` ve `dist/output/*.svg` dosyaları otomatik commit edilir.

---

## 3. Günlük otomatik güncelleme

Her iki workflow da `schedule` ile kendiliğinden çalışır:

| Workflow | Saat (UTC) | Ne yapar |
|---|---|---|
| `metrics.yml` | 03:17 | İstatistikleri yeniler ve commit eder |
| `snake.yml` | 03:23 | Katkı grafiğini yeniler ve commit eder |

Ek olarak, `main` dalına push yapıldığında da çalışırlar.

---

## 4. Temayı değiştirme

### 4.1 Renkler

Tüm görseller **elle yazılmış, harici bağımlılığı olmayan** SVG dosyalarıdır.
GitHub bu SVG'leri `<img>` olarak servis ettiği için içlerinde `@import` (Google Fonts)
veya `<script>` **çalışmaz**; bu yüzden sistem font yığını kullanılır.

Marka renkleri uygulamanın kendi renklerinden gelir:

| Değer | Nerede | Kaynak |
|---|---|---|
| `#FF0000` | ana vurgu, buton, aktif link | `src/App.jsx` → MUI `primary.main` |
| `#FF4444` | degrade ucu | `linear-gradient(135deg,#FF0000,#ff4444)` |
| `#000000` / `#101010` | zemin / kart yüzeyi | `background.default`, `paper` |
| `#E5E5E5` | ikincil metin | gövde metin rengi |
| `#9a9a9a` | kart etiketleri | bu temada eklenen kurumsal gri |

Değiştirmek için ilgili SVG'yi bir metin editöründe aç ve `fill="#..."` / `stroke="#..."`
değerlerini değiştir. Kart sistemi her dosyada aynıdır:

```
kart zemin     : fill #101010, stroke #FFFFFF @ %10
üst vurgu şerit : 3px, degrade #FF4444 → #FF0000
etiket         : 10.5px, letter-spacing 2.6, #9a9a9a, uppercase
başlık         : #FFFFFF, ağırlık 700–800
gövde          : 11.5px, #E5E5E5 @ %66
```

### 4.2 İstatistik kartının rengi

Kart, `lowlighter/metrics` tarafından üretilir; renkler workflow içindeki
`extras_css` (başlık/ikon renkleri) ve `extras_js` (takvim ısı skalası) ile ayarlanır.

```yaml
extras_css: |
  h1, h2, h3 { color: #FF0000 !important; }
  .field    { color: #E5E5E5; }
  .field svg{ fill: #FF0000; }

extras_js: |
  for (const [from, to] of [
    ["#ebedf0", "#1a1a1a"],   # boş hücre
    ["#9be9a8", "#4a0000"],
    ["#40c463", "#a30000"],
    ["#30a14e", "#d40000"],
    ["#216e39", "#ff0000"],   # en yoğun hücre
  ])
    document.querySelectorAll(`svg g [fill="${from}"]`).forEach(n => n.setAttribute("fill", to))
```

> Not: metrics kendi CSS sınıfları sürümler arasında değişebilir. `extras_css` çalışmazsa
> en olası sebep budur; `.field` / `h1,h2,h3` yerine güncel sınıfları kullan.

### 4.3 Hangi eklenti kartları görünsün

`.github/workflows/metrics.yml` içinde her satır bir ayar:

| Ayar | Etki |
|---|---|
| `plugin_calendar: yes` | Yıllık katkı ızgarası |
| `plugin_isocalendar: yes` | Uzun (1 yıllık) izgara — `duration: half-year` yaparsan kısalır |
| `plugin_languages_limit: 8` | En çok kullanılan dil sayısı |
| `plugin_activity_limit: 20` | Son aktivite satırı |
| `plugin_repositories_pinned: yes` | Sabitlenmiş repo kartları |
| `plugin_sponsors: yes` | Sponsor bölümü (sponsor yoksa boş görünmez) |
| `plugin_habits_days: 30` | Alışkanlık istatistiği penceresi |

Kapatmak için `yes` → `no`.

---

## 5. İçerik düzenleme

### Metinler
`README.md` normal Markdown + HTML. Tablo içinde iki sütun kullanılıyor; görseller
`./assets/...` yoluyla çağrılıyor (Profil reposu olduğu için göreli yol şart).

### Yeni proje kartı eklemek

1. `assets/repo-yeni.svg` olarak aynı kart şablonuyla bir SVG yaz
   (`assets/repo-backend.svg` dosyasını kopyala, yazıları değiştir).
2. `README.md` içindeki "Projelerim" tablosuna bir `<td>` ekle.

### Site / Discord bağlantıları
Şu an: `anitilky.com` ve `discord.gg/ncp623cWwr`.
Değiştirmek için: `README.md` (badge + link) ve `assets/header.svg` (sağ kartlar).

---

## 6. Lokal önizleme

README'deki görselleri tarayıcıda denetlemek için:

```bash
cd hyqanxd
python -m http.server 8765
# tarayıcıda: http://127.0.0.1:8765/assets/header.svg
```

Tam sayfa düzenini görmek için `README.md` dosyasını VS Code'da "Open Preview" ile aç
veya GitHub'ı bekle — profil zaten GitHub'da render edilir.

---

## 7. Sık karşılaşılan sorunlar

| Belirti | Sebep | Çözüm |
|---|---|---|
| Görsel kırık / "henüz üretilmedi" | Workflow henüz çalışmadı | Actions → Metrics / Generate snake → Run workflow |
| Metrics: *"you're trying to use a fine-grained personal access token"* | Fine-grained PAT, GraphQL'de desteklenmiyor | **Classic** token üret (bkz. 2.2) ve secret'ı güncelle |
| Metrics: *"required token is missing"* | `METRICS_TOKEN` secret'ı yok | Settings → Secrets → Actions → `METRICS_TOKEN` ekle |
| Snake: *"Unexpected input(s) 'github_token'"* / *"Build dir does not exist"* | Eski `crazy-max/ghaction-github-pages` adımı | Bu adım kaldırıldı; README görseli depodaki dosyadan okuyor, gh-pages'a yayın gerekmiyor |
| Snake: *"git-status failed: not a git repository"* | `Platane/snk` checkout yapmıyor, commit adımı boş dizinde çalışıyor | `actions/checkout@v5` adımı eklendi (snk'dan **önce**) |
| Node.js 20 deprecated uyarısı | Kullanılan action Node 20 hedefliyor | `git-auto-commit-action` kaldırıldı, commit ham `git` ile yapılıyor; checkout `@v5` (Node 24). Bu uyarı artık çıkmamalı |
| Workflow hata: `403` / `rate limit` | Token kapsamı yetersiz | Hata çıktısında istenen scope'u **classic** token'a ekle |
| Kartlar kırmızı görünmüyor | `extras_css` uygulanmamış | metrics sürümü değişmiş olabilir; `extras_css` içindeki seçicileri güncelle |
| Yıllık ızgara çok uzun | — | `plugin_isocalendar_duration: half-year` yap |
| Profilde görünmüyor | Repo adı kullanıcı adıyla eşleşmiyor veya README boş | Repo adı `hyqanxd` olmalı; README dolu olmalı |
| Değişiklikler geç görünüyor | GitHub CDN önbelleği | `.../hyqanxd/blob/main/README.md?raw=1` ile zorla yenile |
| README'de `</td>` gibi kod bloğu çıkıyor | Tablo içinde boş satır bırakılmış | HTML bloğu boş satırda biter; `<table>` içinde **hiç boş satır olmasın**, içerikler saf HTML olsun |

---

## 8. Yayınlama (komutlar)

```bash
cd hyqanxd
git add -A
git commit -m "Profil temasini guncelle"
git push origin main
```

Push'tan sonra iki workflow otomatik tetiklenir.