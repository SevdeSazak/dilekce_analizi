# Dilekçe Bilgi Çıkarımı
Bu proje, doğal dil işleme (NLP) tekniklerini kullanarak dilekçeler üzerinden **anlamlı bilgi çıkarımı** ve **anlamsal sınıflandırma** yapmayı amaçlamaktadır.PDF formatındaki dilekçelerden metin çıkarılarak; kişi, kurum, tarih gibi varlık isimleri tespit edilir ve metinlerin anlamlı sınıflara ayrılması sağlanır.  
**1- KULLANILAN KÜTÜPHANELER**  
from transformers import AutoTokenizer, AutoModelForTokenClassification, pipeline #NER modeli  ve Zero-shot (RoBERTa) için  
import re  
import pdfplumber #PDF den Metine dönüştürmek için  
import json #JSON olarak kaydetmek için  
import re  
import string  
import stanza #Lemmatization için 

**-** pdfplumber kütüphanesi, PDF içindeki metinleri sayfa bazında hassas ve düzenli şekilde çıkarma konusunda yüksek doğruluk sağlar. Bu nedenle, PDF dosyalarını metne dönüştürmek için pdfplumber kullanılmıştır.

**2- KULLANILAN MODELLER**  
**NER Modeli (BERTurk - savasy):** Metinlerden kişi isimleri, adresler ve kurum adları gibi temel bilgileri tespit etmek için kullanıldı. BERTurk modeli, Türkçe dilinde en çok tercih edilen ve yüksek başarı sağlayan modellerden biridir.  
**Zero-Shot Classification (RoBERTa tabanlı):** Dilekçelerin olay konusu ve talep açıklamalarını sınıflandırmak için kullanıldı. Etiketlenmemiş (etiketiz) veriler üzerinde iyi performans gösteren esnek bir modeldir.  
**Stanza:** Metinlerde lemmatization (kelime köklerine indirgeme) işlemi için kullanıldı.  
**Text Classification(Toxicity Detection):** Dilekçelerde dilin resmiyet seviyesinde argo ifadeleri tespit etmek amacıyla kullanıldı.  

  **3- AÇIK BİLGİ ÇIKARIMI**    
Ad Soyad tahmini için NER (PER etiketi) kullanılmıştır.  
Adres veya bölge bilgisi için NER (LOC etiketi) kullanılmıştır.  
Tarih aralıkları için NER (MISC etiketi) kullanılmış, daha geniş tanımlamalar için Regex ile desteklenmiştir.  
Kurum ismi için NER (ORG etiketi) ve Regex birlikte kullanılmıştır.  
Olay konusu, önceden tanımlanmış kategoriler (asansör, su, elektrik, çöp, yol, internet, gürültü, aydınlatma, ısınma, trafik, kargo) arasından Zero-Shot Classification yöntemi ile belirlenmiştir.  
Talep türü, “Çözüm Talebi”, “Açıklama / Talebi” veya “Denetim Talebi” olarak Zero-Shot Classification yöntemi ile sınıflandırılmıştır.    
 
  **4-  Yorumla Anlam Çıkarma (Inference)**  
 Metin içindeki gizli anlamları veya çıkarımları belirlemek için Kural Tabanlı bir yöntemi kullanılmıştır. “Tekrarlayan sorun”, “Uzun süredir devam eden sorun”, “Eksiklik Bildirimi” ve “Ciddi bir endişe olan sorun” olmak üzere 4 gruba ayrılmıştır.  

  **5- TON VE DİL ANALİZİ**  
  **Ton Analizi:** Dilekçelerde kullanılan dilin tonu; Öfkeli, Çaresiz, Kibar Şikâyet, Nesnel ve Sakin etiketleriyle, kural tabanlı bir yöntem kullanılarak analiz edilmiştir.  
    
  **Dilin Resmiyeti:** Dilekçelerde kullanılan dilin resmiyet seviyesi, kural tabanlı ve model destekli bir yaklaşımla analiz edilmiştir.Metin önce küçük harfe çevrilmiş, ardından başlık ve kapanış ifadeleri incelenmiştir.Eğer metinde “sayın”, “değerli” gibi resmi hitaplar ve “arz ederim”, “talep ediyorum” gibi resmi kapanışlar varsa, dil Resmi olarak etiketlenmiştir.Eğer metin bu kriterleri sağlamıyor ancak içinde argo veya saldırgan ifadeler varsa, toxic classification modeli (fc63/toxic-classification-model) kullanılarak değerlendirilmiş ve Argo olarak etiketlenmiştir.Her iki duruma da uymayan metinler ise Samimi olarak değerlendirilmiştir.  
    
**Yazım Kuralları:** Dilekçe metinlerinin yazım kurallarına uyumu, çeşitli basit kural ve kontrollerle değerlendirilmiştir.  
  Metnin ilk harfinin büyük olup olmadığı,  
  Cümlenin sonunda uygun noktalama işaretinin (nokta, ünlem, soru işareti) bulunup bulunmadığı,  
  Ardışık üç veya daha fazla noktalama işaretinin varlığı,  
  Noktalama işaretlerinden sonra boşluk bırakılmaması gibi yazım hataları,  
  NER modelinden destek alınarak kişi, kurum ve yer isimlerinin ilk harfinin büyük olması gerekliliği,   
  Cümlelerin ilk harflerinin büyük olup olmadığı kontrol edilmiştir.  
Bu kurallara göre toplamda tespit edilen hata sayısına göre yazım uyumu Yüksek, Orta veya Düşük olarak sınıflandırılmıştır.  

