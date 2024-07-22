

# rtl8192eu Linux Sürücüleri

Bu sürücü, [CGarces/rtl8192eu-linux-driver](https://github.com/CGarces/rtl8192eu-linux-driver) deposundan, Türk kullanıcılar için daha anlaşılır hale getirilerek forklanmıştır herhangi bir kötü amaç kopyalama kötüleme gibi bir amaç bulunmamaktadır.

- Aşağıda Kurulumla İlgili Gerekli Adımlar Sadeleştirilmiş ve türkçe diline cevrilmiştir



Bu dosya, Realtek RTL8192EU WiFi adaptörünüz için gerekli olan Linux sürücüsünü nasıl kuracağınızı anlatıyor. Bu sürücü, bilgisayarınıza bu WiFi adaptörünü tanıtmak için gereklidir.

**NOT:** Bu sürücü, Realtek’in eski sürüm 4.4.1’e dayanıyor. Başka bir sürümle aynı olmayabilir.

Bu sürücü, D-Link DWA-131 Rev E ve bazı diğer WiFi adaptörleriyle çalışır. Eğer bu cihazları kullanıyorsanız, bu sürücü işinize yarayabilir.

## Resmi Sürücüler Nerede?

Resmi sürücüler, D-Link’in Avustralya sitesinden indirilmiştir. Avustralya tüm versiyonları sunarken, diğer bölgelerde bazı versiyonlar eksik olabilir.

* [DWA-131 İndir](http://support.dlink.com.au/Download/download.aspx?product=DWA-131)
* [Sürücü İndirme Bağlantısı](ftp://files.dlink.com.au/products/DWA-131/REV_E/Drivers/DWA-131_Linux_driver_v4.3.1.1.zip)

## Yamalar

Sürücüye eklenen yamalar ve yapılan değişiklikler hakkında daha fazla bilgi için bu depoyu inceleyebilirsiniz. Kodun her bir değişikliği, belirli bir değişiklik numarası (SHA) ile birlikte gelir, bu sayede yapılan değişiklikleri kolayca takip edebilirsiniz.

## DKMS ile Sürücü Kurulumu

Bu adımlar, sürücüyü bilgisayarınıza kurmak içindir:

1. **Gerekli Araçları Yükleyin:**

   Bilgisayarınızda gerekli araçları yükleyin:

   * Normal Linux bilgisayarlar için:

     ```shell
     sudo apt-get install git linux-headers-generic build-essential dkms
     ```

   * Raspberry Pi için:

     ```shell
     sudo apt-get install git raspberrypi-kernel-headers build-essential dkms
     ```

2. **Sürücüyü İndirin ve Açın:**

   Sürücü dosyasını bilgisayarınıza indirin ve açın:

   ```shell
   git clone https://github.com/abdullah-eksi/realtek_wifi_adapter_driver
   cd rtl8192eu-linux-driver
   ```

3. **Platform Seçimini Yapın:**

   Derleme için platform seçimini yapın. (Çoğu bilgisayarda bu adımı atlayabilirsiniz. Eğer Raspberry Pi veya başka bir cihaz kullanıyorsanız, uygun seçeneği seçmelisiniz.)

4. **Sürücüyü DKMS’ye Ekleyin:**

   Sürücüyü sisteminize ekleyin:

   ```shell
   sudo dkms add .
   ```

5. **Sürücüyü Kurun:**

   Sürücüyü derleyip kurun:

   ```shell
   sudo dkms install rtl8192eu/1.0
   ```

6. **Varsayılan Sürücüyü Kara Listeye Alın:**

   RTL8XXXU sürücüsünü kapatmalısınız:

   ```shell
   echo "blacklist rtl8xxxu" | sudo tee /etc/modprobe.d/rtl8xxxu.conf
   ```

7. **Sürücüyü Aktif Hale Getirin:**

   Yeni sürücüyü sistem başlangıcında etkinleştirin:

   ```shell
   echo -e "8192eu\n\nloop" | sudo tee /etc/modules
   ```

8. **Güncellemeleri Yapın ve Yeniden Başlatın:**

   Güncellemeleri yapın ve bilgisayarınızı yeniden başlatın:

   ```shell
   sudo update-grub; sudo update-initramfs -u
   systemctl reboot -i
   ```

9. **Sürücünün Yüklenip Yüklenmediğini Kontrol Edin:**

   Doğru sürücünün yüklendiğinden emin olun:

   ```shell
   sudo lshw -c network
   ```

   Eğer doğru sürücü yüklenmişse, `driver=8192eu` şeklinde bir bilgi göreceksiniz.

## Sürücüyü Kaldırma

Sürücüyü kaldırmak isterseniz:

   ```shell
   sudo dkms uninstall rtl8192eu/1.0
   ```

   Tamamen kaldırmak için:

   ```shell
   sudo dkms remove rtl8192eu/1.0 --all
   ```



## Lisans ve Telif Hakları

Orijinal kodun telif hakkı mevcut, ama kimin tarafından olduğu bilinmiyor. Kod GNU Genel Kamu Lisansı (GPL) altında lisanslanmıştır.

---
