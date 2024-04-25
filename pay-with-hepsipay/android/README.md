# **Pay With HepsiPay Android Documentation**

# Contents
- [Integration](#integration)
- [Initialization](#initialization)
    - [PWHPConfig](#pwhpconfig)
    - [Container View](#containerview)
    - [Initialize](#initialize)
- [Usage](#usage)
- [UI Customization](#customization)
    - [Font](#font)
    - [Color](#color)

# <a name="integration"> Integration </a>
Root **`build.gradle`** dosyasına belirtilen Maven repository tanımlamalarını ekleyin:

```kotlin
allprojects {
    repositories {
        google()
        mavenCentral()
        maven(url = "https://images.hepsiburada.net/payment/assets/pwhp-native") {
            content {
                includeModule("com.hepsiburada.hepsipay", "paywithhp-ui")
                includeModule("com.hepsiburada.hepsipay", "paywithhp-data")
                includeModule("com.hepsiburada.hepsipay", "paywithhp-domain")
            }
        }
    }
}
```

Eğer projenizde **`settings.gradle`** kullanılıyorsa Maven repository tanımlamalarını belirtilen şekilde yapabilirsiniz:

```kotlin
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        maven(url = "https://images.hepsiburada.net/payment/assets/pwhp-native") {
            content {
                includeModule("com.hepsiburada.hepsipay", "paywithhp-ui")
                includeModule("com.hepsiburada.hepsipay", "paywithhp-data")
                includeModule("com.hepsiburada.hepsipay", "paywithhp-domain")
            }
        }
    }
}
```

İlgili modülün **`build.gradle`** dosyasına belirtilen implementasyonu ekleyin:

```kotlin
dependencies {
    implementation("com.hepsiburada.hepsipay:paywithhp-ui:0.0.3")
}
```

# <a name="initialization">Initialization</a>
### <a name="pwhpconfig">1. PWHPConfig</a>
Konfigürasyon objesini oluşturun.
```kotlin
val config = PWHPConfig(
    token = "MERCHANT_TOKEN",
    uniqueDeviceId = "UNIQUE_DEVICE_ID",
    environment = Environment.TEST | Environment.PROD
)

```
**`token: String`** değerini belirleyin. **(Required)**

**`uniqueDeviceId: String?`** benzersiz bir cihaz kimliği belirleyin. **(Optional)**

**`environment: Environment`** test veya production ortamını tanımlayın. **(Optional)** *(Default: Environment.TEST)*

### <a name="containerview">2. Container View</a>
SDK arayüzünün gösterileceği bir FragmentContainerView oluşturun:
```xml
<androidx.fragment.app.FragmentContainerView
         android:id="@+id/fl_container"
         android:layout_width="match_parent"
         android:layout_height="wrap_content" />
```
### <a name="initialize">3. Initialize</a>
`PWHPPresenterImpl` sınıfından bir obje oluşturun:
```kotlin
private val presenter = PWHPPresenterImpl()
```

`presenter` objesi üzerinden `init()` metodunu belirtildiği gibi çağırın:
```kotlin
presenter.init { builder ->
            builder.fragmentManager = childFragmentManager
            builder.pwhpConfig = config // 1. adımda oluşturuldu
            builder.containerViewId = R.id.fl_container // 2. adımda oluşturuldu
            builder.resultCallback = { result ->
                when (result) {
                    is PWHPResult.CompletePayment -> {
                        val callbackUrl = result.callbackUrl
                        val token = result.token
                        val orderNumber = result.orderNumber
                    }

                    is PWHPResult.PaymentAvailable -> {
                        // Arayüzde bulunan "Ödemeyi Tamamla" butonu
                        binding.btnMakePayment.isEnabled = result.isAvailable
                    }
                }
            }
        }
```

#### PWHPResult.CompletePayment
Ödeme işlemleri tamamlandığında tetiklenir. Result içerisinde `calllbackUrl`, `token` ve `orderNumber` döndürülür. Burada success ekranı gösterilebilir.
#### PWHPResult.PaymentAvailable
Mevcut durumda ödeme yapılıp yapılamayacağının bilgisini verir. Arayüzde **"Ödemeyi Tamamla"** gibi bir buton varsa **enabled/disabled** ayarı bu bilgiye göre yapılmalıdır.

# <a name="usage">Usage</a>
Ödeme işlemlerinin tamamlanması için `presenter` objesi üzerinden `makePayment()` metodunu belirtildiği gibi çağırın:
```kotlin
// Arayüzde bulunan "Ödemeyi Tamamla" butonu
binding.btnMakePayment.setOnClickListener {
    presenter.makePayment()
}
```

# <a name="customization">UI Customization</a>
UI customization işlemleri resources üzerinden yapılmaktadır. Projenizin `values` resource klasörünün içerisine yeni bir resource file oluşturun. Örneğin; `pwhp_resources.xml` 

[<img src="sample-pwhp-resources.png" width="250"/>](sample-pwhp-resources.png)

### <a name="font">1. Font</a>
Varsayılan olarak **Inter** font kullanılmaktadır. Custom font tanımlaması yapabilmek için `pwhp_resources.xml` içerisinde key-value tanımlamalarını yapın:

```xml
<resources>
    <!-- Fonts -->
    <font name="font_regular_pwhp">@font/quicksand_regular</font>
    <font name="font_medium_pwhp">@font/quicksand_medium</font>
    <font name="font_semi_bold_pwhp">@font/quicksand_semi_bold</font>
</resources>
```

### <a name="color">2. Color</a>

[<img src="pwhp-color-scheme.png" width="800"/>](pwhp-color-scheme.png)

Custom renk tanımlaması yapabilmek için `pwhp_resources.xml` içerisinde Color Scheme üzerinden karşılık gelen key'lerin değerlerinin değiştirilmesi gerekmektedir:

```xml
<resources>
    <!-- Colors -->
    <color name="color_primary_pwhp">@android:color/holo_red_dark</color>
</resources>
```
