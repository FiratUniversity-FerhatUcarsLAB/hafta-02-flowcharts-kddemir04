BAŞLA

    // --- 1. GİRİŞ EKRANI ---
    YAZ "Ders Kayıt Sistemine Hoşgeldiniz"
    YAZ "Öğrenci numaranızı giriniz: "
    OKU ogrenciNo
    YAZ "Şifrenizi giriniz: "
    OKU sifre

    EĞER kimlikDogrula(ogrenciNo, sifre) = DOĞRU İSE
        YAZ "Giriş başarılı."
    DEĞİLSE
        YAZ "Hatalı giriş. Program sonlandırılıyor."
        BİTİR
    SON

    // --- 2. DERS LİSTESİNİ GÖSTER ---
    YAZ "Alınabilecek dersler:"
    DÖNGÜ i = 1'dan dersiSayısına kadar
        YAZ dersListesi[i].ad, " (Kredi:", dersListesi[i].kredi, 
            ", Kontenjan:", dersListesi[i].kontenjan, 
            ", Ön koşul:", dersListesi[i].onKosul, 
            ", Gün/Saat:", dersListesi[i].zaman, ")"
    SON

    // --- 3. DERS SEÇİMİ ---
    seçilenDersler ← boş liste
    toplamKredi ← 0

    DÖNGÜ
        YAZ "Ders kodunu giriniz (veya 0 ile bitir): "
        OKU dersKod

        EĞER dersKod = 0 İSE
            ÇIK DÖNGÜDEN
        SON

        ders ← dersBul(dersKod)

        EĞER ders = YOKSA
            YAZ "Geçersiz ders kodu!"
        DEĞİLSE
            // --- 4. KONTROLLER ---
            
            // 4.1. Kontenjan kontrolü
            EĞER ders.kontenjan = 0 İSE
                YAZ "Bu dersin kontenjanı dolu!"
            DEĞİLSE
                // 4.2. Ön koşul kontrolü
                EĞER ders.onKosul ≠ YOKSA
                    EĞER onKosulSaglandiMi(ogrenciNo, ders.onKosul) = YANLIŞ İSE
                        YAZ "Ön koşul dersi tamamlanmamış: ", ders.onKosul
                    DEĞİLSE
                        // 4.3. Zaman çakışması kontrolü
                        zamanCakisti ← YANLIŞ
                        DÖNGÜ her secilen IN seçilenDersler
                            EĞER ders.zaman = secilen.zaman İSE
                                zamanCakisti ← DOĞRU
                            SON
                        SON

                        EĞER zamanCakisti = DOĞRU İSE
                            YAZ "Bu ders, seçilen başka bir dersle çakışıyor!"
                        DEĞİLSE
                            // 4.4. Kredi limiti kontrolü
                            EĞER toplamKredi + ders.kredi > ogrenciKrediLimiti(ogrenciNo) İSE
                                YAZ "Kredi limitiniz aşılıyor!"
                            DEĞİLSE
                                // 4.5. Ders zaten seçilmiş mi kontrolü
                                EĞER ders IN seçilenDersler İSE
                                    YAZ "Bu dersi zaten seçtiniz!"
                                DEĞİLSE
                                    // TÜM KONTROLLER TAMAM
                                    seçilenDersler’e ekle(ders)
                                    toplamKredi ← toplamKredi + ders.kredi
                                    YAZ ders.ad, " dersi başarıyla eklendi."
                                SON
                            SON
                        SON
                    SON
                DEĞİLSE
                    // Ön koşul yoksa direkt ekleme kontrolü
                    EĞER toplamKredi + ders.kredi <= ogrenciKrediLimiti(ogrenciNo) İSE
                        seçilenDersler’e ekle(ders)
                        toplamKredi ← toplamKredi + ders.kredi
                        YAZ ders.ad, " dersi başarıyla eklendi."
                    DEĞİLSE
                        YAZ "Kredi limitiniz aşılıyor!"
                    SON
                SON
            SON
        SON
    SON DÖNGÜ

    // --- 5. ONAYLAMA AŞAMASI ---
    EĞER seçilenDersler BOŞSA
        YAZ "Hiç ders seçilmedi. Kayıt tamamlanamadı."
        BİTİR
    SON

    YAZ "Seçilen dersler:"
    DÖNGÜ her ders IN seçilenDersler
        YAZ "-", ders.ad, " (", ders.kredi, " kredi)"
    SON
    YAZ "Toplam kredi:", toplamKredi

    YAZ "Danışman onayına gönderilsin mi? (E/H)"
    OKU cevap

    EĞER cevap = 'E' İSE
        EĞER danismanOnayi(ogrenciNo, seçilenDersler) = DOĞRU İSE
            YAZ "Ders kayıt işleminiz tamamlandı ✅"
        DEĞİLSE
            YAZ "Danışman onayı reddedildi ❌"
        SON
    DEĞİLSE
        YAZ "Ders seçimi iptal edildi."
    SON

BİTİR
