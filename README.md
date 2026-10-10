# Kalp Hastalığı Tahmini

### Makine Öğrenmesi ve Bayesian Ağları Tabanlı Analiz

**2025-2026 Veri Madenciliği** dersi kapsamında geliştirilen bu ekip projesi, klinik özelliklerden yararlanarak `HeartDisease` hedef değişkenini tahmin etmeyi amaçlar. Çalışmada veri ön işleme, **Random Forest**, **Gradient Boosting** ve **Tree Augmented Naive Bayes (TAN)** yöntemleri uygulanmış; sınıflandırma sonuçları ve değişkenler arasındaki olasılıksal ilişkiler incelenmiştir.

## Proje ekibi

- **Muzaffer Türkoğlu**
- **Serhat Özdemir**
- **Hüseyin Konak**
- **Tolga Karateke**

**Ders:** Veri Madenciliği

**Dersi veren:** Dr. Öğr. Üyesi Ümit Demirbaga

Ekip ve ders bilgileri, projenin “Kalp Hastalığı Tahmini için Makine Öğrenmesi ve Bayesian Ağları Tabanlı Analiz” başlıklı raporundan alınmıştır.

## Projede neler yapılıyor?

1. Ham veri incelenerek eksik ve aykırı değerler işlenir.
2. Kategorik değişkenler sayısallaştırılır, sürekli değişkenler ölçeklendirilir ve yaş kategorisi oluşturulur.
3. Random Forest ve Gradient Boosting modelleri eğitilir; beş katlı çapraz doğrulama, test ve holdout değerlendirmeleri yapılır.
4. TAN modeliyle değişkenler arasındaki bağlantılar ve koşullu olasılık tabloları öğrenilir.
5. Karışıklık matrisleri, özellik önemleri, sınıf bazlı metrikler ve Bayesian ağ yapısı görselleştirilir.
6. Gradient Boosting modeli ve holdout tahminleri dosyaya kaydedilir.

## Veri seti

| Dosya | Satır | Sütun | İçerik |
| --- | ---: | ---: | --- |
| `heart.csv` | 918 | 12 | 11 girdi özelliği ve hedef değişken |
| `heart_onislenmis.csv` | 890 | 13 | Temizlenmiş veriler ve eklenen `Age_Category` |
| `tahmin_sonuclari.csv` | 89 | 14 | Gradient Boosting holdout girdileri, gerçek sınıflar ve tahminler |

**Hedef değişken:** `HeartDisease` (`0`: kalp hastalığı yok, `1`: kalp hastalığı var).

| Özellik | Açıklama |
| --- | --- |
| `Age` | Yaş |
| `Sex` | Cinsiyet |
| `ChestPainType` | Göğüs ağrısı tipi |
| `RestingBP` | Dinlenme kan basıncı |
| `Cholesterol` | Kolesterol |
| `FastingBS` | Açlık kan şekeri göstergesi |
| `RestingECG` | Dinlenme elektrokardiyogramı sonucu |
| `MaxHR` | Ulaşılan maksimum kalp atış hızı |
| `ExerciseAngina` | Egzersizle ilişkili anjina |
| `Oldpeak` | ST depresyonu değeri |
| `ST_Slope` | Egzersiz ST segmentinin eğimi |
| `Age_Category` | Ön işleme sırasında türetilen yaş grubu |

### Ön işleme adımları

- `Cholesterol` sütunundaki **172 sıfır değer** eksik kabul edilerek sütun ortalamasıyla doldurulur.
- `Age`, `RestingBP`, `Cholesterol`, `MaxHR` ve `Oldpeak` üzerinde mutlak Z-skoru 3'ü aşan **28 satır** çıkarılır.
- `Sex`, `ChestPainType`, `RestingECG`, `ExerciseAngina` ve `ST_Slope`, `LabelEncoder` ile dönüştürülür.
- Beş sürekli değişken `MinMaxScaler` ile 0-1 aralığına ölçeklendirilir.
- Orijinal yaş değerlerinden `pd.cut` ile `(0, 40]`, `(40, 60]`, `(60, 100]` aralıkları oluşturulur ve sırasıyla 0, 1, 2 olarak kodlanır.

## Modeller ve kayıtlı sonuçlar

Aşağıdaki değerler **teslim edilen Jupyter defterinin kayıtlı çıktılarından ve proje raporundan** aktarılmıştır. Yeni bir deney çalıştırmasının sonuçları değildir. Gradient Boosting değerleri, özgün çıktıdaki yuvarlama hassasiyetiyle verilmiştir.

| Model | 5 katlı çapraz doğrulama | Test doğruluğu | Holdout doğruluğu |
| --- | ---: | ---: | ---: |
| Random Forest | %86,56 | %83,85 | %94,38 |
| Gradient Boosting | ≈ %87 | ≈ %84 | ≈ %91 |
| TAN Bayesian ağı | Uygulanmadı | %84,27 | Ayrılmadı |

**Random Forest ve Gradient Boosting:** Önce verinin %10'u holdout olarak ayrılır. Kalan %90'ın %80'i eğitim, %20'si test için kullanılır. Sonuçta **640 eğitim, 161 test ve 89 holdout örneği** elde edilir. Veri bölmelerinde `random_state=42` kullanılır.

- Random Forest: `n_estimators=100`, `random_state=42`.
- Gradient Boosting: `n_estimators=100`, `learning_rate=0.1`, `max_depth=3`, `random_state=42`.

**TAN Bayesian ağı:** 890 örnek %80 eğitim / %20 test olarak bölünür (**712 / 178 örnek**). Sürekli değişkenler beş kategoriye ayrılır. `pgmpy.TreeSearch` ile TAN yapısı, `MaximumLikelihoodEstimator` ile koşullu olasılık tabloları öğrenilir. Kayıtlı ağda 23 bağlantı bulunur.

| TAN hedef değişken metriği | Değer |
| --- | ---: |
| Accuracy | 0,8427 |
| Ağırlıklı F1 | 0,8423 |
| Ağırlıklı Precision | 0,8428 |
| Ağırlıklı Recall | 0,8427 |

TAN bölümünde ayrıca her sütun sırayla gizlenerek diğer sütunlardan tahmin edilir. Bu ayrı analizde tüm değişkenler için ortalama doğruluk **0,6422**'dir; bu değer kalp hastalığı sınıflandırma doğruluğu değildir.

## Dosya yapısı

```text
heart-disease-prediction/
├── README.md
├── requirements.txt
├── .gitignore
├── kalp_hastaligi_tahmini.ipynb  # Ön işleme, modelleme ve görselleştirme
├── heart.csv                    # Ham veri
├── heart_onislenmis.csv          # Ön işlenmiş veri
├── kalp_modeli_gb.pkl            # Kaydedilmiş Gradient Boosting modeli
├── tahmin_sonuclari.csv          # Holdout tahminleri
└── HeartDisease_TAN_Network.pdf  # TAN ağ görselleştirmesi
```

Defterin arşivdeki adı `Untitled.ipynb` idi; içerik değiştirilmeden daha açıklayıcı bir ad verilmiştir.

## Kurulum ve çalıştırma

Projeyi indirin veya klonlayın ve proje klasöründe bir terminal açın. Orijinal defterin ortam bilgisi **Python 3.13.5** olarak kayıtlıdır. Özgün paket sürümleri arşivde bulunmadığından `requirements.txt` doğrudan kullanılan bağımlılıkları listeler; birebir ortam kilidi değildir.

```bash
python -m venv .venv
```

Sanal ortamı etkinleştirin:

```powershell
# Windows PowerShell
.\.venv\Scripts\Activate.ps1
```

```bash
# macOS / Linux
source .venv/bin/activate
```

Bağımlılıkları kurup defteri açın:

```bash
python -m pip install -r requirements.txt
python -m jupyterlab kalp_hastaligi_tahmini.ipynb
```

Jupyter'da kurduğunuz sanal ortamın Python çekirdeğini seçin ve hücreleri **baştan sona sırayla** çalıştırın. Defter, CSV dosyalarını bulunduğu klasörden göreli yollarla okur. Baştan çalıştırma `heart_onislenmis.csv`, `kalp_modeli_gb.pkl`, `tahmin_sonuclari.csv` ve `HeartDisease_TAN_Network.pdf` dosyalarını yeniden oluşturur.

## Yöntemle ilgili notlar

Bu depo, teslim edilen ders projesini ve kayıtlı deney sonuçlarını belgelemektedir. Sonuçları yorumlarken aşağıdaki uygulama ayrıntıları dikkate alınmalıdır:

- Ön işleme, eğitim/test ayrımından önce tüm veri üzerinde yapılmıştır. İleri çalışmalarda dönüşümlerin yalnızca eğitim verisine uyarlanması, veri sızıntısını önlemek için gereklidir.
- TAN bölümünde eğitim ve test verisine `pd.cut(..., bins=5)` ayrı ayrı uygulanır. Yeniden geliştirilecek bir sürümde eğitimden öğrenilen ortak bin sınırları kullanılmalıdır.
- TAN ve diğer iki model aynı test kümesini kullanmaz; tablodan doğrudan bir model üstünlüğü sonucu çıkarılmamalıdır.
- Kayıtlı TAN log-likelihood değeri `-inf`'dir. Olasılık tablolarındaki sıfır olasılıklar ve yumuşatma yöntemleri ayrıca incelenebilir.
- Kaydedilen `.pkl` dosyası yalnızca Gradient Boosting sınıflandırıcısını içerir; kodlayıcılar ve ölçekleyici bu dosyada saklanmaz. Ham veriyi doğrudan bu modele vermek yerine eğitimdeki aynı dönüşüm ve sütun düzeni uygulanmalıdır.

## Kaynak ve kullanım

Bu çalışma bir **akademik veri madenciliği projesidir**; klinik kullanım için doğrulanmış bir tanı sistemi değildir.

Proje açıklaması ve ekip bilgileri, ekip tarafından hazırlanan proje raporuna; uygulama ayrıntıları ve sonuçlar ise teslim edilen defter ve veri dosyalarına dayanır. Arşivde `heart.csv` için özgün veri kaynağı bağlantısı veya lisans bilgisi bulunmadığından bu bilgiler varsayılmamıştır. Depoya ayrıca bir açık kaynak lisansı eklenmemiştir.
