// Başlangıç: Sistem başlatma
INIT:
  yükle yapılandırma (sensör listesi, bildirim ayarları, bekleme süresi)
  sistem_aktif := true
  alarm_durum := "inaktif"    // "inaktif", "alarm", "bekle"
  alarm_seviyesi := 0         // 0 = yok, 1 = düşük, 2 = orta, 3 = yüksek
  sahip_evde_mi := false     // false: ev sahibi dışarda (varsayılan)

// Ana sonsuz döngü (Sürekli çalışan)
DO_WHILE (sistem_aktif == true):      // Ana DONG sürekli çalışır
  // 1. Tüm sensörleri oku
  sensör_verileri := oku_tüm_sensörler()    // hareket, kapı/pencere, cam kırılma, duman, vb.
  kamera_durum := kontrol_kameralar()       // canlı akış var/ yok

  // 2. Tehdit tespiti
  tehdit_algilandi := false
  if sensör_verileri.hareket_algılandı == true then
    if saat_gece_mi() veya güvenlik_modu == "dışarı" then
      tehdit_algilandi := true
    end if
  end if

  if sensör_verileri.kapi_acildi == true or sensör_verileri.pencere_acildi == true then
    tehdit_algilandi := true
  end if

  if sensör_verileri.cam_kirildi == true then
    tehdit_algilandi := true
  end if

  // 3. Kamera aktivasyonu (koşul)
  if tehdit_algilandi == true and kamera_durum == "kapalı" then
    kamera_aktif_et()            // kamera kaydını başlat, canlı görüntü al
  end if

  // 4. Yanlış alarm kontrolü — ev sahibi evde mi? (koşul)
  sahip_evde_mi := sorgula_ev_sahibi_var_mi()   // örn. geofencing, anahtar kart, manuel durum
  if tehdit_algilandi == true and sahip_evde_mi == true then
    // muhtemel yanlış alarm — ek doğrulama iste
    bekle(5 saniye)
    tekrar_veriler := oku_tüm_sensörler()
    if tekrar_veriler.hareket_algılandı == false and tekrar_veriler.kapi_acildi == false then
      // yanlış alarm tespit edildi — kaydet ve döngüye devam et
      log("Yanlış alarm - sahip evde")
      alarm_durum := "inaktif"
      alarm_seviyesi := 0
      goto WAIT_AND_LOOP
    end if
  end if

  // 5. Alarm seviyesi belirleme (koşul)
  if tehdit_algilandi == true then
    alarm_durum := "alarm"
    alarm_seviyesi := belirle_alarm_seviyesi(sensör_verileri, kamera_analizi)
    // örnek kural: tek hareket = 1, hareket + kapı = 2, cam kırılma veya çoklu sensör = 3
  else
    alarm_durum := "inaktif"
    alarm_seviyesi := 0
  end if

  // 6. Alarm ver ve bildirim gönder (SMS + App + Email)
  if alarm_durum == "alarm" then
    // yerel sirenleri tetikle (seviyeye göre farklı davranış)
    tetikle_siren(alarm_seviyesi)

    // bildirim içeriğini hazırla
    mesaj := "Alarm! Seviye: " + alarm_seviyesi + " Sensörler: " + sensör_özet(sensör_verileri)
    gönder_SMS(kayıtlı_sahip_numarasi, mesaj)
    gönder_App_Bildirim(kullanici_id, mesaj)
    gönder_Email(kullanici_email, "Ev Güvenlik Alarmı", mesaj)

    // kamera görüntüsünü sahibin uygulamasına veya güvenlik merkezine aktar
    upload_canli_goruntu(kamera_stream)

    // kaydet: olay logu
    log_olay(sensör_verileri, alarm_seviyesi, zaman_saat())
  end if

  // 7. Bekle ve tekrar kontrol et (döngü)
  WAIT_AND_LOOP:
  // Alarm sıfırlama komutu gelene kadar devam eder
  // Eğer alarm aktifse, periyodik tekrar kontrol ve bildirim (ör. her 15s'de bir)
  tekrar_sure := (alarm_durum == "alarm") ? kısa_tekrar_suresi : normal_okuma_suresi
  bekle(tekrar_sure)

  // 8. Alarm sıfırlama veya devam ettirme (koşul)
  if alarm_durum == "alarm" then
    if alınan_komut == "sıfırla" then
      alarm_durum := "inaktif"
      alarm_seviyesi := 0
      sonlandir_siren()
      gönder_App_Bildirim(kullanici_id, "Alarm sıfırlandı")
      log("Alarm sıfırlandı: kullanıcı talimatı")
    else
      // alarm devam ediyor, gerekirse ek eylemler (güvenlik servisi arama vb.)
      if alarm_seviyesi == 3 then
        arama_guvenlik_hizmeti(abonelik_bilgileri, sensör_verileri)
      end if
    end if
  end if

  // Döngü sonu — tekrar en başa dön (sonsuz döngü)
END_DO_WHILE

// Yardımcı fonksiyonlar (detaylandırılabilir)
function oku_tüm_sensörler() -> sensör_verileri
function kontrol_kameralar() -> kamera_durum
function belirle_alarm_seviyesi(veriler, kamera_analizi) -> int
function sorgula_ev_sahibi_var_mi() -> bool
function gönder_SMS(numara, mesaj)
function gönder_App_Bildirim(kullanici_id, mesaj)
function gönder_Email(mail, konu, mesaj)
function bekle(saniye)
