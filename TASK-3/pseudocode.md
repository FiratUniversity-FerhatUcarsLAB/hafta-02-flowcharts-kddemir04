BAŞLA

// 1. Kimlik Doğrulama
KULLANICIDAN TC_KIMLIK, SIFRE AL
EĞER (TC_KIMLIK ve SIFRE DOĞRUYSA)
    DEVAM ET
DEĞİLSE
    "Kimlik bilgileri hatalı!" YAZ
    BİTİR

// 2. Ana Menü
TEKRARLA
    "----- ANA MENÜ -----" YAZ
    "1 - Randevu Sistemi" YAZ
    "2 - Tahlil Sonucu Görüntüleme" YAZ
    "3 - Çıkış" YAZ
    KULLANICIDAN SECIM AL

    EĞER (SECIM == 1) İSE
        // --- RANDEVU SİSTEMİ ---
        POLIKLINIK_LISTESINI_GÖSTER()
        KULLANICIDAN POLIKLINIK_SECIMI AL

        DOKTOR_LISTESI = POLIKLINIGE_GÖRE_DOKTORLAR(POLIKLINIK_SECIMI)
        DOKTOR_LISTESINI_GÖSTER()
        KULLANICIDAN DOKTOR_SECIMI AL

        UYGUN_SAATLER = DOKTORUN_UYGUN_SAATLERI(DOKTOR_SECIMI)
        UYGUN_SAATLERI_GÖSTER()
        KULLANICIDAN SAAT_SECIMI AL

        EĞER SAAT_SECIMI UYGUNSA
            RANDEVU_OLUSTUR(TC_KIMLIK, DOKTOR_SECIMI, SAAT_SECIMI)
            "Randevunuz başarıyla oluşturuldu." YAZ
            SMS_GÖNDER(TC_KIMLIK, "Randevunuz onaylandı: " + SAAT_SECIMI)
        DEĞİLSE
            "Seçilen saat artık uygun değil!" YAZ

    EĞER (SECIM == 2) İSE
        // --- TAHLİL SONUCU GÖRÜNTÜLEME ---
        EĞER (TAHLIL_VAR_MI(TC_KIMLIK))
            EĞER (TAHLIL_HAZIR_MI(TC_KIMLIK))
                TAHLIL_SONUCU = TAHLIL_SONUCU_GETIR(TC_KIMLIK)
                TAHLIL_SONUCU_GÖSTER(TAHLIL_SONUCU)
                KULLANICIYA "Sonucu PDF olarak indirmek ister misiniz?" DİYE SOR
                EĞER (EVET)
                    PDF_OLUSTUR(TAHLIL_SONUCU)
                    "PDF indiriliyor..." YAZ
                DEĞİLSE
                    "İşlem sonlandırıldı." YAZ
            DEĞİLSE
                "Tahlil sonucunuz henüz hazır değil. Lütfen daha sonra tekrar deneyiniz." YAZ
        DEĞİLSE
            "Herhangi bir tahlil kaydınız bulunmamaktadır." YAZ

    EĞER (SECIM == 3) İSE
        "Sistemden çıkılıyor..." YAZ
        ÇIKIŞ_YAP()
        DÖNGÜDEN ÇIK

    DEĞİLSE
        "Geçersiz seçim! Lütfen 1, 2 veya 3 girin." YAZ

SONA KADAR TEKRARLA

BİTİR
