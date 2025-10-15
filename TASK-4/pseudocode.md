BAŞLA

// --- Veriler ---
kurslar <- ListeTümDönemKursları()   // her kurs: {kod, isim, kredi, kontenjan, dolu, önkosulListesi, zamanSlot}
öğrenciNo, şifre <- GİRİŞ_BİLGİLERİ_AL()
öğrenci <- KULLANICI_DOGRULA(öğrenciNo, şifre)
EĞER öğrenci == HATA İSE
    YAZ("Giriş başarısız. Tekrar deneyin.")
    DUR
SON

tamamlananDersler <- öğrenci.tamamlananDersler
kayıtlıDersler <- boş liste
toplamKredi <- 0
GPA <- öğrenci.GPA
danışmanOnayıGerekli <- FALSE

// --- Ders listesi göster ---
YAZ("Dönem dersleri:")
DÖNGÜ kurs içinde kurslar
    YAZ(kurs.kod, kurs.isim, "Kredi:", kurs.kredi, "Kontenjan:", kurs.kontenjan - kurs.dolu, "Önkoşul:", kurs.önkosulListesi, "Zaman:", kurs.zamanSlot)
SON

// --- Ders ekleme/çıkarma döngüsü ---
YAZ("Ders eklemek için 'E', ders çıkarmak için 'C', bitirmek için 'B' girin.")
TEKRAR
    işlem <- KULLANICIDAN_AL()
    EĞER işlem == 'E' İSE
        ekKursKod <- KULLANICIDAN_AL("Eklenecek ders kodunu girin:")
        kurs <- KURS_BUL(kurslar, ekKursKod)
        EĞER kurs == YOK İSE
            YAZ("Hatalı kurs kodu.")
            DEVAM_ET
        SON

        // 1) Kontenjan kontrolü
        EĞER kurs.dolu >= kurs.kontenjan İSE
            YAZ("Bu dersin kontenjanı dolu.")
            DEVAM_ET
        SON

        // 2) Önkoşul kontrolü
        eksikOnKosul <- FALSE
        DÖNGÜ okDers içinde kurs.önkosulListesi
            EĞER okDers NOT IN tamamlananDersler İSE
                eksikOnKosul <- TRUE
                KIR
            SON
        SON
        EĞER eksikOnKosul İSE
            YAZ("Bu dersin önkoşulları tamamlanmamış.")
            DEVAM_ET
        SON

        // 3) Zaman çakışması kontrolü
        çakışma <- FALSE
        DÖNGÜ r içinde kayıtlıDersler
            EĞER r.zamanSlot == kurs.zamanSlot İSE
                çakışma <- TRUE
                KIR
            SON
        SON
        EĞER çakışma İSE
            YAZ("Zaman çakışması tespit edildi.")
            DEVAM_ET
        SON

        // 4) Kredi limiti kontrolü
        EĞER toplamKredi + kurs.kredi > 35 İSE
            YAZ("Kredi limiti aşılıyor. (Toplam kredi: ", toplamKredi, ")")
            DEVAM_ET
        SON

        // Hepsi geçtiyse dersi ekle
        kayıtlıDersler.ekle(kurs)
        toplamKredi <- toplamKredi + kurs.kredi
        kurs.dolu <- kurs.dolu + 1
        YAZ("Ders eklendi:", kurs.kod, kurs.isim)
        DEVAM_ET

    EĞER işlem == 'C' İSE
        silKursKod <- KULLANICIDAN_AL("Silinecek ders kodunu girin:")
        kurs <- KURS_BUL(kayıtlıDersler, silKursKod)
        EĞER kurs == YOK İSE
            YAZ("Bu derse kayıtlı değilsiniz.")
            DEVAM_ET
        SON
        kayıtlıDersler.sil(kurs)
        toplamKredi <- toplamKredi - kurs.kredi
        // kontenjanı geri düşür
        gerçekKurs <- KURS_BUL(kurslar, silKursKod)
        gerçekKurs.dolu <- gerçekKurs.dolu - 1
        YAZ("Ders çıkarıldı:", kurs.kod, kurs.isim)
        DEVAM_ET

    EĞER işlem == 'B' İSE
        KIR
    SON

TEKRARSON

// --- Danışman onayı kontrolü ---
EĞER GPA < 2.5 İSE
    danışmanOnayıGerekli <- TRUE
    YAZ("GPA ", GPA, " — Danışman onayı gerekiyor.")
    onay <- DANIŞMANDAN_ONAY_AL(öğrenciNo, kayıtlıDersler)
    EĞER onay == RED İSE
        YAZ("Danışman onayı reddedildi. Kayıt tamamlanamadı.")
        DUR
    SON
SON

// --- Kayıt özeti göster ---
YAZ("Kayıt özeti:")
DÖNGÜ d içinde kayıtlıDersler
    YAZ(d.kod, d.isim, "Kredi:", d.kredi, "Zaman:", d.zamanSlot)
SON
YAZ("Toplam kredi:", toplamKredi)

YAZ("Kaydı onaylıyor musunuz? (E/H)")
cevap <- KULLANICIDAN_AL()
EĞER cevap == 'E' İSE
    KAYDI_SISTEME_KAYDET(öğrenciNo, kayıtlıDersler)
    YAZ("Kayıt başarıyla tamamlandı.")
    DUR
SENIHALINDE
    YAZ("Kayıt onaylanmadı. Değişiklik yapmak için tekrar giriş yapın.")
    DUR
SON

BİTİR
