# 🧠 IBM HR Attrition — SQL Çalışma Kitabı

> **Dataset:** IBM tarafından oluşturulan kurgusal İK verisi · **1.470 çalışan · 35 kolon**  
> **Amaç:** Kolaydan zora SQL pratiği — her bölüm bir öncekinin üzerine inşa edilir.

---

## 📖 Sözlük — Sayısal Kodların Anlamları

| Kolon | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| `Education` | Lise Altı | Ön Lisans | Lisans | Yüksek Lisans | Doktora |
| `EnvironmentSatisfaction` | Düşük | Orta | Yüksek | Çok Yüksek | — |
| `JobInvolvement` | Düşük | Orta | Yüksek | Çok Yüksek | — |
| `JobSatisfaction` | Düşük | Orta | Yüksek | Çok Yüksek | — |
| `PerformanceRating` | Düşük | İyi | Mükemmel | Olağanüstü | — |
| `RelationshipSatisfaction` | Düşük | Orta | Yüksek | Çok Yüksek | — |
| `WorkLifeBalance` | Kötü | İyi | Daha İyi | En İyi | — |

---

## 🗄️ Bölüm 0 — Tablo Kurulumu

Tüm sorularda kullanacağın tek tablo: `hr_attrition`

```sql
CREATE TABLE hr_attrition (
    Age                      INT,
    Attrition                VARCHAR(3),        -- 'Yes' / 'No'
    BusinessTravel           VARCHAR(30),
    DailyRate                INT,
    Department               VARCHAR(50),
    DistanceFromHome         INT,
    Education                INT,
    EducationField           VARCHAR(50),
    EmployeeCount            INT,
    EmployeeNumber           INT PRIMARY KEY,
    EnvironmentSatisfaction  INT,
    Gender                   VARCHAR(10),
    HourlyRate               INT,
    JobInvolvement           INT,
    JobLevel                 INT,
    JobRole                  VARCHAR(50),
    JobSatisfaction          INT,
    MaritalStatus            VARCHAR(15),
    MonthlyIncome            INT,
    MonthlyRate              INT,
    NumCompaniesWorked       INT,
    Over18                   VARCHAR(1),
    OverTime                 VARCHAR(3),        -- 'Yes' / 'No'
    PercentSalaryHike        INT,
    PerformanceRating        INT,
    RelationshipSatisfaction INT,
    StandardHours            INT,
    StockOptionLevel         INT,
    TotalWorkingYears        INT,
    TrainingTimesLastYear    INT,
    WorkLifeBalance          INT,
    YearsAtCompany           INT,
    YearsInCurrentRole       INT,
    YearsSinceLastPromotion  INT,
    YearsWithCurrManager     INT
);
```

**CSV yükleme:**

```sql
-- PostgreSQL
COPY hr_attrition
FROM '/path/to/WA_Fn-UseC_-HR-Employee-Attrition.csv'
DELIMITER ',' CSV HEADER;

-- MySQL
LOAD DATA INFILE '/path/to/WA_Fn-UseC_-HR-Employee-Attrition.csv'
INTO TABLE hr_attrition
FIELDS TERMINATED BY ','
IGNORE 1 ROWS;
```

---

## 🟢 Bölüm 1 — Başlangıç Seviyesi

> **Konular:** `SELECT` · `WHERE` · `ORDER BY` · `LIMIT` · `CASE WHEN`

---

### S1 · Kaç çalışan var? Kaçı işten ayrılmış?

```sql
-- Toplam çalışan
SELECT COUNT(*) AS toplam_calisan
FROM hr_attrition;

-- İşten ayrılanlar
SELECT COUNT(*) AS ayrilan_calisan
FROM hr_attrition
WHERE Attrition = 'Yes';
```

> 💡 **Beklenen sonuç:** 1470 toplam · 237 ayrılan

---

### S2 · Sales departmanı çalışanlarını maaşa göre büyükten küçüğe sırala

```sql
SELECT
    EmployeeNumber,
    Age,
    JobRole,
    MonthlyIncome,
    BusinessTravel
FROM hr_attrition
WHERE Department = 'Sales'
ORDER BY MonthlyIncome DESC;
```

---

### S3 · Eğitim seviyelerini Türkçe etiketle, her seviyeden kaç çalışan var?

```sql
SELECT
    CASE Education
        WHEN 1 THEN 'Lise Altı'
        WHEN 2 THEN 'Ön Lisans'
        WHEN 3 THEN 'Lisans'
        WHEN 4 THEN 'Yüksek Lisans'
        WHEN 5 THEN 'Doktora'
    END AS egitim_seviyesi,
    COUNT(*) AS calisan_sayisi
FROM hr_attrition
GROUP BY Education
ORDER BY Education;
```

---

### S4 · Mesafesi 20 km'den fazla olan VE fazla mesai yapan çalışanlar

```sql
SELECT
    EmployeeNumber,
    JobRole,
    Department,
    DistanceFromHome,
    OverTime,
    Attrition
FROM hr_attrition
WHERE DistanceFromHome > 20
  AND OverTime = 'Yes'
ORDER BY Attrition DESC, DistanceFromHome DESC;
```

> 💡 `Attrition DESC` → 'Yes' alfabetik olarak 'No'dan büyük, ayrılanlar üstte gelir.

---

### S5 · En yüksek maaş alan ilk 10 çalışan

```sql
SELECT
    EmployeeNumber,
    JobRole,
    Department,
    MonthlyIncome,
    Attrition
FROM hr_attrition
ORDER BY MonthlyIncome DESC
LIMIT 10;
```

---

## 🟡 Bölüm 2 — Orta Seviye

> **Konular:** `GROUP BY` · `HAVING` · Aggregate Fonksiyonlar · `Subquery`

---

### S6 · Ortalama maaşın üzerinde olan departmanlar

```sql
SELECT
    Department,
    ROUND(AVG(MonthlyIncome), 2) AS ort_maas
FROM hr_attrition
GROUP BY Department
HAVING AVG(MonthlyIncome) > (
    SELECT AVG(MonthlyIncome)
    FROM hr_attrition
)
ORDER BY ort_maas DESC;
```

> 💡 `HAVING` içinde `WHERE` gibi subquery kullanabilirsin — bu klasik bir pattern.

---

### S7 · En yüksek ayrılma oranına sahip 5 iş rolü

```sql
SELECT
    JobRole,
    COUNT(*)                                                            AS toplam,
    SUM(CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END)                AS ayrilan,
    ROUND(
        100.0 * SUM(CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END)
              / COUNT(*),
        2
    )                                                                   AS ayrilma_orani_pct
FROM hr_attrition
GROUP BY JobRole
ORDER BY ayrilma_orani_pct DESC
LIMIT 5;
```

---

### S8 · Medeni durum × cinsiyet bazında iş tatmini ve çevre memnuniyeti

```sql
SELECT
    MaritalStatus,
    Gender,
    ROUND(AVG(JobSatisfaction), 2)          AS ort_is_tatmini,
    ROUND(AVG(EnvironmentSatisfaction), 2)  AS ort_cevre_memnuniyeti,
    COUNT(*)                                AS calisan_sayisi
FROM hr_attrition
GROUP BY MaritalStatus, Gender
ORDER BY MaritalStatus, Gender;
```

---

### S9 · En az 5 şirkette çalışmış ama bu şirkette 3 yıldan az olanlar — hangi eğitim alanından?

```sql
SELECT
    EducationField,
    COUNT(*)                        AS calisan_sayisi,
    ROUND(AVG(MonthlyIncome), 0)    AS ort_maas
FROM hr_attrition
WHERE NumCompaniesWorked >= 5
  AND YearsAtCompany < 3
GROUP BY EducationField
ORDER BY calisan_sayisi DESC;
```

---

### S10 · Şirket ortalamasının üzerinde deneyimli ama iş-yaşam dengesi kötü olanlar

```sql
SELECT
    Department,
    COUNT(*) AS kotu_denge_calisan
FROM hr_attrition
WHERE WorkLifeBalance = 1
  AND TotalWorkingYears > (
      SELECT AVG(TotalWorkingYears)
      FROM hr_attrition
  )
GROUP BY Department
ORDER BY kotu_denge_calisan DESC;
```

---

## 🔴 Bölüm 3 — İleri Seviye

> **Konular:** Window Functions (`RANK`, `AVG OVER`, `NTILE`) · `CTE` (`WITH`) · Çok Adımlı Analiz

---

### S11 · Her departmanda maaş sıralaması — ilk 3'ü getir

```sql
WITH maas_siralama AS (
    SELECT
        EmployeeNumber,
        Department,
        JobRole,
        MonthlyIncome,
        Attrition,
        RANK() OVER (
            PARTITION BY Department
            ORDER BY MonthlyIncome DESC
        ) AS maas_sirasi
    FROM hr_attrition
)
SELECT *
FROM maas_siralama
WHERE maas_sirasi <= 3
ORDER BY Department, maas_sirasi;
```

> 💡 `RANK()` aynı maaşa eşit sıra verir. Bağ olmadan sıra istersen `ROW_NUMBER()` kullan.

---

### S12 · Her çalışanın maaşını kendi departman ortalamasıyla karşılaştır

```sql
SELECT
    EmployeeNumber,
    Department,
    JobRole,
    MonthlyIncome,
    ROUND(AVG(MonthlyIncome) OVER (PARTITION BY Department), 2)  AS dept_ort_maas,
    MonthlyIncome
        - ROUND(AVG(MonthlyIncome) OVER (PARTITION BY Department), 2) AS maas_farki,
    CASE
        WHEN MonthlyIncome > AVG(MonthlyIncome) OVER (PARTITION BY Department)
        THEN 'Ortalama Üstü'
        ELSE 'Ortalama Altı'
    END AS maas_durumu
FROM hr_attrition
ORDER BY Department, maas_farki DESC;
```

> 💡 `AVG() OVER (PARTITION BY ...)` — her satırda tablo daralmadan grup ortalaması hesaplar. Subquery yazmana gerek yok.

---

### S13 · Eğitim alma sıklığına göre ayrılma oranı nasıl değişiyor?

```sql
SELECT
    TrainingTimesLastYear,
    COUNT(*)                                                              AS toplam,
    SUM(CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END)                  AS ayrilan,
    ROUND(
        100.0 * SUM(CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END)
              / COUNT(*),
        2
    )                                                                     AS ayrilma_orani_pct,
    ROUND(AVG(MonthlyIncome), 0)                                         AS ort_maas
FROM hr_attrition
GROUP BY TrainingTimesLastYear
ORDER BY TrainingTimesLastYear;
```

---

### S14 · "Yüksek Risk" Çalışan Profili — Risk Skoru Modeli

Aşağıdaki kriterlerden **en az 3'ünü** karşılayan çalışanlar yüksek risk altındadır:

| # | Kriter |
|---|--------|
| a | `OverTime = 'Yes'` |
| b | `JobSatisfaction <= 2` |
| c | `WorkLifeBalance <= 2` |
| d | `YearsSinceLastPromotion >= 4` |
| e | `DistanceFromHome > 15` |

```sql
WITH risk_skoru AS (
    SELECT
        EmployeeNumber,
        JobRole,
        Department,
        Attrition,
        (
            CASE WHEN OverTime = 'Yes'               THEN 1 ELSE 0 END +
            CASE WHEN JobSatisfaction <= 2            THEN 1 ELSE 0 END +
            CASE WHEN WorkLifeBalance <= 2            THEN 1 ELSE 0 END +
            CASE WHEN YearsSinceLastPromotion >= 4    THEN 1 ELSE 0 END +
            CASE WHEN DistanceFromHome > 15           THEN 1 ELSE 0 END
        ) AS risk_puani
    FROM hr_attrition
)
SELECT
    risk_puani,
    COUNT(*)                                                              AS calisan_sayisi,
    SUM(CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END)                  AS ayrilan,
    ROUND(
        100.0 * SUM(CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END)
              / COUNT(*),
        2
    )                                                                     AS ayrilma_orani_pct
FROM risk_skoru
GROUP BY risk_puani
ORDER BY risk_puani DESC;
```

> 💡 Risk puanı arttıkça ayrılma oranının nasıl sıçradığını gözlemle — bu bir korelasyon analizidir.

---

### S15 · İş seviyesine göre maaş çeyrekleri ve ayrılma oranı (`NTILE`)

```sql
WITH maas_ceyrekleri AS (
    SELECT
        EmployeeNumber,
        JobLevel,
        MonthlyIncome,
        Attrition,
        NTILE(4) OVER (
            PARTITION BY JobLevel
            ORDER BY MonthlyIncome
        ) AS maas_ceyregi   -- 1=En Düşük · 4=En Yüksek
    FROM hr_attrition
)
SELECT
    JobLevel,
    maas_ceyregi,
    COUNT(*)                                                              AS calisan_sayisi,
    ROUND(MIN(MonthlyIncome), 0)                                         AS min_maas,
    ROUND(MAX(MonthlyIncome), 0)                                         AS max_maas,
    ROUND(AVG(MonthlyIncome), 0)                                         AS ort_maas,
    SUM(CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END)                  AS ayrilan,
    ROUND(
        100.0 * SUM(CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END)
              / COUNT(*),
        2
    )                                                                     AS ayrilma_orani_pct
FROM maas_ceyrekleri
GROUP BY JobLevel, maas_ceyregi
ORDER BY JobLevel, maas_ceyregi;
```

> 💡 `NTILE(4)` veriyi 4 eşit gruba böler. Burada her `JobLevel` kendi içinde 4'e bölünüyor.

---

## 🏆 Bonus — Gerçek Hayat Senaryosu

### S16 · İK Direktörü için Attrition Dashboard (Tek Sorgu)

```sql
SELECT
    Department,
    COUNT(*)                                                              AS toplam_calisan,
    ROUND(AVG(Age), 1)                                                   AS ort_yas,
    ROUND(
        100.0 * SUM(CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END)
              / COUNT(*),
        2
    )                                                                     AS ayrilma_orani_pct,
    ROUND(AVG(MonthlyIncome), 0)                                         AS ort_maas,
    ROUND(
        100.0 * SUM(CASE WHEN OverTime = 'Yes' THEN 1 ELSE 0 END)
              / COUNT(*),
        2
    )                                                                     AS fazla_mesai_orani_pct,
    ROUND(AVG(JobSatisfaction), 2)                                       AS ort_is_tatmini_puan,
    CASE
        WHEN AVG(JobSatisfaction) >= 3.5 THEN '🟢 Çok Yüksek'
        WHEN AVG(JobSatisfaction) >= 2.5 THEN '🟡 Yüksek'
        WHEN AVG(JobSatisfaction) >= 1.5 THEN '🟠 Orta'
        ELSE                                   '🔴 Düşük'
    END                                                                   AS is_tatmini_etiketi
FROM hr_attrition
GROUP BY Department
ORDER BY ayrilma_orani_pct DESC;
```

---

## 📚 Kavram Özeti

| Bölüm | Konular | Sorular |
|---|---|---|
| 🟢 Başlangıç | SELECT, WHERE, ORDER BY, LIMIT, CASE | S1 – S5 |
| 🟡 Orta | GROUP BY, HAVING, Aggregate, Subquery | S6 – S10 |
| 🔴 İleri | RANK, AVG OVER, NTILE, CTE (WITH) | S11 – S15 |
| 🏆 Bonus | Çok metrikli tek sorgu dashboard | S16 |

---

> 📝 **İpucu:** Sorularda takıldığında önce küçük parçaları çalıştır, sonra birleştir.  
> Örnek: S14'te önce sadece `risk_skoru` CTE'sini SELECT et, sonra gruplama ekle.
