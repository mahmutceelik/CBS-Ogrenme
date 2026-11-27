# CBS Öğrenme Rehberi 🗺️

Bu klasör (`CBS-Ogrenme`, ana klasörünüzde) sizin çalışma alanınız. İçinde:

| Dosya | Ne işe yarar |
|---|---|
| `REHBER.md` | Bu rehber |
| `Ders1_Ilk_Haritam.ipynb` | Python ile adım adım ilk harita dersi |
| `jupyter-ac.command` | Çift tıklayınca Python dersini açar |
| `veri/turkiye_iller.geojson` | 81 ilin sınırları (alan / poligon) |
| `veri/sehirler.csv` | 14 yerin enlem-boylamı (Excel'de de açılır) |

---

## Bölüm 0 — Temel kavramlar (5 dakikalık okuma)

**CBS (Coğrafi Bilgi Sistemleri)** = konumu olan verileri saklama, analiz etme ve haritalama.

### Vektör ve raster
| | Vektör | Raster |
|---|---|---|
| Ne? | Nokta, çizgi, alan (poligon) | Piksellerden oluşan görüntü |
| Örnek | Okul (nokta), yol (çizgi), il (alan) | Uydu görüntüsü, yükseklik modeli (DEM) |
| Dosya | .gpkg, .shp, .geojson, .kml | .tif (GeoTIFF), .jpg, .png |
| Bilgi | Her nesnenin bir **öznitelik tablosu** satırı var | Her pikselin bir **değeri** var |

### Katman (layer)
Harita, üst üste konmuş saydam katmanlardan oluşur — tıpkı asetat kâğıtları gibi. Üstteki katman alttakini örter.

### Koordinat Referans Sistemi (KRS / CRS) ⚠️ en önemlisi
- **EPSG:4326 (WGS 84)** — enlem/boylam, **derece**. GPS, Google Earth.
- **EPSG:3857** — web haritaları (Google Maps, OpenStreetMap). Görsel için iyi, ölçüm için **kötü**.
- **UTM 35N/36N/37N** (EPSG:32635/32636/32637) — **metre**. Türkiye'nin batısı/ortası/doğusu.
- **TUREF / TM** (ör. EPSG:5254 TM33) — Türkiye'nin resmi sistemleri, tapu-kadastro.

> **Altın kural:** Alan veya mesafe ölçeceksen **metre** kullanan bir KRS'ye geç. Katmanlar üst üste düşmüyorsa önce KRS'yi kontrol et.

---

## Bölüm 1 — QGIS ilk adımlar

### Alıştırma 1.1: QGIS'i açmak
1. Launchpad → **QGIS**. İlk açılışta "internetten indirildi, açılsın mı?" derse **Aç**.
2. Dil sorarsa Türkçe seçebilirsiniz; ancak internetteki dersler çoğunlukla İngilizce menüyü kullanır. Bu rehber menü adlarını **İngilizce** verir (parantezde Türkçe).

Ekranın bölümleri:
- **Sol üst — Browser (Tarayıcı):** bilgisayarınızdaki dosyalar.
- **Sol alt — Layers (Katmanlar):** haritaya eklediğiniz katmanlar.
- **Orta — Harita tuvali.**
- **Sağ alt köşe:** fare koordinatı, ölçek ve **KRS (EPSG:xxxx)** düğmesi.

### Alıştırma 1.2: Altlık harita eklemek
1. Sol üstteki Browser'da **XYZ Tiles** → **OpenStreetMap**'e çift tıklayın.
2. Dünya haritası gelir. Fare tekerleğiyle yakınlaşıp Türkiye'yi bulun.
3. Sağ alttaki KRS'ye bakın: **EPSG:3857** (web haritası).

### Alıştırma 1.3: İl sınırlarını eklemek
1. Finder'da `CBS-Ogrenme/veri/turkiye_iller.geojson` dosyasını QGIS penceresine **sürükleyip bırakın**.
2. İller renkli poligonlar olarak gelir. KRS'si 4326 olduğu halde OSM'nin (3857) üstüne doğru oturur — QGIS **anında dönüştürme (on-the-fly)** yapıyor.
3. Layers'ta katmana sağ tık → **Open Attribute Table** (Öznitelik Tablosunu Aç). Her satır bir il!
4. Tabloda bir satıra tıklayın → haritada o il sarıya boyanır (seçim).

### Alıştırma 1.4: Görünümü değiştirmek (Symbology)
1. Katmana çift tıklayın → **Symbology** sekmesi.
2. Üstteki **Single Symbol**'ü **Categorized** yapın, Value: `shapeName`, **Classify** → her il ayrı renk.
3. **Labels** sekmesi → **Single Labels**, Value: `shapeName` → il adları haritada yazar.
4. Saydamlık: Symbology'de **Opacity**'yi %50 yapın ki alttaki OSM görünsün.

### Alıştırma 1.5: CSV'den nokta katmanı
1. Menü: **Layer → Add Layer → Add Delimited Text Layer**.
2. File name: `veri/sehirler.csv`.
3. **Geometry Definition → Point coordinates**: X field = `boylam`, Y field = `enlem`.
   ⚠️ X = boylam (doğu-batı), Y = enlem (kuzey-güney). Tersini yaparsanız noktalar okyanusa düşer!
4. Geometry CRS: **EPSG:4326**. → **Add**.

### Alıştırma 1.6: Alan hesabı (Field Calculator)
1. İller katmanının öznitelik tablosunu açın → kalem simgesi (düzenleme modu) → **Field Calculator** (abaküs simgesi).
2. Output field name: `alan_km2`, type: Decimal.
3. Expression: `$area / 1000000` → OK. Kalemle düzenlemeyi kapatıp kaydedin.
   > `$area`, proje ayarlarındaki elipsoide göre **metrekare** hesaplar (Project → Properties → General → Ellipsoid). Bu yüzden veri derece olsa da sonuç doğru çıkar.
4. Konya ~41.000 km² olmalı.

### Alıştırma 1.7: Tematik harita
1. İller → Symbology → **Graduated**, Value: `alan_km2`, Color ramp: *Greens*, Mode: *Natural Breaks (Jenks)*, Classes: 5 → **Classify**.
2. Büyük iller koyu, küçükler açık yeşil. Bu bir **koroplet haritadır**.

### Alıştırma 1.8: Kaydetmek ve dışa aktarmak
- Projeyi kaydet: **Project → Save As** → `CBS-Ogrenme/ilk_proje.qgz`.
- Katmanı başka biçimde kaydet: katmana sağ tık → **Export → Save Features As** → Format: **GeoPackage**.
- Google Earth'e aktar: aynı menüden Format: **KML** → dosyaya çift tıklayınca Google Earth'te açılır! 🌍
- Haritayı resim/PDF yapmak: **Project → New Print Layout** (harita, başlık, lejant, kuzey oku, ölçek çubuğu ekleyin).

---

## Bölüm 2 — Google Earth Pro ile çalışmak
- **Yer imi (Placemark):** üstteki raptiye simgesi → bir nokta koyun → sol panelde sağ tık → **Save Place As** → `.kml`. Bu dosyayı QGIS'e sürükleyebilirsiniz.
- **Çokgen çizmek:** üstteki poligon simgesi → bir tarla/parsel çizin → KML olarak kaydedip QGIS'te alanını hesaplayın (Alıştırma 1.6).
- **Geçmiş görüntüler:** saat simgesi → aynı yerin yıllar içindeki uydu görüntüleri.
- **Yükseklik profili:** cetvel → Path çizin → sağ tık **Show Elevation Profile**.

---

## Bölüm 3 — Python dersi
1. `CBS-Ogrenme/jupyter-ac.command` dosyasına **çift tıklayın** (ilk seferde macOS sorarsa sağ tık → Aç).
2. Tarayıcıda JupyterLab açılır → solda `Ders1_Ilk_Haritam.ipynb` dosyasına çift tıklayın.
3. Hücreleri sırayla **Shift + Enter** ile çalıştırın.
4. Bitirince açılan Terminal penceresinde **Ctrl + C** ile kapatın.

---

## Bölüm 4 — Veritabanı (ileri seviye, acelesi yok)
- **DB Browser for SQLite:** `.gpkg` dosyasını açıp içindeki tabloları görebilirsiniz — GeoPackage aslında bir SQLite veritabanıdır.
- **Postgres.app + PostGIS:** Büyük/çok kullanıcılı veriler için. Açın → **Initialize** → çift tıklayınca açılan ekranda `CREATE EXTENSION postgis;` yazın. QGIS'te Browser → **PostgreSQL → New Connection** ile bağlanılır. Bunu birlikte yapalım.

---

## Ücretsiz veri kaynakları
| Kaynak | Ne var |
|---|---|
| **QuickMapServices** (QGIS eklentisi) | Hazır altlık haritalar. *Plugins → Manage and Install Plugins* → arayın |
| **OpenStreetMap** (QGIS eklentisi: *QuickOSM*) | Yollar, binalar, okullar… her şey |
| **geoBoundaries / GADM** | İdari sınırlar (il, ilçe) |
| **Copernicus / USGS EarthExplorer** | Uydu görüntüleri (Sentinel, Landsat), yükseklik (DEM) |
| **Natural Earth** | Dünya ölçeğinde ülke, nehir, şehir |

## Öğrenme kaynakları
- **QGIS Training Manual** (resmi, ücretsiz): docs.qgis.org → Training Manual
- **QGIS Tutorials and Tips** — Ujaval Gandhi: adım adım dersler
- **Automating GIS Processes** (Helsinki Üniv.) — Python ile CBS, ücretsiz

## Önerilen öğrenme sırası
1. ✅ Bölüm 0 kavramlarını okuyun
2. QGIS Alıştırma 1.1 → 1.8
3. Google Earth'te bir alan çizip QGIS'e aktarın
4. Python Ders 1
5. QuickOSM ile kendi ilçenizin yollarını/binalarını indirin
6. Raster: bir DEM (yükseklik) indirip eğim/bakı haritası yapın — bunu birlikte yapalım

> Takıldığınız her yerde ekran görüntüsü alıp (⌘ + Shift + 4) Claude'a sorun. Hata mesajlarını aynen kopyalayın.
