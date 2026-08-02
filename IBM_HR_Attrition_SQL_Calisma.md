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


## 🟢 Bölüm 1 

---

### S1 · Kaç çalışan var? Kaçı işten ayrılmış?

```sql
-- Toplam çalışan
SELECT COUNT(*) AS toplam_calisan
FROM `sql-practise-491318.IBM.IBM_HR`;

-- İşten ayrılanlar
SELECT COUNT(*) AS ayrilan_calisan
FROM `sql-practise-491318.IBM.IBM_HR` AS hr
WHERE hr.Attrition IS TRUE;
```


---

### S2 · Sales departmanı çalışanlarını maaşa göre büyükten küçüğe sırala

```sql
SELECT hr.EmployeeNumber, hr.Department, hr.MonthlyIncome 
FROM `sql-practise-491318.IBM.IBM_HR` AS hr
WHERE hr.Department = "Sales"
ORDER BY hr.MonthlyIncome DESC;
```

---

### S3 · Eğitim seviyelerini Türkçe etiketle, her seviyeden kaç çalışan var?

```sql
SELECT
CASE 
  WHEN hr.Education = 1 THEN "Lise"
  WHEN hr.Education = 2 THEN "Önlisans"
  WHEN hr.Education = 3 THEN "Lisans"
  WHEN hr.Education = 4 THEN "Yüksek Lisans"
  WHEN hr.Education = 5 THEN "Doktora"
 
END AS egitim_seviyesi,
COUNT(hr.EmployeeNumber) AS calisan_sayisi

FROM `sql-practise-491318.IBM.IBM_HR` AS hr
GROUP BY egitim_seviyesi, hr.Education
ORDER BY hr.Education;
```

---

### S4 · Mesafesi 20 km'den fazla olan VE fazla mesai yapan çalışanlar

```sql
SELECT * FROM `sql-practise-491318.IBM.IBM_HR` AS hr
WHERE hr.DistanceFromHome > 20 AND
hr.OverTime IS TRUE;
```


---

### S5 · En yüksek maaş alan ilk 10 çalışan

```sql
SELECT * FROM `sql-practise-491318.IBM.IBM_HR` AS hr 
ORDER BY hr.MonthlyIncome DESC
LIMIT 10;
```

---

## 🟡 Bölüm 2 

---

### S6 · Ortalama maaşın üzerinde olan departmanlar

```sql
SELECT hr.Department, ROUND(AVG(hr.MonthlyIncome),2) AS avg_income 
FROM `sql-practise-491318.IBM.IBM_HR` AS hr
GROUP BY hr.Department
HAVING avg_income > (SELECT AVG(monthlyincome) FROM `sql-practise-491318.IBM.IBM_HR`)
```

---

### S7 · En yüksek ayrılma oranına sahip 5 iş rolü

```sql
WITH attrition_rate AS(
SELECT hr.JobRole, COUNT(hr.Attrition) AS toplam, 
(SELECT COUNT(Attrition) 
  FROM `sql-practise-491318.IBM.IBM_HR` 
    WHERE Attrition = TRUE AND JobRole = hr.jobrole
    GROUP BY JobRole) AS ayrilik_true
FROM `sql-practise-491318.IBM.IBM_HR` AS hr
GROUP BY hr.JobRole
)

SELECT jobrole, ROUND((ayrilik_true/toplam)*100, 2)AS rate FROM attrition_rate
ORDER BY rate DESC
LIMIT 5;
```

---

### S8 · Medeni durum × cinsiyet bazında iş tatmini ve çevre memnuniyeti

```sql
SELECT hr.MaritalStatus, hr.gender,
    ROUND(AVG(hr.JobSatisfaction),2) AS js,
    ROUND(AVG(hr.EnvironmentSatisfaction),2) AS es 
FROM `sql-practise-491318.IBM.IBM_HR` AS hr
GROUP BY hr.MaritalStatus, hr.Gender
```

---

### S9 · En az 5 şirkette çalışmış ama bu şirkette 3 yıldan az olanlar — hangi eğitim alanından?

```sql
SELECT hr.EducationField,
    COUNT(hr.EmployeeNumber) AS sayi,
    ROUND(AVG(hr.MonthlyIncome),2) AS avg_salary
FROM `sql-practise-491318.IBM.IBM_HR` AS hr
WHERE hr.NumCompaniesWorked >= 5 AND hr.YearsAtCompany < 3
GROUP BY hr.EducationField
ORDER BY sayi DESC;
```

---

### S10 · Şirket ortalamasının üzerinde deneyimli ama iş-yaşam dengesi kötü olanlar

```sql
WITH wlb AS(
SELECT hr.department, hr.EmployeeNumber, hr.TotalWorkingYears, hr.WorkLifeBalance
FROM `sql-practise-491318.IBM.IBM_HR` AS hr
WHERE hr.TotalWorkingYears > (
  SELECT AVG(totalworkingyears) 
    FROM `sql-practise-491318.IBM.IBM_HR`
  ) AND
  hr.WorkLifeBalance = 1
)

SELECT department, COUNT(employeenumber)AS calisan_sayi FROM wlb
GROUP BY department
ORDER BY calisan_sayi DESC
          
```

---

## 🔴 Bölüm 3 


---

### S11 · Her departmanda maaş sıralaması — ilk 3'ü getir

```sql
WITH calisan_gelir AS(
SELECT hr.Department, hr.EmployeeNumber, hr.MonthlyIncome,
RANK() OVER(
  PARTITION BY hr.Department ORDER BY hr.MonthlyIncome DESC
) AS aylik_gelir
FROM `sql-practise-491318.IBM.IBM_HR` AS hr
)

SELECT * FROM calisan_gelir
WHERE aylik_gelir <= 3;
```


---

### S12 · Her çalışanın maaşını kendi departman ortalamasıyla karşılaştır

```sql
WITH avg_incomeCTE AS(
SELECT hr.EmployeeNumber, hr.Department, hr.MonthlyIncome, 
ROUND(AVG(hr.MonthlyIncome)OVER(
  PARTITION BY hr.Department
),2)AS avg_income_bydept
FROM `sql-practise-491318.IBM.IBM_HR` AS hr
)

SELECT *,
ROUND(monthlyincome - avg_income_bydept) AS fark,
CASE 
  WHEN monthlyincome > avg_income_bydept THEN "Ortalama Üstü"
  ELSE "Ortalama Altı"
END AS maas_durumu
FROM avg_incomeCTE;
```


---

### S13 · Eğitim alma sıklığına göre ayrılma oranı nasıl değişiyor?

```sql
SELECT
    TrainingTimesLastYear,
    COUNT(*) AS toplam,
    SUM(CASE WHEN Attrition = TRUE THEN 1 ELSE 0 END) AS ayrilan,
    ROUND(100.0 * SUM(CASE WHEN Attrition = TRUE THEN 1 ELSE 0 END) / COUNT(*), 2) AS oran,
    ROUND(AVG(MonthlyIncome)) AS ort_maas
FROM `sql-practise-491318.IBM.IBM_HR`
GROUP BY TrainingTimesLastYear
ORDER BY TrainingTimesLastYear
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
WITH risk_skor AS(
SELECT hr.EmployeeNumber, hr.attrition, hr.OverTime, hr.JobSatisfaction, hr.WorkLifeBalance, hr.YearsSinceLastPromotion, hr.DistanceFromHome,
CASE
  WHEN hr.OverTime = True THEN 1 ELSE 0
END AS risk_skor1,

CASE
  WHEN hr.JobSatisfaction <= 2 THEN 1 ELSE 0
END AS risk_skor2,

CASE
  WHEN hr.WorkLifeBalance <= 2 THEN 1 ELSE 0
END AS risk_skor3,

CASE
  WHEN hr.YearsSinceLastPromotion >= 4 THEN 1 ELSE 0
END AS risk_skor4,

CASE
  WHEN hr.DistanceFromHome > 15 THEN 1 ELSE 0
END AS risk_skor5,

FROM `sql-practise-491318.IBM.IBM_HR` AS hr
),

toplam_skorCTE AS(
SELECT *, (risk_skor1+risk_skor2+risk_skor3+risk_skor4+risk_skor5) AS toplam_skor
FROM risk_skor
)

SELECT toplam_skor, COUNT(EmployeeNumber) AS toplam_calisan, 
  SUM(CASE WHEN attrition = TRUE THEN 1 ELSE 0 END) AS ayrilan_sayi,
  ROUND(100.0 * SUM(CASE WHEN attrition = TRUE THEN 1 ELSE 0 END) / COUNT(*), 2) AS ayrilma_orani
FROM toplam_skorCTE
GROUP BY toplam_skor
ORDER BY ayrilma_orani DESC


```


---

### S15 · İş seviyesine göre maaş çeyrekleri ve ayrılma oranı (`NTILE`)

```sql

```

> 💡 `NTILE(4)` veriyi 4 eşit gruba böler. Burada her `JobLevel` kendi içinde 4'e bölünüyor.

---

## 🏆 Bonus — Gerçek Hayat Senaryosu

### S16 · İK Direktörü için Attrition Dashboard (Tek Sorgu)

```sql

```

---
