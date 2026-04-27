# DATALEAGUE DATATHON

## Sosyal Medya Manipulasyonu ve Anomali Tespit Sistemi

Bu repo, DataLeague datathon gorevi icin hazirlanmistir.
Amac, global sosyal medya verisinde organik icerik ile manipule icerigi ayristiran
ve yeni gelen metinleri anlik degerlendiren bir sistem tasarlamaktir.

---

## 1) Gorev Ozeti

**Gorev Ismi:** Sosyal Medya Manipulasyonu ve Anomali Tespit Sistemi

Beklenen ana ciktilar:

1. **Guvenilirlik / Organiklik Skoru**
   - Her icerik veya `author_hash` icin 0-1 arasi skor.
2. **Manipulasyon Haritasi (Dashboard)**
   - Supheli kumelerin dil, platform ve tema dagilimi.
3. **Canli Tahmin Pipeline**
   - Sunum sirasinda verilen yeni metinler icin anlik
     "Organik / Manipulatif" karari ve karar gerekcesi.

---

## 2) Veri Seti Bilgisi

- **Dosya:** `datathonFINAL.parquet`
- **Format:** Apache Parquet
- **Satir:** ~5.000.000
- **Boyut:** ~1 GB

### Sutunlar

| Sutun | Aciklama |
|---|---|
| `original_text` | Gonderinin ham metni |
| `english_keywords` | Metinden cikarilan Ingilizce anahtar kelimeler |
| `sentiment` | Duygu skoru (`-1.0`, `0.0`, `1.0`) |
| `main_emotion` | Baskin duygu (`Joy`, `Anger`, vb.) |
| `primary_theme` | Gonderinin ana temasi |
| `language` | Dil kodu (`tr`, `en`, `es`, vb.) |
| `url` | Kaynak platform (`x.com`, `reddit.com`, vb.) |
| `author_hash` | Anonim kullanici kimligi |
| `date` | UTC tarih/saat |

---

## 3) Teknik Notlar (Kritik)

1. **Label yoktur**
   - Problem unsupervised anomali tespit problemidir.

2. **RAM yonetimi gerekir**
   - 5M satiri tek seferde yuklemek OOM riski olusturur.
   - Batch okuma, kolon secimi ve ornekleme kullanin.

3. **Gercek dunya verisi kirli olabilir**
   - Bos degerler, typo, dil karmasi, tekrarli icerikler beklenir.

---

## 4) Onerilen Pipeline

1. **Veri okuma**
   - Gerekli kolonlar ile parquet okuma.
2. **Temizleme**
   - Bos metin, duplicate, basic normalize.
3. **Ozellik cikarma**
   - Metin, davranissal (author), zamansal ozellikler.
4. **Skorlama**
   - Isolation Forest / LOF / clustering tabanli anomali skoru.
5. **Acilanabilir karar**
   - Karara etkili sinyalleri `reasons` olarak don.
6. **Dashboard**
   - Dil-platform-tema-zaman kesisimlerinde supheli yogunluk.

---

## 5) Repo Icerigi

- `main.ipynb`: Ana notebook (modelleme/tahmin)
- `eda.ipynb`: Kesifsel analiz
- `datathonFINAL.parquet`: Ham veri
- `datathon_simple_scored.parquet`: Skorlanmis cikti
- `tfidf_logreg_model.joblib`: Kayitli model
- `catboost_simple_model.cbm`: Kayitli model
- `simple_manipulation_summary.csv`: Ozet cikti
- `simple_feature_importance.csv`: Ozellik onemi
- `DataLig.pptx`: Sunum

---

## 6) Kurulum

### Gereksinimler

- Python 3.10+
- pip
- Jupyter Notebook veya JupyterLab

### Ornek Kurulum

```bash
python -m venv .venv
.venv\Scripts\activate
pip install --upgrade pip
pip install pandas pyarrow numpy scikit-learn catboost matplotlib seaborn plotly jupyter
```

---

## 7) Hizli Baslangic

1. `datathonFINAL.parquet` dosyasini proje kokune koy.
2. `main.ipynb` dosyasini ac.
3. Sirayla:
   - veri okuma
   - temizleme
   - ozellik cikarma
   - skor uretimi
   - raporlama adimlarini calistir.

### OOM Riski Icin Ornek

```python
import pandas as pd

cols = [
    "original_text", "english_keywords", "sentiment", "main_emotion",
    "primary_theme", "language", "url", "author_hash", "date"
]

df = pd.read_parquet("datathonFINAL.parquet", columns=cols)
df_sample = df.sample(n=500_000, random_state=42)
```

---

## 8) Inference Cikti Ornegi

```json
{
  "label": "manipulatif",
  "organicity_score": 0.18,
  "risk_score": 0.82,
  "reasons": [
    "Yuksek tekrarli keyword paterni",
    "Anormal author aktivite frekansi",
    "Tema-platform dagilimindan sapma"
  ]
}
```

---

## 9) Sunum Icin Kisa Tavsiye

1. "Label yok, unsupervised yaklasim kullandik" mesajini net ver.
2. Skorun nasil hesaplandigini sade bir diyagramla goster.
3. Dashboard uzerinden hangi dil/platform kombinasyonlarinin riskli oldugunu anlat.
4. Canli metin testinde sadece label degil, karar gerekcesini de goster.

---

## 10) Not

Model ciktilari karar destek amaclidir; nihai karar surecinde analist dogrulamasi gerekir.
