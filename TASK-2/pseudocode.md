/* Veri yapıları */
SEPET = {
  items: []  // her öğe: { productId, isim, fiyat, adet, stok, vergiler, seçenekler }
  userId: null   // giriş yapılmışsa kullanıcı id'si
  promos: []     // uygulanmış kupon/indirim kodları
  currency: "TRY"
}

/* Yardımcı fonksiyonlar */
Fonksiyon BulItem(productId):
  için her item in SEPET.items:
    eğer item.productId == productId ise dön item
  dön null

Fonksiyon HesaplaAraToplam():
  total = 0
  için her item in SEPET.items:
    total = total + (item.fiyat * item.adet)
  dön total

Fonksiyon HesaplaVergi():
  vergi = 0
  için her item in SEPET.items:
    vergi = vergi + ((item.fiyat * item.adet) * item.vergiler)
  dön vergi

Fonksiyon UygulaIndirim(kuponKodu):
  eğer KuponGeçerliMi(kuponKodu) ise
    promos = HesaplaKuponİndirimi(kuponKodu, SEPET)
    SEPET.promos.append(promos)
    dön true
  aksi halde dön false

/* Temel işlemler */
Fonksiyon SepeteEkle(product):
  mevcut = BulItem(product.productId)
  eğer mevcut != null ise
    yeniAdet = mevcut.adet + product.adet
    eğer yeniAdet > product.stok ise
      göster "Yeterli stok yok"
      mevcut.adet = product.stok
    başka
      mevcut.adet = yeniAdet
  başka
    eğer product.adet <= product.stok ise
      SEPET.items.append(product)
    başka
      product.adet = product.stok
      SEPET.items.append(product)

Fonksiyon SepettenÇıkar(productId):
  item = BulItem(productId)
  eğer item != null ise
    SEPET.items.remove(item)
  başka
    göster "Ürün sepette yok"

Fonksiyon AdetGüncelle(productId, yeniAdet):
  item = BulItem(productId)
  eğer item == null ise göster "Ürün bulunamadı" ve dön
  eğer yeniAdet <= 0 ise
    SepettenÇıkar(productId)
    dön
  eğer yeniAdet > item.stok ise
    item.adet = item.stok
    göster "İstenen adetten az stok var; adet stok seviyesine ayarlandı"
  başka
    item.adet = yeniAdet

Fonksiyon HesaplaToplam():
  araToplam = HesaplaAraToplam()
  vergi = HesaplaVergi()
  indirimToplam = 0
  için her promo in SEPET.promos:
    indirimToplam = indirimToplam + promo.tutar
  kargo = HesaplaKargo(araToplam)
  toplam = araToplam + vergi + kargo - indirimToplam
  eğer toplam < 0 ise toplam = 0
  dön { araToplam, vergi, kargo, indirimToplam, toplam }

Fonksiyon Ödemeİşle(ödemeBilgisi):
  ödemeyeHazır = DoğrulaAdresVeÖdeme(SEPET, ödemeBilgisi)
  eğer !ödemeyeHazır ise
    göster "Adres veya ödeme bilgileri eksik/yanlış"
    dön false
  result = ÖdemeGateway.Öde(ödemeBilgisi, HesaplaToplam().toplam)
  eğer result.başarılı ise
    StoğuGüncelle(SEPET.items)
    SiparişKaydet(SEPET, result)
    SEPET.items = []
    SEPET.promos = []
    göster "Ödeme başarılı, sipariş numarası: " + result.siparisNo
    dön true
  aksi halde
    göster "Ödeme başarısız: " + result.hata
    dön false

/* Ek fonksiyonlar (sistem entegrasyonu) */
Fonksiyon StoğuGüncelle(items):
  için her item in items:
    ÜrünServisi.AzaltStok(item.productId, item.adet)

Fonksiyon SiparişKaydet(sepet, ödemeSonucu):
  siparis = {
    userId: sepet.userId,
    items: sepet.items,
    toplam: HesaplaToplam().toplam,
    ödemeId: ödemeSonucu.id,
    tarih: Şimdi()
  }
  SiparişVeritabanı.Kaydet(siparis)

Fonksiyon KuponGeçerliMi(kod):
  // kupon doğrulama mantığı: tarih, kullanım sayısı, minimum sepet tutarı
  dön KuponServisi.Doğrula(kod, HesaplaAraToplam())

Fonksiyon HesaplaKargo(araToplam):
  eğer araToplam >= ÜcretsizKargoEşiği ise dön 0
  aksi halde dön KargoTablosundanTutar(araToplam)

