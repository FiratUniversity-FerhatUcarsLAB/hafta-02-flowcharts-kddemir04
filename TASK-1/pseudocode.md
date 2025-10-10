BAŞLA

    # Başlangıç değişkenleri
    TANIMLA kartTakildi = DOĞRU
    TANIMLA hesapBakiyesi = 3500.00        # örnek bakiye (TL)
    TANIMLA gunlukLimit = 1000.00         # günlük çekilebilecek toplam limit (TL)
    TANIMLA gunlukCekilen = 0.00          # o gün içinde çekilen miktar (TL)
    TANIMLA PIN_DOGRU = "1234"            # gerçek sistemde bu banka sunucusunda olur
    TANIMLA PIN_DENEME_SINIRI = 3

    # Kart takılı ise devam et, değilse işlemi sonlandır
    EĞER-İSE kartTakildi = HATALI
        YAZ "Lütfen kartınızı takınız."
        SON
    GERİSE
        YAZ "Kart okundu. Lütfen PIN giriniz."

    # PIN doğrulama - 3 hak
    TANIMLA denemeSayisi = 0
    TANIMLA pinBasarili = YANLIŞ

    DÖNGÜ (denemeSayisi < PIN_DENEME_SINIRI) YAP
        YAZ "PIN kodunu giriniz veya 'iptal' yazınız:"
        OKU girilenPIN

        EĞER-İSE girilenPIN = "iptal"
            YAZ "İşlem iptal edildi. Kart iade ediliyor."
            kartTakildi = HATALI
            SON

        EĞER-İSE girilenPIN = PIN_DOGRU
            pinBasarili = DOĞRU
            YAZ "PIN doğrulandı."
            DUR
        GERİSE
            denemeSayisi = denemeSayisi + 1
            KALAN = PIN_DENEME_SINIRI - denemeSayisi
            EĞER-İSE KALAN > 0
                YAZ "Yanlış PIN. Kalan deneme hakkı: " + KALAN
            GERİSE
                YAZ "PIN denemeleri tükendi. Kart bloke edildi ve iade ediliyor."
                kartTakildi = HATALI
                SON
            GERİSE
        SON
    SON_DÖNGÜ

    EĞER-İSE pinBasarili = YANLIŞ
        YAZ "PIN doğrulanamadı. İşlem sonlandırılıyor."
        SON
    GERİSE
        # Ana işlem menüsüne geç

    DÖNGÜ (TRUE) YAP   # Kullanıcı işlem tekrarı için ana döngü
        YAZ ""
        YAZ "Lütfen yapmak istediğiniz işlemi seçiniz:"
        YAZ "1 - Bakiye Sorgulama"
        YAZ "2 - Para Çekme"
        YAZ "3 - Çıkış"
        OKU secim

        EĞER-İSE secim = "1"    # Bakiye sorgulama
            YAZ "Mevcut bakiyeniz: " + hesapBakiyesi + " TL"
            YAZ "Bugün çektiğiniz toplam: " + gunlukCekilen + " TL (Günlük limit: " + gunlukLimit + " TL)"
        GERİSE EĞER-İSE secim = "2"    # Para çekme
            # Miktar okuma ve kontroller
            YAZ "Çekmek istediğiniz tutarı giriniz (20 TL katları). İptal için 'iptal' yazın:"
            OKU tutarGirdi

            EĞER-İSE tutarGirdi = "iptal"
                YAZ "İşlem iptal edildi. Ana menüye dönülüyor."
                DEVAM_ET
            GERİSE
                # Girilen değerin sayısal olup olmadığını kontrol et (basit doğrulama)
                EĞER-İSE TutarGirisiNumerikMi(tutarGirdi) = YANLIŞ
                    YAZ "Geçersiz tutar. Lütfen sayısal bir değer giriniz."
                    DEVAM_ET
                GERİSE
                    TANIMLA tutar = SAYIYA_DONUSTUR(tutarGirdi)

                    # Tutarın pozitif olup olmadığı kontrolü
                    EĞER-İSE tutar <= 0
                        YAZ "Çekilecek tutar pozitif olmalıdır."
                        DEVAM_ET
                    GERİSE
                        # 20 TL katı kontrolü
                        EĞER-İSE (tutar % 20) != 0
                            YAZ "Tutar 20 TL'nin katı olmalıdır. Lütfen 20, 40, 60 ... gibi bir tutar giriniz."
                            DEVAM_ET
                        GERİSE
                            # Bakiye kontrolü
                            EĞER-İSE tutar > hesapBakiyesi
                                YAZ "Yetersiz bakiye. Mevcut bakiye: " + hesapBakiyesi + " TL"
                                DEVAM_ET
                            GERİSE
                                # Günlük limit kontrolü
                                EĞER-İSE (gunlukCekilen + tutar) > gunlukLimit
                                    TANIMLA kalanLimit = gunlukLimit - gunlukCekilen
                                    EĞER-İSE kalanLimit <= 0
                                        YAZ "Günlük limitiniz dolmuştur. Başka işlem yapamazsınız."
                                        DEVAM_ET
                                    GERİSE
                                        YAZ "Bu işlem günlük limitinizi aşar. Kalan günlük çekilebilir miktar: " + kalanLimit + " TL"
                                        DEVAM_ET
                                    SON
                                GERİSE
                                    # Tüm kontroller geçti - işlem onayı
                                    YAZ "Lütfen işlemi onaylayınız: 'evet' veya 'hayır'"
                                    OKU onay
                                    EĞER-İSE onay = "evet"
                                        # Para verme ve güncelleme
                                        YAZ "Lütfen nakitinizi alınız. " + tutar + " TL verildi."
                                        hesapBakiyesi = hesapBakiyesi - tutar
                                        gunlukCekilen = gunlukCekilen + tutar
                                        YAZ "İşlem başarılı. Kalan bakiye: " + hesapBakiyesi + " TL"
                                        YAZ "Bugün çekilen toplam: " + gunlukCekilen + " TL"
                                        # Fis ister mi?
                                        YAZ "Fiş ister misiniz? (evet/hayır)"
                                        OKU fisSecim
                                        EĞER-İSE fisSecim = "evet"
                                            YAZ "Fiş basılıyor..."
                                        GERİSE
                                            YAZ "Fiş verilmedi."
                                        SON
                                    GERİSE
                                        YAZ "İşlem iptal edildi. Ana menüye dönülüyor."
                                    SON
                                SON
                            SON
                        SON
                    SON
                SON
        GERİSE EĞER-İSE secim = "3"
            YAZ "Çıkış seçildi. Kart iade ediliyor. Hoşçakalın."
            kartTakildi = HATALI
            SON
        GERİSE
            YAZ "Geçersiz seçim. Lütfen 1, 2 veya 3 giriniz."
        SON

        # İşlem tekrarı sorusu (ana döngü devam/bitir)
        YAZ "Yeni bir işlem yapmak ister misiniz? (evet/hayır)"
        OKU tekrar
        EĞER-İSE tekrar = "evet"
            YAZ "Ana menüye dönülüyor..."
            DEVAM_ET   # döngü devam eder
        GERİSE
            YAZ "İşlem sonlandırılıyor. Kart iade ediliyor. Güle güle."
            kartTakildi = HATALI
            SON
        SON
    SON_DÖNGÜ

SON  # program sonu


# Yardımcı fonksiyon (kullanım amaçlı, dil bağımsız)
FONKSİYON TutarGirisiNumerikMi(girdi)
    DENEME: sayi = SAYIYA_DONUSTUR(girdi)  # hata olursa FALSE döndür
    EĞER-İSE dönüştürme başarılıysa
        DÖN TRUE
    GERİSE
        DÖN FALSE
    SON
SONFONKSİYON
