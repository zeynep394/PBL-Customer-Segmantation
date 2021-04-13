# PBL-Customer-Segmantation

Costumer Segmantation homework using clustering

## Learn Clustering with Python and Pandas

Merhaba, bu ödevde costumer segmantation verisiyle çalışarak clusteringin mantığını öğreneceğiz. Clustering gerçekleştirmek için bir çok algoritma mevcut ancak biz bugün sadece bir tanesini ele alacağız: K-means clustering. Hazırsanız haydi başlayalım!

Takip edeceğimiz adımlar kabaca şu şekilde:

- Veri Ön Hazırlığı
- Benzerlik Matrisi Hazırlama
- K-means Algoritmasının Çalıştırılması
- Sonuçların İşlenmesi

### Veri Ön Hazırlığı
Kullanacağımız veri seti:

[recommendationdata.csv](https://github.com/zeynep394/AIZA-Costumer-Segmantation/blob/main/data.csv) - bir şirkete ait müşteri, ürün ve sipariş bilgilerini içeren csv formatındaki veri seti. 

Önce kullanacağımız modülleri import ediyoruz: 

```python
import pandas as pd
from sklearn.cluster import KMeans
from sklearn.metrics import silhouette_score
import matplotlib.pyplot as plt
import numpy as np
%matplotlib inline
```
```
NOT: Elimizdeki data.csv isimli excel dosyasını okuyabilmek için `df=pd.read_csv('data.csv')`

kullanıyoruz, ancak bu aşamada bir utf-8 hatası aldık bunu gidermek için encodingi değiştirmemiz gerekiyor ;

`df=pd.read_csv('data.csv',encoding='latin1')` Şimdi veri setimizi incelemeye başlayabiliriz.
```

**İstenilenler**

Bizden istenilenleri şu şekilde özetleyebiliriz:

- Bu proje için gerçekleştirilecek ilk adım veri setinin import edilmesi ve üç tane veri yapısı oluşturulması
  - Customer
  - Products
  - Orders
- Daha sonra bu üç veri yapısını bir SQLite tablosuna dönüştür.
- Tablodaki değerleri veritabanından çekerek tek bir veri yapısı üzerinde birleştir.
- Segment the customers using KMeans and highlight the characteristics
of the segments
- Selection of number of customers segments should be justified
- Create RFM table ( Recency, Frequency and Money)

Veri ön hazırlamasının en önemli adımı olan null değerleri ele alacağız. Verimize baktığımızda bazı sütünların olması gerekenden 10 kat az veri içerdiğini görebiliyoruz. Bu tarz sütunlar için null değerleri tahmin etmek veya onları mod veya medyan alma gibi işlemler söz konusu olamaz yani bu sütunları verimizden çıkarmamız gerekir. Dataframedeki tüm sütunları dolaşarak null değer sayısı 1000den fazla olan tüm sütunları dataframe'imizden çıkarıyoruz. 

```python
for col in df.columns:
    if df[col].notnull().sum()<1000:
        df.drop(col, inplace=True, axis=1)
```

Bazı sütunlar ise yine eksik veri içermesine rağmen eksik veri sayısı makul bir değer olduğundan bu eksikler tahmin yöntemiyle doldurulabilir bu sayede bu tarz sütunları veri setimizde tutmaya devam edebiliriz. Eğer null değer ieren tüm sütunları dataframeden çıkarmaya kalkarsak bu tahminlerimizi olumsuz etkileyebilir bu noktaya dikkat etmek gerekir.

```python
for co in df.select_dtypes(include='object').columns:
    df[co].fillna(df[co].mode()[0], inplace=True)
    
for co in df.select_dtypes(include=['float64']).columns:
    df[co].fillna(df[co].mean(), inplace=True)
```

Başlangıç için elimizdeki veriyi import ederek üç farklı kategoriye ayıralım.
Veri import edildikten sonra görüleceği gibi customer, orders ve products olmak üzere üç farklı başlık çeşidi içeriyor. Verileri ayrı dataframe'lere bölebilmek için bu başlık isimlerinden yararlanacağız.

```python
df=pd.read_csv('data.csv',encoding='latin1')


df_customer = df.filter(regex='Customers.')
df_product = df.filter(regex='Products.')
df_order = df.filter(regex='Orders.')
```
