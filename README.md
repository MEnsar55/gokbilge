# Merhabalar. Proje görsellerinde TUA Hackhathon 2026 yerine 2025 yazmışız yanlışlıkla. Onun da düzeltildiği kodu yeni dosya olarak paylaşıyorum. PROJENİN SON HALİ BUDUR VE LÜTFEN BUNU DEĞERLENDİRİN :) SON_DOSYA_BU_GOKBİLGE.zip


# 🛰️ GÖKBILGE — Uzay Çöpü Çarpışma Tahmin Sistemi

> **TUA Hackathon 2026 · Yörünge Temizliği Kategorisi**

Gerçek ABD Uzay Kuvvetleri yörünge verisiyle beslenen, SGP4 propagasyonu + fizik-temelli öznitelik mühendisliği + kalibre edilmiş ML modeli içeren uçtan uca uzay çöpü çarpışma tahmin sistemi. Türk uydularına özel 5.000 km güvenlik tamponu uygular, sonuçları interaktif Streamlit dashboard'da sunar.

---

## 🚀 Hızlı Başlangıç (Jüriler İçin)

### Windows — Tek Tıkla

```bash
baslat.bat
```

### Manuel Kurulum

```bash
# 1. Bağımlılıkları yükle
pip install -r requirements.txt

# 2. Geçmiş gerçek verilerle API gerektirmeden çalıştır (Önerilen)
python main.py --skip-fetch

# → Tarayıcınızda http://localhost:8501 adresinde dashboard açılır
```

### Tüm Çalıştırma Modları

```bash
python main.py --skip-fetch      # Geçmiş veriyle çalış (Jüriler için önerilen mod)
python main.py                   # Tam pipeline (Space-Track API gerekli)
python main.py --demo            # Sentetik veri ile deneme yap
python main.py --dashboard-only  # Sadece dashboard'u başlat
python main.py --no-dashboard    # Dashboard olmadan pipeline çalıştır
```

---

## 📐 Mimari

```
Space-Track.org API ──→ Veri Çekme ──→ Temizleme ──→ SGP4 Propagasyon
                         1.000 enkaz     TLE doğrulama    TLE → ECI
                         500 uydu        Kepler irtifa     (x,y,z,vx,vy,vz)
                         9 Türk uydusu   hesabı
                                            │
                    ┌───────────────────────┘
                    ▼
         Öznitelik Mühendisliği ──→ ML Eğitimi ──→ Risk Tahmini
         cKDTree O(N log N)         RF + XGBoost    HIGH / MEDIUM / LOW
         11 fizik öznitelik         Kalibrasyon      Kalibre edilmiş P(collision)
         50.000 çift analiz         (Platt/Isotonic)
                                            │
                    ┌───────────────────────┘
                    ▼
            Streamlit Dashboard (5 Sekme)
            3D Globe │ Risk Paneli │ Analitik │ Yörünge Animasyonu │ Kessler Sim.
```

---

## 📂 Proje Yapısı

```
space-debris-tracker/
├── main.py                             # Ana pipeline orkestratörü
├── config.py                           # Tüm sabitler, API URL, ML parametreleri
├── requirements.txt                    # Python bağımlılıkları
├── baslat.bat                          # Windows tek tıkla kurulum
├── baslat.sh                           # Linux/macOS tek tıkla kurulum
├── .env.example                        # Ortam değişkenleri şablonu
│
├── pipeline/
│   ├── fetch_data.py                   # Space-Track REST API istemcisi
│   ├── clean_data.py                   # TLE veri temizleme ve doğrulama
│   ├── transform_data.py              # SGP4 TLE → ECI vektör dönüşümü
│   └── generate_demo_data.py          # API gereksiz sentetik veri üretici
│
├── models/
│   ├── feature_engineering.py         # cKDTree uzamsal tarama + 11 öznitelik
│   ├── train_model.py                 # RF + XGBoost eğitim + kalibrasyon
│   └── predict.py                     # Batch çarpışma riski tahmini
│
├── dashboard/
│   ├── app.py                         # Streamlit 5-sekmeli dashboard
│   ├── styles/custom.css              # Dashboard tema
│   └── components/
│       ├── globe_3d.py                # 3D Scatter3D yörünge haritası
│       ├── risk_panel.py              # Çarpışma risk tablosu
│       ├── stats_cards.py             # İstatistik kartları
│       ├── timeline_chart.py          # Analitik grafikler
│       ├── orbit_animation.py         # 24h Keplerian yörünge animasyonu
│       ├── kessler_sim.py             # Kessler kaskad simülasyonu
│       ├── conjunction_timer.py       # TCA geri sayım sayacı
│       └── pdf_report.py             # PDF rapor üretici
│
├── presentation/
│   ├── images/                        # Sunum görselleri
│   └── index.html                     # HTML sunum
│
└── data/                              # Pipeline çıktıları (gitignore)
    ├── raw/                           # Ham API verileri
    ├── processed/                     # İşlenmiş CSV dosyaları
    └── models/                        # Eğitilmiş modeller + raporlar
```

---

## 🔬 Teknik Detaylar

### 6 Adımlı Pipeline

| Adım | Modül | Açıklama |
|------|-------|----------|
| 1. Veri Toplama | `fetch_data.py` | Space-Track.org API'den 1.500 nesne (enkaz + uydu + 9 Türk uydusu) |
| 2. Temizleme | `clean_data.py` | TLE doğrulama, Kepler irtifa hesabı, Türk uydusu bayrağı |
| 3. Propagasyon | `transform_data.py` | SGP4 ile TLE → ECI konum/hız vektörleri |
| 4. Öznitelik | `feature_engineering.py` | cKDTree O(N log N) tarama + 11 fizik öznitelik, 50.000 çift |
| 5. ML Eğitimi | `train_model.py` | Random Forest + XGBoost + olasılık kalibrasyonu |
| 6. Tahmin | `predict.py` | Kalibre edilmiş çarpışma olasılığı → HIGH/MEDIUM/LOW |

### 11 Fizik-Temelli Öznitelik

| # | Öznitelik | Fiziksel Anlam |
|---|-----------|---------------|
| 1 | `distance_km` | Öklid mesafesi (Akella & Alfriend 2000) |
| 2 | `relative_speed_kms` | Göreli hız büyüklüğü |
| 3 | `altitude_diff_km` | İrtifa farkı |
| 4 | `avg_altitude_km` | Ortalama irtifa |
| 5 | `closing_speed_kms` | Kapanma hızı (negatif = yaklaşıyor) |
| 6 | `time_to_closest_approach_s` | TCA tahmini (Foster & Estes 1992) |
| 7 | `combined_cross_section` | Çarpışma kesit alanı (Chan 2008) |
| 8 | `orbit_regime_match` | Aynı yörünge rejiminde mi? |
| 9 | `inclination_diff` | Eğim farkı |
| 10 | `eccentricity_sum` | Eksantriklik toplamı |
| 11 | `involves_turkish_satellite` | Türk uydusu bayrağı |

### İzlenen Türk Uyduları

| Uydu | NORAD ID | Yörünge | İrtifa | Eğim |
|------|----------|---------|--------|------|
| TÜRKSAT-3A | 33056 | GEO | ~35.786 km | ~0.05° |
| TÜRKSAT-4A | 39522 | GEO | ~35.786 km | ~0.04° |
| TÜRKSAT-4B | 40984 | GEO | ~35.786 km | ~0.03° |
| TÜRKSAT-5A | 47306 | GEO (31°E) | ~35.786 km | ~0.02° |
| TÜRKSAT-5B | 50212 | GEO (42°E) | ~35.786 km | ~0.02° |
| GÖKTÜRK-1 | 41875 | SSO | ~695 km | 98.1° |
| GÖKTÜRK-2 | 39030 | SSO | ~684 km | 98.1° |
| RASAT | 37791 | SSO | ~685 km | 98.1° |
| İMECE | 56178 | SSO | ~680 km | 97.9° |

---

## 🖥️ Dashboard Sekmeleri

| Sekme | İçerik |
|-------|--------|
| 🌍 **3D Yörünge Haritası** | Plotly 3D globe ile tüm nesneler, risk renklendirmesi |
| ⚠️ **Risk & Conjunction** | En tehlikeli çiftler, Türk uydusu özel paneli, TCA geri sayım |
| 📈 **Analitik** | Yörünge rejimi dağılımı, ML öznitelik önemi, irtifa-hız scatter |
| 🎬 **Yörünge Animasyonu** | 24 saatlik Keplerian + J2 yörünge hareketi, pürüzsüz akış modu |
| 💥 **Kessler Simülasyonu** | NASA SBM fragment fiziği ile kaskad animasyonu |

---

## 🧪 Teknolojiler

| Bileşen | Teknoloji | Referans |
|---------|-----------|----------|
| Yörünge propagasyonu | `sgp4` (SGP4/SDP4) | Vallado et al. (2006) |
| Uzamsal tarama | `scipy.spatial.cKDTree` | Hoots et al. (2004) |
| ML: Random Forest | `scikit-learn` (200 ağaç) | Breiman (2001) |
| ML: XGBoost | `xgboost` (200 ağaç) | Chen & Guestrin (2016) |
| Olasılık kalibrasyonu | `CalibratedClassifierCV` | Niculescu-Mizil & Caruana (2005) |
| Dashboard | `Streamlit` | — |
| 3D Görselleştirme | `Plotly` (Scatter3D, Surface) | — |
| Veri işleme | `pandas`, `NumPy` | — |

---

## 📚 Bilimsel Referanslar

1. **Akella & Alfriend (2000)** — *"Probability of Collision Between Space Objects"*, JGCD 23(5)
2. **Alfano (2005)** — *"Satellite Conjunction Monte Carlo Analysis"*, JSR 42(1)
3. **Chan (2008)** — *Spacecraft Collision Probability*, AIAA Press
4. **Foster & Estes (1992)** — *"Parametric Analysis of Orbital Debris Collision Probability"*, NASA/JSC-25898
5. **Hoots et al. (2004)** — Üç-filtre conjunction tarama algoritması, AAS/AIAA
6. **Johnson et al. (2001)** — *"NASA's New Breakup Model of EVOLVE 4.0"*, ASR 28(9)
7. **Kessler & Cour-Palais (1978)** — *"Collision Frequency of Artificial Satellites"*, JGR 83(A6)
8. **Niculescu-Mizil & Caruana (2005)** — *"Predicting Good Probabilities"*, ICML 2005
9. **Vallado (2013)** — *Fundamentals of Astrodynamics*, 4th ed.
10. **ESA Space Environment Report (2025)** — 40.000+ izlenen nesne

---

## 📄 Lisans

MIT License — Eğitim ve araştırma amaçlı serbestçe kullanılabilir.

---

<p align="center">
  <b>GÖKBILGE</b> · TUA Hackathon 2026 · Yörünge Temizliği Kategorisi
</p>
