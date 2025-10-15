ALGORİTMA HastaneYonetimSistemi

BAŞLA

    // Ana menü
    YAZ "HASTANE YÖNETİM SİSTEMİNE HOŞGELDİNİZ"
    YAZ "1 - Randevu Sistemi"
    YAZ "2 - Tahlil Sonucu Görüntüleme Sistemi"
    YAZ "3 - Çıkış"
    YAZ "Seçiminizi yapınız: "
    SECIM ← GİRİŞ()

    EĞER SECIM = 1 İSE
        HastaneRandevuSistemi()
    DEĞİLSE EĞER SECIM = 2 İSE
        TahlilSonucuGoruntulemeSistemi()
    DEĞİLSE EĞER SECIM = 3 İSE
        YAZ "Sistemden çıkılıyor..."
        ÇIKIŞ YAP
    DEĞİLSE
        YAZ "Geçersiz seçim! Lütfen tekrar deneyiniz."
    SON

BİTİR


// -----------------------------
// 1. MODÜL: HASTANE RANDEVU SİSTEMİ
// -----------------------------
PROSEDÜR HastaneRandevuSistemi()

    YAZ "T.C. kimlik numaranızı giriniz: "
    TC ← GİRİŞ()
    YAZ "Şifrenizi giriniz: "
    SIFRE ← GİRİŞ()

    EĞER KimlikDogrula(TC, SIFRE) = DOĞRU İSE
        YAZ "Kimlik doğrulama başarılı."
    DEĞİLSE
        YAZ "Kimlik doğrulama başarısız! Tekrar deneyiniz."
        GERİ DÖN
    SON

    YAZ "Lütfen poliklinik seçiniz: "
    POLIKLINIK_LISTESI ← PoliklinikleriGetir()
    LISTEYI_GOSTER(POLIKLINIK_LISTESI)
    SECILEN_POLIKLINIK ← GİRİŞ()

    DOKTOR_LISTESI ← DoktorlariGetir(SECILEN_POLIKLINIK)
    YAZ "Bu poliklinikteki doktorlar:"
    LISTEYI_GOSTER(DOKTOR_LISTESI)
    SECILEN_DOKTOR ← GİRİŞ()

    UYGUN_SAATLER ← UygunSaatleriGetir(SECILEN_DOKTOR)
    YAZ "Uygun randevu saatleri:"
    LISTEYI_GOSTER(UYGUN_SAATLER)
    SECILEN_SAAT ← GİRİŞ()

    YAZ "Randevunuzu onaylamak istiyor musunuz? (E/H)"
    CEVAP ← GİRİŞ()

    EĞER CEVAP = "E" İSE
        RandevuKaydet(TC, SECILEN_DOKTOR, SECILEN_SAAT)
        YAZ "Randevunuz başarıyla oluşturuldu."
        SMSGonder(TC, "Randevunuz onaylandı: " + SECILEN_DOKTOR + " - " + SECILEN_SAAT)
        YAZ "SMS bilgilendirmesi gönderildi."
    DEĞİLSE
        YAZ "Randevu işlemi iptal edildi."
    SON

SON


// -----------------------------
// 2. MODÜL: TAHLİL SONUCU GÖRÜNTÜLEME SİSTEMİ
// -----------------------------
PROSEDÜR TahlilSonucuGoruntulemeSistemi()

    YAZ "T.C. kimlik numaranızı giriniz: "
    TC ← GİRİŞ()
    YAZ "Şifrenizi giriniz: "
    SIFRE ← GİRİŞ()

    EĞER KimlikDogrula(TC, SIFRE) = DOĞRU İSE
        YAZ "Kimlik doğrulama başarılı."
    DEĞİLSE
        YAZ "Kimlik doğrulama başarısız! Tekrar deneyiniz."
        GERİ DÖN
    SON

    EĞER TahlilVarMi(TC) = YANLIŞ İSE
        YAZ "Sistemde kayıtlı tahlil bulunmamaktadır."
        GERİ DÖN
    SON

    EĞER SonucHazirMi(TC) = DOĞRU İSE
        YAZ "Tahlil sonuçlarınız hazır."
        SonucuGoster(TC)

        YAZ "Sonucu PDF olarak indirmek ister misiniz? (E/H)"
        CEVAP ← GİRİŞ()

        EĞER CEVAP = "E" İSE
            PDFIndir(TC)
            YAZ "Tahlil sonucu PDF olarak indirildi."
        DEĞİLSE
            YAZ "PDF indirme işlemi iptal edildi."
        SON
    DEĞİLSE
        YAZ "Tahlil sonuçlarınız henüz hazır değil. Lütfen daha sonra tekrar deneyiniz."
    SON

SON


// -----------------------------
// ORTAK FONKSİYONLAR
// -----------------------------
FONKSİYON KimlikDogrula(TC, SIFRE)
    // Gerçek sistemde veri tabanı kontrolü yapılır
    EĞER TC VE SIFRE DOĞRU İSE GERİ DÖN DOĞRU
    DEĞİLSE GERİ DÖN YANLIŞ
SON
