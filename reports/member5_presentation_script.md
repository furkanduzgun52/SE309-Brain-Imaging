# Sunum Videosu Konuşma Metni — Üye 5 (Fatih Osman Darılmaz)

**Süre Hedefi:** ~2 dakika *(5 kişilik grup toplam ~10 dakika; her üye ~2 dakika)*  
**Gösterilecek Görseller:** `images/roc_threshold_cv.png` ve `images/test_confusion_matrix.png`

---

## İngilizce Konuşma Metni (Videoda Okumak / Anlatmak İçin)

> **Slide 1 — Model Comparison (0:00 – 0:40)**  
> "Hi, I'm Fatih Osman Darılmaz, and I led Part 5: choosing a model and decision threshold for memory-clinic dementia screening. Using the exact same leakage-free preparation as Part 1—excluding `CDR` and `Group`, imputing missing `SES` and `MMSE` with training medians inside pipelines, and keeping subjects strictly separated—we compared four classifiers against the ZeroR baseline using 10-fold subject-wise stratified cross-validation (`StratifiedGroupKFold`) on the 112 training subjects. ZeroR achieved 54.5% accuracy with zero sensitivity. Among the real models, standardized Logistic Regression performed best across Accuracy (0.828), Sensitivity (0.756), F1 (0.800), and ROC AUC (0.908), outperforming Naive Bayes (0.858), Random Forest (0.852), and a depth-3 Decision Tree (0.766)."
>
> **Slide 2 — ROC Curve & Screening Threshold (0:40 – 1:15)**  
> "Looking at the training CV ROC curve on the left, Logistic Regression at the default 0.50 threshold still missed 31 of 127 demented training visits. In memory-clinic screening, missing a demented patient is much more serious than a false alarm that triggers a follow-up cognitive exam. Treating a false negative as three times as costly as a false positive gives a threshold of `1 / (1 + 3) = 0.25`. Choosing `t = 0.25` strictly on training data raised CV sensitivity from 75.6% to 91.3% while keeping specificity at 65.8%."
>
> **Slide 3 — Prevalence Effect & Final Test Evaluation (1:15 – 2:00)**  
> "Next, we computed precision per 1,000 screened older adults at two prevalences. At OASIS-2's 44.8% prevalence, precision is 68.4%. However, at a realistic 10% general older population prevalence, precision drops to 22.9%—meaning about 77% of positive flags would be false alarms. This shows the model can only be a first-stage screening prompt before specialist assessment, never a standalone diagnosis. Finally, evaluating our locked Logistic Regression model and `0.25` threshold once on the 38 unseen test subjects gave an AUC of 0.873 and 90.0% sensitivity—catching 36 of 40 demented visits and missing only 4, compared to 11 missed at the default 0.50 threshold. Thank you."

---

## Türkçe Kısa Özet (Ne Anlattığını Akılda Tutmak İçin)
1. **CV Karşılaştırması:** 112 eğitim hastasında 10-fold subject-wise CV ile 4 modeli ZeroR'a karşı kıyasladık; **Logistic Regression** en iyi model oldu (`AUC 0.908`, `Acc 0.828`, `F1 0.800`).
2. **Threshold Seçimi (`t = 0.25`):** Sadece eğitim CV'de seçtik; 3:1 FN:FP maliyet oranıyla (`1/(1+3)=0.25`) duyarlılık `%75.6 → %91.3` oldu (Spec `%65.8`).
3. **Prevalans Etkisi:** Aynı model OASIS-2'de (`%44.8`) `%68.4` precision verirken, `%10` toplum prevalansında `%22.9`'a düşüyor (~%77 yanlış alarm) → tek başına tanı aracı olamaz.
4. **Final Held-Out Test (38 yeni hasta, 94 ziyaret):** Tek seferlik testte `AUC 0.873`, `t=0.25` ile **%90.0 sensitivity (36/40 yakalandı, sadece 4 kaçtı)**, varsayılan `0.50`'de ise 11 hasta kaçıyordu.

---

## Hoca Sözlü Soru Sorarsa Hızlı Cevaplar (Q&A)
* **"Neden `CDR` ve `Group` sütunlarını çıkardınız?"** → `CDR` doğrudan hedef etiketi tanımlıyor (`CDR > 0`), `Group` ise hastanın gelecekteki ziyaretlerine bakılarak verilmiş retrospektif bir etiket (`Converted`); ikisi de veri sızıntısı (leakage) yapar.
* **"Neden normal `StratifiedKFold` değil `StratifiedGroupKFold` kullandınız?"** → OASIS-2'de aynı hastanın 2–5 ziyareti var. Normal CV'de aynı kişi hem eğitim hem doğrulama fold'una düşüyor ve Random Forest AUC'si yapay olarak `0.852`'den `0.940`'a (`+0.087`) şişiyor.
* **"Neden `t = 0.25` seçtiniz, `0.20` değil?"** → `0.25` hem 3:1 maliyet oranının tam teorik karşılığı (`1/(1+3)`) hem de eğitim CV'de `%90` üzeri duyarlılık (`%91.3`) sağlıyor; `0.20`'ye inince özgüllük `%65.8`'den `%57.2`'ye sert düşüp yanlış alarmları gereksiz artırıyor.
* **"Modelin duyarlılığı değişmediği halde `%10` prevalansta precision neden `%22.9`'a düştü?"** → Toplumun `%90`'ı sağlıklı olduğu için `900 × (1 − 0.658) ≈ 308` yanlış pozitif üretiliyor; bu sayı gerçek pozitiflerin (`~91`) üç katını geçtiği için `91 / (91 + 308) = %22.9` oluyor.
