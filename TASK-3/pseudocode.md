BAŞLA

    HASTALAR ← boş liste
    DOKTORLAR ← boş liste
    RANDEVULAR ← boş liste

    FONKSİYON hasta_kayıt()
        hasta_ad ← GİRİŞ("Hasta adı giriniz: ")
        hasta_tc ← GİRİŞ("TC kimlik numarası giriniz: ")
        HASTALAR’e (hasta_ad, hasta_tc) ekle
    SON

    FONKSİYON doktor_kayıt()
        doktor_ad ← GİRİŞ("Doktor adı giriniz: ")
        branş ← GİRİŞ("Doktor branşı giriniz: ")
        DOKTORLAR’a (doktor_ad, branş) ekle
    SON

    FONKSİYON randevu_al()
        hasta_tc ← GİRİŞ("Randevu alacak hastanın TC’si: ")
        doktor_ad ← GİRİŞ("Randevu alınacak doktor adı: ")
        tarih ← GİRİŞ("Randevu tarihi (GG/AA/YYYY): ")
        saat ← GİRİŞ("Randevu saati (SS:DD): ")
        RANDEVULAR’a (hasta_tc, doktor_ad, tarih, saat) ekle
        YAZ("Randevu başarıyla oluşturuldu.")
    SON

    FONKSİYON randevu_listele()
        HER randevu İÇİN RANDEVULAR’da
            YAZ("Hasta TC:", randevu.hasta_tc, "Doktor:", randevu.doktor_ad, 
                "Tarih:", randevu.tarih, "Saat:", randevu.saat)
        SON
    SON

    TEKRAR
        YAZ("1. Hasta Kayıt")
        YAZ("2. Doktor Kayıt")
        YAZ("3. Randevu Al")
        YAZ("4. Randevuları Listele")
        YAZ("5. Çıkış")

        seçim ← GİRİŞ("Seçiminizi yapınız: ")

        EĞER seçim = 1 İSE
            hasta_kayıt()
        DEĞİLSE EĞER seçim = 2 İSE
            doktor_kayıt()
        DEĞİLSE EĞER seçim = 3 İSE
            randevu_al()
        DEĞİLSE EĞER seçim = 4 İSE
            randevu_listele()
        DEĞİLSE EĞER seçim = 5 İSE
            ÇIKIŞ YAP
        DEĞİLSE
            YAZ("Geçersiz seçim!")
        SON
    SONTEKRAR

BİTİR
