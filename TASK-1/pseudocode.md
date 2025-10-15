PROGRAM ATM_Para_Cekme

YAPILAR:
    HESAP
        hesapNo
        sifre
        bakiye
        hesapKilitliMi
        gunlukCekilenTutar
        gunlukLimit
    SON

    ATM
        kasa        // örnek: 200, 100, 50, 20, 10 TL’lik banknot sayıları
    SON

BAŞLA
    kart = kartOku()
    hesap = hesapBilgisiGetir(kart)

    EĞER hesap == YOKSA
        EKRANA_YAZ("Geçersiz kart. Lütfen bankanızla iletişime geçiniz.")
        DUR
    SON

    EĞER hesap.hesapKilitliMi == TRUE
        EKRANA_YAZ("Hesabınız kilitli. Bankanızla iletişime geçiniz.")
        DUR
    SON

    DENE
        PIN = KULLANICIDAN_AL("Lütfen şifrenizi giriniz: ")
    SON

    EĞER PIN ≠ hesap.sifre İSE
        EKRANA_YAZ("Hatalı şifre! Tekrar deneyiniz.")
        hataliDenemeSayisi = hataliDenemeSayisi + 1
        EĞER hataliDenemeSayisi ≥ 3 İSE
            hesap.hesapKilitliMi = TRUE
            EKRANA_YAZ("3 kez yanlış şifre girildi. Hesap kilitlendi.")
            DUR
        SON
        TEKRAR_BAŞA_DÖN
    SON

    // Şifre doğruysa para çekim işlemi başlar
    miktar = KULLANICIDAN_AL("Çekmek istediğiniz tutarı giriniz: ")

    EĞER miktar ≤ 0 İSE
        EKRANA_YAZ("Geçersiz tutar.")
        DUR
    SON

    EĞER miktar % 10 ≠ 0 İSE
        EKRANA_YAZ("Tutar 10 TL’nin katı olmalıdır.")
        DUR
    SON

    EĞER miktar > hesap.bakiye İSE
        EKRANA_YAZ("Yetersiz bakiye.")
        DUR
    SON

    EĞER hesap.gunlukCekilenTutar + miktar > hesap.gunlukLimit İSE
        EKRANA_YAZ("Günlük para çekme limitinizi aştınız.")
        DUR
    SON

    EĞER ATMdeYeterliNakitVarMi(miktar) == FALSE
        EKRANA_YAZ("ATM'de yeterli nakit bulunmamaktadır.")
        DUR
    SON

    // İşlem onaylanıyor
    ONAY = KULLANICIDAN_AL("İşlemi onaylıyor musunuz? (E/H): ")

    EĞER ONAY == 'H' İSE
        EKRANA_YAZ("İşlem iptal edildi.")
        DUR
    SON

    // Para çekimi işlemi
    hesap.bakiye = hesap.bakiye - miktar
    hesap.gunlukCekilenTutar = hesap.gunlukCekilenTutar + miktar
    ATMdenParaVer(miktar)

    EKRANA_YAZ("İşlem başarılı! Lütfen paranızı alınız.")
    EKRANA_YAZ("Kalan bakiye: " + hesap.bakiye)

    makbuz = KULLANICIDAN_AL("Makbuz ister misiniz? (E/H): ")
    EĞER makbuz == 'E' İSE
        makbuzYazdir(hesap, miktar)
    SON

    EKRANA_YAZ("Teşekkür ederiz. İyi günler dileriz.")
SON
