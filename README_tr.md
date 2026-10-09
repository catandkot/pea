<div align="center">

[한국어](README.md) · [简体中文](README_zh.md) · [繁體中文](README_zh-TW.md) · [日本語](README_ja.md) · **Türkçe** · [Русский](README_ru.md) · [Tiếng Việt](README_vi.md) · [ไทย](README_th.md) · [Deutsch](README_de.md) · [Español](README_es.md) · [Français](README_fr.md) · [Українська](README_uk.md) · [Polski](README_pl.md) · [Português (Brasil)](README_pt-BR.md) · [العربية](README_ar.md) · [فارسی](README_fa.md)

# 🫛 PEA

### Akıllı tarım ajanı — yetiştirme sahasındaki veriyi okur, değerlendirir ve harekete geçer.

**Sensörler zaten konuşuyor. PEA konuştuklarını anlamanızı sağlar.**

[![license](https://img.shields.io/badge/license-MIT-f4c430?style=flat-square)](LICENSE)
[![status](https://img.shields.io/badge/status-concept-5c4ee5?style=flat-square)](#proje-durumu)

[Hızlı başlangıç](#hızlı-başlangıç) · [Ne yapar?](#ne-yapar) · [Nasıl çalışır](#nasıl-çalışır) · [Güvenlik ilkeleri](#güvenlik-ilkeleri) · [Yol haritası](#yol-haritası)

</div>

---

PEA (bezelye), sera ve açık alan yetiştirme sahaları için bir **akıllı tarım ajanıdır**. Toprak nemi, sıcaklık ve bağıl nem, güneş radyasyonu, CO2, drenaj EC/pH gibi sensör değerlerini toplayarak *şu anda olup biteni* açıklar ve sulama, havalandırma, gölgeleme, ilave aydınlatma gibi aktüatörleri **güvenli sınırlar içinde** kontrol eder.

Her kararın bir gerekçesi vardır. PEA **gözlemlenen**, **çıkarım yapılan** ve **henüz bilinmeyen** değerleri birbirinden ayırarak raporlar. "Toprak kuru" bir sensör geçmişidir, "öğleden sonra yüksek hastalık riski var" bir çıkarımdır, "sulama vanası gerçekten açıldı mı" ise debimetre bunu doğrulayana kadar bilinmezdir.

İş, çiftliğin içinde kalır. Dışarıya giden veri yalnızca kullanıcının açıkça seçtiği kalemlerdir.

## Hızlı başlangıç

> **Aşağıdaki komutlar ve paket adları henüz kesinleşmedi.** Şu anda npm üzerinde yayınlanmış bir `pea` paketi yok. Yayın sırasında bu belgeyi gerçek komutlarla güncelleyeceğiz.

```bash
# 1) install (package not published yet)
npx pea-agents setup

# 2) readiness check — gateway, sensors and actuators
pea doctor

# 3) ask about today's greenhouse
pea analyze ./farm.toml
```

Ajana bağlandıktan sonra şöyle sorabilirsiniz.

```text
3 numaralı seranın sabah sulama miktarı hedefin %12 üzerinde. Sebebi ne?
Drenaj oranını %25'e çekmek için bu öğleden sonraki sulama programını nasıl değiştirmeliyim?
Dün geceki en düşük sıcaklığa göre yaprak ıslaklığının kaç saat sürdüğünü göster.
```

## Ne yapar?

| Alan | PEA'nın yaptığı iş | Kullanılan veri |
| --- | --- | --- |
| Yetiştirme ortamı | Sera içi sıcaklık ve nem, VPD, güneş radyasyonu birikim eğilimleri ve anormal aralıkların gösterimi | Sıcaklık-nem sensörü, radyasyon ölçer |
| Sulama ve besin çözeltisi | Radyasyon birikimine dayalı sulama miktarı hesabı, drenaj oranının hedefe göre sapma uyarısı | Toprak nemi ve tartım göstergesi, drenaj EC/pH |
| Havalandırma ve iklimlendirme | VPD hedefini korumaya yönelik kontrol, havalandırma penceresi ve fan kademesi önerileri | Pencere açıklığı geri bildirimi, dış hava durumu |
| Hastalık ve zararlı uyarısı | Yaprak ıslaklık süresi ile sıcaklık-nem bileşiminden mantar hastalığı riskinin hesaplanması | Sıcaklık-nem geçmişi, yaprak ıslaklık sensörü |
| Yetiştirme günlüğü | Sulama, çevre ve iş geçmişinin otomatik kaydı, yetiştirme gözlemi girişine destek | İş kaydı, saha fotoğrafları |
| Hasat tahmini | Birikimli sıcaklığa dayalı hasat zamanı tahmini, dönem karşılaştırması | Sıcaklık geçmişi, çeşit parametreleri |

Tablodaki kalemler ve eşik değerler **her ürün, çeşit ve bölge için yeniden kalibre edilmesi gereken değerlerdir**. PEA yalnızca varsayılan değerler önerir; bunları o çiftliğin referans değeri olarak kesinleştirmez.

## Nasıl çalışır

Ajan MCP üzerinden PEA'ya soru sorduğunda PEA veriyi ağ geçidinden toplar ve sonucu gerekçesiyle birlikte döndürür. Ajan bu sonuçla durumu açıklar veya bir kontrol önerisi sunar. Aynı iş akışı terminalde CLI ile de birebir kullanılır.

```text
Ajan → MCP → PEA → Ağ geçidi → Sensör
                            ← Gözlem + çıkarım + bilinmeyen
```

- **Önce yerel** — Veri toplama ve değerlendirme çiftliğin içinde yapılır.
- **Önce kanıt** — Sonuçla birlikte ham değer, zaman ve kaynak saklanır.
- **Bilinmeyen bilinmeyen olarak kalır** — Sensörün olmadığı veya iletişimin koptuğu aralıklar tahminle doldurulmaz, açıkça belirtilir.
- **Aynı sonuç** — CLI ve MCP aynı iş akışını ve aynı sonuç sözleşmesini paylaşır.

## Desteklenen bağlantılar

| Entegrasyon | Yöntem | Durum |
| --- | --- | --- |
| MQTT ağ geçidi | Broker aboneliğiyle sensör konularının toplanması | Planlandı |
| Modbus RTU / TCP | Denetleyici yazmaçlarının yoklanması | Planlandı |
| LoRaWAN | Ağ geçidi üzerinden açık alan sensörleri | Planlandı |
| Hava durumu verisi | Yerel hava durumu API'si, dış ortam sıcaklığı, nemi ve yağış | Planlandı |
| Mevcut yetiştirme yönetim sistemi | Dosya içe aktarma (CSV/JSON) ile geçmişin birleştirilmesi | Planlandı |

## Güvenlik ilkeleri

Aktüatörleri hareket ettirmek geri alınamaz. Bu nedenle PEA şunlara uyar.

1. **Varsayılan olarak insan onayı** — Sulama, havalandırma, ilave aydınlatma gibi gerçek işlemler önce planı yazdırır ve onay alır.
2. **Sert sınırlama** — Çiftlikte tanımlanan üst sınırlar (tek seferde en fazla sulama miktarı, izin verilen sıcaklık aralığı vb.) ajan kararıyla aşılamaz.
3. **Güvenli duruma düşme** — İletişim kopması veya elektrik kesintisinde sistem güvenli tarafa (sulamayı durdurma, havalandırmayı açma vb.) geçer.
4. **İlaçlama otomatikleştirilmez** — Tarım ilacı ve gübre uygulamasını PEA yürütmez; yalnızca zamanlama ve gerekçe önerir.
5. **Tam kayıt** — Her kontrol için hedef, zaman, gerekçe ve uygulama sonucu saklanır.

## Yol haritası

- **M0 — Konsept ve belgeler** *(güncel)*: Ürün tanımı, güvenlik ilkeleri, sonuç sözleşmesi tasarımı
- **M1 — Toplama**: Ağ geçidi entegrasyonu, sensör geçmişinin saklanması, çevre eğilimi sorgulama
- **M2 — Öneri**: Sulama ve havalandırma danışmanı, hastalık ve zararlı riski, yetiştirme günlüğü otomasyonu
- **M3 — Onay tabanlı kontrol**: İnsan onayı altında otomatik kontrol ve güvenli duruma düşme
- **M4 — İşletme**: Hasat tahmini, çoklu çiftlik karşılaştırması, dönem raporları

## Proje durumu

Bu belge **PEA'nın ürün konseptini** açıklar. Mevcut çalışma kopyasındaki kod bir tersine mühendislik aracıdır (`rea-agents`) ve yukarıda anlatılan toplama ve kontrol işlevlerini henüz uygulamamıştır. Komut adları, entegrasyon listesi ve eşik değerler uygulamayla birlikte kesinleşecektir. Çeviri README'ler henüz önceki belgeyi temel aldığından, yeni marka yapısına uygun olarak sırayla düzenlenecektir.

## Katkı

Hata bildirimleri ve pull request'ler memnuniyetle karşılanır. Sahadan gelen gözlemler, başarısızlık örnekleri ve ürüne özel kalibrasyon değerleri özellikle değerlidir.

## Lisans

[MIT](LICENSE)
