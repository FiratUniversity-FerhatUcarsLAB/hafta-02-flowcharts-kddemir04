BAŞLA
    // 1. KULLANICI GİRİŞİ
    EKRANA "Giriş yap veya kayıt ol" YAZ
    KULLANICI_SEÇİMİ ← GİRİŞ_YAP veya KAYIT_OL seçimi al
    
    EĞER KULLANICI_SEÇİMİ = KAYIT_OL İSE
        EKRANA "Email ve şifre giriniz" YAZ
        EMAIL, SIFRE ← KULLANICIDAN AL
        YENİ_KULLANICI_OLUŞTUR(EMAIL, SIFRE)
        EKRANA "Kayıt başarılı" YAZ
    DEĞİLSE EĞER KULLANICI_SEÇİMİ = GİRİŞ_YAP İSE
        EKRANA "Email ve şifre giriniz" YAZ
        EMAIL, SIFRE ← KULLANICIDAN AL
        EĞER KULLANICI_DOĞRU(EMAIL, SIFRE) İSE
            EKRANA "Giriş başarılı" YAZ
        DEĞİLSE
            EKRANA "Hatalı giriş" YAZ
            BİTİR
        SON
    SON

    // 2. ÜRÜN SEÇME VE SEPETE EKLEME
    SEPET ← BOŞ
    DÖNGÜ ÜRÜN_SEÇİMİ = KULLANICI "Ürün ID gir (0 çıkış)" DEYİNCEYE KADAR
        EĞER ÜRÜN_SEÇİMİ = 0 İSE
            DÖNGÜDEN ÇIK
        DEĞİLSE
            MİKTAR ← KULLANICIDAN "Kaç adet?" AL
            EĞER STOK_KONTROL(ÜRÜN_SEÇİMİ) ≥ MİKTAR İSE
                SEPETE_EKLE(SEPET, ÜRÜN_SEÇİMİ, MİKTAR)
                EKRANA "Ürün sepete eklendi" YAZ
            DEĞİLSE
                EKRANA "Yetersiz stok!" YAZ
            SON
        SON
    DÖNGÜ SONU

    // 3. SEPET GÖRÜNTÜLEME VE GÜNCELLEME
    EKRANA "Sepetiniz:" YAZ
    SEPETİ_GÖSTER(SEPET)
    
    EKRANA "Sepeti düzenlemek ister misiniz? (E/H)" YAZ
    CEVAP ← KULLANICIDAN AL
    EĞER CEVAP = "E" İSE
        DÖNGÜ GÜNCELLEME = KULLANICI "Ürün ID gir (0 çıkış)" DEYİNCEYE KADAR
            EĞER GÜNCELLEME = 0 İSE
                DÖNGÜDEN ÇIK
            DEĞİLSE
                YENİ_MİKTAR ← KULLANICIDAN "Yeni miktar?" AL
                EĞER STOK_KONTROL(GÜNCELLEME) ≥ YENİ_MİKTAR İSE
                    SEPET_GÜNCELLE(SEPET, GÜNCELLEME, YENİ_MİKTAR)
                    EKRANA "Sepet güncellendi" YAZ
                DEĞİLSE
                    EKRANA "Stok yetersiz" YAZ
                SON
            SON
        DÖNGÜ SONU
    SON

    // 4. İNDİRİM KODU UYGULAMA
    EKRANA "İndirim kodunuz var mı? (E/H)" YAZ
    CEVAP ← KULLANICIDAN AL
    EĞER CEVAP = "E" İSE
        KOD ← KULLANICIDAN "Kodu giriniz" AL
        EĞER KUPON_GEÇERLİ(KOD) İSE
            İNDİRİM_ORANI ← KUPON_DEĞERİ(KOD)
            SEPET.TOPLAM ← SEPET.TOPLAM - (SEPET.TOPLAM * İNDİRİM_ORANI)
            EKRANA "İndirim uygulandı" YAZ
        DEĞİLSE
            EKRANA "Geçersiz kod" YAZ
        SON
    SON

    // 5. KARGO HESAPLAMA
    EKRANA "Adresinizi giriniz:" YAZ
    ADRES ← KULLANICIDAN AL
    AĞIRLIK ← SEPET_AĞIRLIĞI_HESAPLA(SEPET)
    KARGO_FİYATI ← KARGO_HESAPLA(ADRES, AĞIRLIK)
    EKRANA "Kargo ücreti: " + KARGO_FİYATI YAZ

    // 6. SİPARİŞ ÖZETİ
    TOPLAM ← SEPET.TOPLAM + KARGO_FİYATI
    EKRANA "Ara toplam: " + SEPET.TOPLAM YAZ
    EKRANA "Kargo: " + KARGO_FİYATI YAZ
    EKRANA "GENEL TOPLAM: " + TOPLAM YAZ

    // 7. ÖDEME AŞAMASI
    EKRANA "Ödeme yöntemini seçiniz: (1-Kredi Kartı, 2-Kapıda Ödeme)" YAZ
    ÖDEME_TÜRÜ ← KULLANICIDAN AL

    EĞER ÖDEME_TÜRÜ = 1 İSE
        KART_BİLGİSİ ← KULLANICIDAN "Kart numarası, SKT, CVV" AL
        EĞER ÖDEME_ONAY(KART_BİLGİSİ, TOPLAM) = BAŞARILI İSE
            EKRANA "Ödeme başarılı!" YAZ
            STOK_GÜNCELLE(SEPET)
            SİPARİŞ_OLUŞTUR(SEPET, ADRES, TOPLAM, "ÖDENDİ")
        DEĞİLSE
            EKRANA "Ödeme başarısız, lütfen tekrar deneyin" YAZ
            BİTİR
        SON
    DEĞİLSE EĞER ÖDEME_TÜRÜ = 2 İSE
        EKRANA "Kapıda ödeme seçildi" YAZ
        SİPARİŞ_OLUŞTUR(SEPET, ADRES, TOPLAM, "KAPIDA_ÖDEME")
    SON

    // 8. SİPARİŞ SONUCU
    EKRANA "Siparişiniz oluşturuldu. Teşekkür ederiz!" YAZ
BİTİR
