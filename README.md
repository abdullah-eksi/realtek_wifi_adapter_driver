# rtl8192eu Linux Sürücüleri

**NOT:** Bu dal, Realtek'in 4.4.1 sürümüne dayanmaktadır. `master` dalı ise başlangıçta 4.3.1.1 sürümüne dayanmaktadır.

Bu resmi sürücüler, D-Link DWA-131 Rev E için güncellenmiş yamalarla birlikte sunulmaktadır. Ayrıca Rosewill RNX-N180UBE v2 N300 Kablosuz Adaptör ve TP-Link TL-WN821N V6 ile de uyumludur.

**NOT:** Bu yalnızca bir "ayna"dır. Bu kod hakkında bilgi sahibi değilim ve nasıl çalıştığını bilmiyorum. Burada benden veya herhangi bir katkı sağlayandan destek beklemeyin. GitHub'ın, forum gönderilerinden ve e-posta ile gönderilen önceden derlenmiş ikili dosyalardan daha iyi bir takip yöntemi olduğunu düşünüyorum. Başka birinin, çalışana kadar rastgele yamalarla derlemek için 5 gün harcamasını istemiyorum.

## Resmi Sürücülerin Kaynağı

Resmi sürücüler, D-Link Avustralya'dan indirildi. D-Link ABD ve Avrupa ülkelerinde yalnızca A ve B revizyonları listelenmektedir. Avustralya ise tüm üç revizyonu listelemektedir.

* [DWA-131 için indirme sayfası][driver-downloads]
* [Linux sürücüleri için doğrudan indirme bağlantısı][direct-download]
  * GitHub, `ftp://` şemasına bağlantı vermeyecek. Ham bağlantı içeriği:

      `ftp://files.dlink.com.au/products/DWA-131/REV_E/Drivers/DWA-131_Linux_driver_v4.3.1.1.zip`

Ek olarak, bu sürümün içeriğini bu deponun ilk commit'inde bulabilirsiniz: [1387cf623d54bc2caec533e72ee18ef3b6a1db29][initial-commit]

## Yamalar

Uygulanan yamaları, kaynaklarını ve motivasyonlarını commit'lere bakarak görebilirsiniz. `master` dalı, çoğunlukla her yama için tek bir commit ile temiz tutulacaktır. Commit'leri inceleyip SHA'yı kaydedebilir ve kullanmak için güvenli bir referans alabilirsiniz. SHA aynı kaldığı sürece, ne elde ettiğinizin sizin tarafınızdan incelendiğini bilirsiniz.

Bu README dosyasındaki güncellemeler ayrı commit'ler olarak görünecektir. Bu dosyadaki değişiklikleri, koddaki değişikliklerle karıştırmayacağım, böylece README'yi olmadan aynalamak isterseniz kolayca yapabilirsiniz.

## DKMS ile Derleme ve Kurulum

Bu ağaç, dinamik çekirdek modül desteğini (DKMS) destekler. Çekirdek modüllerini çekirdek kaynaklarından oluşturmaya yarayan bir sistemdir. Çekirdek modüllerini kurmak/kaldırmak için kullanılabilir ve çekirdek yükseltildiğinde (örneğin paket yöneticinizi kullanarak) modül otomatik olarak yeniden derlenir.

1. DKMS ve diğer gerekli araçları yükleyin.

    * Normal Linux sistemleri için

    ```shell
    sudo apt-get install git linux-headers-generic build-essential dkms
    ```

    * Raspberry Pi için

    ```shell
    sudo apt-get install git raspberrypi-kernel-headers build-essential dkms
    ```

    Aynı başlık sürümünü şu an çalışan çekirdeğinizle yüklediğinizden emin olun. Yeni bir Raspbian kurduysanız, çekirdeğinizle uyumsuz eski bir başlık sürümüyle gelir. `sudo apt-get upgrade` komutunu çalıştırmalı veya `raspberrypi-kernel-headers-XXX` sürümünü çekirdeğinizle uyumlu olacak şekilde yüklemelisiniz. Sürüm uyuşmazlığı varsa "Çekirdek başlıklarınız çekirdek XXX için YYY'de bulunamıyor" hatası alırsınız.

2. Bu depoyu klonlayın ve klonlanan yola gidin.

    ```shell
    git clone https://github.com/Mange/rtl8192eu-linux-driver
    cd rtl8192eu-linux-driver
    ```

3. Makefile, çoğu x86/PC sürümü için önceden yapılandırılmıştır. Ancak, intel x86 mimarisinden başka bir şey için derliyorsanız önce platformu seçmelisiniz.

    * Raspberry Pi için I386'yi n ve ARM_RPI'yi y olarak ayarlamalısınız:

    ```sh
    ...
    CONFIG_PLATFORM_I386_PC = n
    ...
    CONFIG_PLATFORM_ARM_RPI = y
    ```

    * arm64 cihazlar için (örn. Orange Pi PC 2):

    ```sh
    ...
    CONFIG_PLATFORM_I386_PC = n
    ...
    CONFIG_PLATFORM_ARM_AARCH64 = y
    ```

4. Sürücüyü DKMS'ye ekleyin. Bu, kaynakları sistem dizinine kopyalayacak ve çekirdek yükseltmelerinde modülün yeniden derlenmesini sağlayacaktır.

    ```shell
    sudo dkms add .
    ```

5. Sürücüyü derleyin ve kurun.

    ```shell
    sudo dkms install rtl8192eu/1.0
    ```

6. Debian ve Ubuntu tabanlı dağıtımlarda RTL8XXXU sürücüsü çekirdek alanında mevcuttur ve çalışır durumda. RTL8192EU sürücümüzü kullanmak için RTL8XXXU'yu kara listeye almalıyız.

    ```shell
    echo "blacklist rtl8xxxu" | sudo tee /etc/modprobe.d/rtl8xxxu.conf
    ```

7. RTL8192EU sürücüsünü boot sırasında etkinleştirin.

    ```shell
    echo -e "8192eu\n\nloop" | sudo tee /etc/modules
    ```

8. Yeni Ubuntu sürümlerinde takma/yeniden takma sorunu var (Bkz #94). Bu, garip bekleme sorunlarını içerir. Bunu düzeltmek için:

    ```shell
    echo "options 8192eu rtw_power_mgnt=0 rtw_enusbss=0" | sudo tee /etc/modprobe.d/8192eu.conf
    ```

9. Grub ve initramfs değişikliklerini güncelleyin.

    ```shell
    sudo update-grub; sudo update-initramfs -u
    ```

10. Yeni oluşturulan initramfs'den yeni değişiklikleri yüklemek için sistemi yeniden başlatın.

    ```shell
    systemctl reboot -i
    ```

11. Çekirdeğinizin doğru modülü yüklediğini kontrol edin:

    ```shell
    sudo lshw -c network
    ```

    Şu satırı görmelisiniz: `driver=8192eu`

Sürücüyü daha sonra kaldırmak isterseniz, `sudo dkms uninstall rtl8192eu/1.0` komutunu kullanın. Sürücüyü DKMS'den tamamen kaldırmak için `sudo dkms remove rtl8192eu/1.0 --all` komutunu kullanın.

## Erişim Noktası (AP) Olarak Kullanma

Referans: Intelbras IWA 3001 USB WiFi Adaptörü  
8192eu yongasını kullanan cihazlar, yaklaşık ~50Mbps hızlarında erişim noktaları olarak hizmet verebilir.  

hostapd kullanarak AP'nizi yönetin ve bu cihaz için uygun ht-capab alanını ayarlayın:

`HT_CAPAB=[RX-STBC1][SHORT-GI-40][SHORT-GI-20][DSSS_CCK-40][MAX-AMSDU-7935]`

Geniş bantı etkinleştirmek için, komşularınız yoksa:  
Bu, ağın verimini artıracaktır, ancak uzak mesafedeki müşterilerin bağlantı kuramamasına neden olabilir.  
Ayrıca, cihazın sinyalini tekrarlayan tekrarlayıcılarla daha iyi çalışmasını sağlayabilir.

`HT_CAPAB=[HT40+][RX-STBC1][SHORT-GI-40][SHORT-GI-20][DSSS_CCK-40][MAX-AMSDU-7935]` (kanallar 1-7 için) veya  
`HT_CAPAB=[HT40-][RX-STBC1][SHORT-GI-40][SHORT-GI-20][DSSS_CCK-40][MAX-AMSDU-7935]` (kanallar 5-13 için)

## Transmit Gücünü Değiştirme

Şu anda, diğer kablosuz cihazlarda olduğu gibi iw veya iwconfig araçları ile sürücüde transmit gücünü değiştirme yolu yoktur.  
Bu sürücüde dönen değerler tamamen kurgusaldır. Ancak, hala transmit gücünü manuel olarak derleme zamanında değiştirebilirsiniz, bunun için `hal/rl8192e/rtl8192e_phycfg.c` dosyasını düzenleyip aşağıdaki satırları değiştirebilirsiniz:

```
/* Manuel Transmit Güç Kontrolü 
   Aşağıdaki seçenekler 0 ile 63 arasında değer alır, burada:
   0 - devre dışı
   1 - cihazın yapabileceği en düşük

 transmit gücü
   63 - cihazın yapabileceği en yüksek transmit gücü
   Bu seçeneklerin, ülkenizin transmit gücü ile ilgili düzenlemelerini geçersiz kılabileceğini unutmayın.
   Cihazı çoğu zaman daha yüksek transmit güçlerinde çalıştırmak, aşırı ısınma nedeniyle erken arızalanmaya veya hasara neden olabilir. Transmit gücünü artırmadan önce cihazın yeterli hava akışına sahip olduğundan emin olun.
   Bu değerlerin dBm olarak ne anlama geldiği şu anda bilinmiyor.
*/


// Transmit Güç Artırımı
// Bu değer, cihazın transmit güç endeksi hesaplamalarına eklenir.
// Güç kullanımını düşük tutarken transmit gücünü artırmak/azaltmak için kullanışlıdır.
// Gücü azaltmak için negatif bir değer de alabilir.
// Sıfır devre dışı bırakır. Varsayılan: 2, küçük bir artış için.
int transmit_power_boost = 2;
// (İLERİ DÜZEY) Bu cihazın dinamik olarak kullanmaya karar verdiği transmit güçlerini öğrenmek için bkz:
// https://github.com/lwfinger/rtl8192ee/blob/42ad92dcc71cb15a62f8c39e50debe3a28566b5f/hal/phydm/rtl8192e/halhwimg8192e_rf.c#L1310


// Transmit Güç Aşımı
// Bu değer, sürücünün hesaplamalarını tamamen geçersiz kılar ve tüm iletimler için tek bir değeri kullanır.
// Sıfır devre dışı bırakır. Varsayılan: 0
int transmit_power_override = 0;


/* Manuel Transmit Güç Kontrolü */
```

## Yama Gönderme

1. Depoyu fork edin
2. Yamanızı bir konu dalında yapın
3. GH'de bir pull request açın veya e-posta ile `Magnus Bergmark <magnus.bergmark@gmail.com>` adresine gönderin.
4. Her şey kontrol edildiğinde commit'lerinizi birleştirip `master` dalına ekleyeceğim.

## Telif Hakları ve Lisanslar

Orijinal kod telif hakkına sahiptir, ancak kimin tarafından olduğu bilinmiyor. Sürücü indirmesi lisans bilgisi içermiyor; lütfen telif hakkı sahibiyseniz bir sorun açın.

Çoğu C dosyası, GNU Genel Kamu Lisansı (GPL) sürüm 2 altında lisanslanmıştır.

[driver-downloads]: http://support.dlink.com.au/Download/download.aspx?product=DWA-131
[direct-download]: ftp://files.dlink.com.au/products/DWA-131/REV_E/Drivers/DWA-131_Linux_driver_v4.3.1.1.zip
[initial-commit]: https://github.com/Mange/rtl8192eu-linux-driver/commit/1387cf623d54bc2caec533e72ee18ef3b6a1db29
