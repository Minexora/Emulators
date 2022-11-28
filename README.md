# Emulatorler

Telefonunuzu bilgisayarınıza yansıtarak birçok işlemi yapabilmenize yarayan emülatörler, bu sayede telefonun birçok özelliğini de daha rahat kullanmanıza imkan tanıyor.

## Emulator oluşturma

SdkManager ve avdmanager kullanılarak emulator oluşturulmaktadır.Command line olarak kullanmak istediğimizde **Android Studio** adlı uygulamayı kurduktan sonra **/home/<username>/android_home_dir/cmdline-tools/latest/bin/sdkmanager** adresinde **sdkmanager** bulunmaktadır. Eğer yol yok ise **sdkmanager** kurulmamış demektir. Kurulumu için aşağıdaki komutları çalıştırılır.

```bash
sudo apt install default-jdk

# Sonrasında  https://developer.android.com/studio/#downloads adresinde aşağıda Command line tools only tablosunda size uygul olani indirip zip paketinden çıkarıyorsunuz. 

mkdir cmdline-tools
mv <folder-you-extracted> /home/<username>/cmdline-tools

export ANDROID_HOME=$HOME/cmdline-tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
source ~/.bashrc
```

Bu app kullanılarak ilk olarak yuklenebilecek sdkleri listelenir. Bunun için aşağıdaki kod çalıştırılır.

```bash
./sdkmanager --list
```

Listelenen sdklar içerisinde yüklenecek olani belirledikten sonra aşağıdaki kmut çalıştırılarak sdk yüklenir oluşturulur.

```bash
./sdkmanager --install system-images;<android-surumu(android-23)>;<google-eklentisi-durumu(google_apis)>;<mimari(x86_64)>
```

Sdk yükleme işlemi tamamlandıktan sonra emulator oluşturmak için aşağıdaki komut çalıştırılır.

```bash
echo "no" | ./avdmanager --verbose create avd --force --name <name("Nexus_6_6.0")> --device <telefon-gorunumu("pixel")> --package <sdk("system-images;android-23;google_apis;x86_64")> --tag <tag("google_apis")> --abi <mimari("x86_64")>
```

## Emulator konfigure etme

Emulator oluşturulduktan sonra **/home/<username>/.android/avd/<emulator-name>.avd/** yolunda bulunan **config.ini** dosyasında konfigure edilir. En çok kullanılan satırllar;

```ini
hw.cpu.ncore=2
hw.lcd.density=160
hw.lcd.height=640
hw.lcd.width=320
hw.ramSize=4096
sdcard.size=4096 MB
vm.heapSize=512M
```

Yukarıda gösterilen format bazen değişiklik göstermektedir.Ornegin hw.RamSize=96M olarak gelebiliyor ya da 1536M gibi.Sondaki M olmamali!!!. Yoksa default degerinde kalabiliyor. 96M ram olunca da haliyle emulator acilmiyor. Gpu destekli olmadigi icin de ekran cozunurlugunu dusuk verip density ile oranlamak daha verimli oluyor. Diger bazi parametreler de default halinde olsa dahi sorun cikartabiliyor. Ornek olarak menu tuslari olmamasi gereken emulatorde config'den acik ise arayuz bazen durdurabiliyor, stabillik dusuyor.


## Snapshot alma

Uygulamanın istenilen halini ram verileriyle birlikte yedeğinir almaya snapshat denir. Emülatöre snapshatlar yüklenilerek kaldığımız yerden devam edebilir.

```bash
adb emu avd snapshot save <snapshot_adı>
```

## Snapshot ile emulator başlatma

Alınan snapshotlar ile emulatorler başlatılabilir.Başlatıldığında snaphot nasıl alındıysa aynı şekilde kaldığı yerden devam edecektir.

```bash
<emulator-path>/emulator -avd <emulator-adı(generic_11.0)> -no-window -noaudio -snapshot <snapshot-adı>
```

## Emulator listesi

Açık olan emulatorları listelemek için aşağıdaki komut çalıştırılmalıdır.

```bash
adb devices
emulator -list-avds
```

## Emulator kapatmak

Açık olan bir emulatörü kapatmak için aşağıdaki komut çalıştırılmalıdır.

```bash
adb <emulator-adı> kill
```

## Emulator açıldıktan sonra işlem yapma

Emulator start dedikten sonra açılıncaya kadar bekleyip hemen işlem yapması için aşağıdaki komut çalıştırılmalıdır.

```bash
adb wait-for-decive shell <komut>
```

## Emulatore data göndermek

Emulatöre frida gbi data göndermek istiyor isek aşağıdaki gibi komut çalıştırmalıyız.

```bash
adb push <data-path> <emulator-path(/data/local/tmp)>
```

## Emulatore apk ya da ipa yüklemek

Emulatöre uygulama yüklemek için aşağıdaki komut çalıştırılmalıdır.

```bash
adb install <app-path>
```

## Emulatorden veri almak

Emulatorde bulunan bir dosyayı dışarı almak isteniyor ise aşağıdaki komut çalıştırılmaıdır.

```bash
adb pull <emulator-path(sdcard/log.txt)> <pc-path(/home/<username>/Desktop)>
```

## Emulatorde frida server başlatma

Frida server tersine mühendisler ve güvenlik için kullanılan araç setidir. Çalıştırmak için öncesinde dosyayı [bu linkten](https://github.com/frida/frida/releases) indirip sonrasında emulatorde
'/data/local/tmp' klasör altına atılmalıdır.Gerekli işlemler yapıldıktan sonra aşağıdaki komut çalıştırılmalıdır.

```bash
adb shell /data/local/tmp/<frida-server-name>
```

## Emulatorde çalışan uygulamanın pidsini alma

Emulatorde çalışan uygulamaların package name kullanılarak pidsini almak için aşağıdaki komut çalıştırılır.

```bash
adb shell pidof <package-name>
```

## Python ile frida hooks çalıştırma

Frida hook ile uygulamadaki fonksiyonlar, sınıflar dinlenileiliyor ve değiştirilebiliyor.Genelde root bypass işlemi için kullanılmaktadır. Pythonda kullanılabilmesi için;

```bash
pip install frida
```

yazılarak frida tool yüklenilmelidir. Sonrasında ise yazılan js hook metodları okunup ran ediliyor. Bu kodlar aşağıdaki gibidir.

```python
device = frida.get_usb_device()
pid_str=subprocess.check_output(["adb","shell","pidof","<package-name>"])
session = device.attach(int(pid_str))
with open("<js-hooks-file-path>") as f:
    script = session.create_script(f.read())
script.on("message", message_test_handler)
script.load()
```

## Appium kullanımı

Mobil test için kullanılan selenium tabanlı bir tooldur. Web driver protokolünü kullanmaktadır. Python eklettisini kurmak için aşağıdaki komut çalıştırılmalıdır.

```bash
pip install Appium-Python-Client
```

Tool kurulduktan sonra python scriptte kullanılması aşağıdaki gibidir.

```python
from appium import webdriver
from appium.webdriver.common.mobileby import MobileBy
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as ES
from selenium.webdriver.common.by import By
from appium.webdriver.common.touch_action import TouchAction

# Appium server bağlantısı kuruluyor.
driver = webdriver.Remote("http://localhost:4723/wd/hub", config['desired_caps'])
wait=WebDriverWait(driver,config['webdriver_timeout'])

# Inspectordan alınan idnin yüklemesini beklemek için kullanılıyor.
wait.until(ES.element_to_be_clickable((By.ID,"<inspector-kullanılarak-alınan-id>")))

# Inspectordan alınan idnin seçilmesi
el1 = driver.find_element_by_id("com.turkcell.bip:id/action_search")

# Seçilen elementin tıklanılması
el1.click()

# Seçilen elemente data gönderme
el2.send_keys(<önderilecek-data>)

# Kaydırma işlemi
subprocess.check_output(["adb","shell","input","swipe","160","580","160","150","150"])
```
