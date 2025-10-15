Başla

// Sistem sürekli çalışacak
while (true) {
    
    // Sistem aktif mi?
    eğer (SistemAktif == true) {

        // Sensör okuma döngüsü (sürekli)
        hareketAlgilandi = false
        kapiPencereAlgilandi = false

        // Hareket sensörünü kontrol et
        eğer (HareketSensoruAktif()) {
            hareketAlgilandi = true
        }

        // Kapı/pencere sensörünü kontrol et
        eğer (KapiPencereSensoruAktif()) {
            kapiPencereAlgilandi = true
        }

        // Kamera aktivasyonu (opsiyonel, hareket veya kapı/pencere açıldığında)
        eğer (hareketAlgilandi veya kapiPencereAlgilandi) {
            KameraAktifEt()
        }

        // Yanlış alarm kontrolü (ev sahibi evde mi?)
        eğer (EvSahibiEvde == true) {
            AlarmVerme = false
        } aksi halde {
            AlarmVerme = hareketAlgilandi veya kapiPencereAlgilandi
        }

        // Alarm seviyesi belirleme
        eğer (AlarmVerme) {
            eğer (hareketAlgilandi ve kapiPencereAlgilandi) {
                AlarmSeviyesi = 3 // Yüksek
            } aksi halde eğer (hareketAlgilandi veya kapiPencereAlgilandi) {
                AlarmSeviyesi = 2 // Orta
            } aksi halde {
                AlarmSeviyesi = 1 // Düşük
            }

            // Alarmı tetikle
            AlarmCal(AlarmSeviyesi)

            // Bildirim gönder
            GonderBildirim("SMS", AlarmSeviyesi)
            GonderBildirim("App", AlarmSeviyesi)
            GonderBildirim("Email", AlarmSeviyesi)
        }

        // Bekle ve tekrar kontrol et
        Bekle(1 saniye) // veya uygun süre

        // Alarm sıfırlama veya devam ettirme kontrolü
        eğer (EvSahibiEvde veya AlarmDurumuGecersiz) {
            AlarmSifirla()
        } aksi halde {
            DevamEt()
        }
    } 
    aksi halde {
        Bekle(5 saniye) // Sistem kapalıysa aralıklarla kontrol
    }
}
