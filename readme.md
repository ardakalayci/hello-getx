# Bağımlılık Yönetimi 
- [Bağımlılık Yönetimi](#dependency-management) 
  - [Örnekleme yöntemleri](#instancing-methods) 
    - [Get.put()](#getput) 
    - [Get.lazyPut](#getlazyput) 
    - [Get .putAsync](#getputasync) 
    - [Get.create](#getcreate) 
  - [Örneklenmiş yöntemleri/sınıfları kullanma](#using-instantiated-methodsclasses) 
  - [Alternatif bir örnek belirtme](#specifying-an-alternate-instance) 
  - [Yöntemler arasındaki farklar](#differences-between-methods) 
  - [Bindings](#bindings) 
    - [Bindings sınıfı](#bindings-class) 
    - [BindingsBuilder](#bindingsbuilder) 
    - [SmartManagement](#smartmanagement) 
      - [Nasıl değiştirilir](#nasıl değiştirilir)
      - [SmartManagement.full](#smartmanagementfull) 
      - [SmartManagement.onlyBuilder](#smartmanagementonlybuilder) 
      - [SmartManagement.keepFactory](#smartmanagementkeepfactory) 
    - [bağlamalar başlık altında nasıl çalışır](#how-bindings- work- -hood) 
  - [Notlar](#notes) Get'in 

basit ve güçlü bir bağımlılık yöneticisi vardır ve Blok veya Denetleyicinizle aynı sınıfı yalnızca 1 satır kodla, Sağlayıcı bağlamı olmadan, devralınanWidget olmadan almanızı sağlar: 

```dart
Controller controller = Get.put(Controller()); // Rather Controller controller = Controller();
```

Sınıfınızı kullandığınız sınıf içinde somutlaştırmak yerine, onu Uygulamanız genelinde kullanılabilir hale getirecek olan Get örneğinde somutlaştırıyorsunuz. 
Böylece denetleyicinizi (veya Bloc sınıfını) normal şekilde kullanabilirsiniz 

- Not: Get's State Manager kullanıyorsanız, görünümünüzü denetleyicinize bağlamayı kolaylaştıracak olan [Bindings](#bindings) API'sine daha fazla dikkat edin. 
- Not²: Al bağımlılık yönetimi paketin diğer bölümlerinden ayrılmıştır, bu nedenle örneğin uygulamanız zaten bir durum yöneticisi kullanıyorsa (herhangi biri, önemli değil), bunu değiştirmeniz gerekmez, kullanabilirsiniz bu bağımlılık ekleme yöneticisi hiçbir sorun yaşamaz 

## Örnekleme yöntemleri 
Yöntemler ve yapılandırılabilir parametreleri şunlardır: 

### Get.put()

Bağımlılık eklemenin en yaygın yolu. Örneğin, görüşlerinizi kontrol edenler için iyi. 

```dart
Get.put<SomeClass>(SomeClass());
Get.put<LoginController>(LoginController(), permanent: true);
Get.put<ListItemController>(ListItemController, tag: "some unique string");
```

Put kullanırken ayarlayabileceğiniz tüm seçenekler bunlardır: 
```dart 
Get.put<S>( 
  // zorunlu: bir denetleyici veya herhangi bir şey gibi kaydetmek istediğiniz sınıf 
  // not: "S ", herhangi bir türden bir sınıf olabileceği anlamına gelir 
  S dependency

  // isteğe bağlı: bu, aynı türden birden çok sınıf istediğinizde içindir 
  // çünkü normalde Get.find kullanarak bir sınıf alırsınız<
  // hangi örneğe ihtiyacınız olduğunu söylemek için etiketi kullanmanız gerekir 
  // benzersiz dize olmalıdır 
  String tag,

  // isteğe bağlı: varsayılan olarak get, örnekleri artık kullanılmadıktan sonra imha eder (örnek, 
  // bir görünümün denetleyicisi kapalıdır), ancak örneğin, tüm uygulama boyunca orada tutulmasına ihtiyacınız olabilir, 
  // paylaşılanPreferences örneği veya başka bir şey gibi 
  // böylece bunu kullanırsınız 
  // varsayılanları false 
  bool permanent = false,

  // isteğe bağlı: izin verir Bir testte soyut bir sınıf kullandıktan sonra, onu başka bir sınıfla değiştirin ve testi takip edin. 
  // varsayılan olarak false 
  bool overrideAbstract = false,

  // isteğe bağlı: bağımlılığın kendisi yerine işlevi kullanarak bağımlılığı oluşturmanıza olanak tanır. 
  // bu yaygın olarak kullanılmaz 
  InstanceBuilderCallback<S> builder,
) 
``` 

### Get.lazyPut 
Yalnızca kullanıldığında somutlaştırılabilmesi için lazyLoad bağımlılığını yüklemek mümkündür. Hesaplamalı pahalı sınıflar için veya birkaç sınıfı tek bir yerde başlatmak istiyorsanız (Bindings sınıfında olduğu gibi) çok kullanışlıdır ve o zaman o sınıfı kullanmayacağınızı bilirsiniz. 

```dart 
/// ApiMock yalnızca birisi Get.find<ApiMock> öğesini ilk kez kullandığında çağrılır 
Get.lazyPut<ApiMock>(() => ApiMock());

Get.lazyPut<FirebaseAuth>(
  () {
    // ... some logic if needed
    return FirebaseAuth();
  },
  tag: Math.random().toString(),
  fenix: true
)

Get.lazyPut<Controller>( () => Controller() ) 
``` 

Bu, lazyPut kullanırken ayarlayabileceğiniz tüm seçeneklerdir: 
``` dart 
Get.lazyPut<S>( 
  // zorunlu: sınıfınız ilk kez çağrıldığında yürütülecek bir yöntem 
  InstanceBuilderCallback builder,
  
  // isteğe bağlı: Get.put() ile aynı, birden çok istediğinizde kullanılır aynı sınıfın farklı bir örneği 
  // benzersiz olmalıdır
  String tag,

  // isteğe bağlı: "Kalıcı" ile benzerdir, fark şu ki, örneğin şu durumlarda atılır:
  // kullanılmıyor, ancak tekrar kullanılması gerektiğinde, Get örneği yeniden oluşturacak 
  // api bağlamalarındaki "SmartManagement.keepFactory" ile aynı 
  // varsayılan olarak false 
  bool fenix = false 
  
) 
``` 

### Get.putAsync 
Eşzamansız bir örnek kaydetmek istiyorsanız, `Get.putAsync` kullanabilirsiniz: 

```dart
Get.putAsync<SharedPreferences>(() async {
  final prefs = await SharedPreferences.getInstance();
  await prefs.setInt('counter', 12345);
  return prefs;
});

Get.putAsync<YourAsyncClass>( () async => await YourAsyncClass() )
```

PutAsync kullanırken ayarlayabileceğiniz seçeneklerin tümü budur: 
```dart
Get.putAsync<S>( 
  AsyncInstanceBuilderCallback<S> builder,
  // isteğe bağlı: Get.put() ile aynı, birden çok farklı örnek istediğinizde kullanılır aynı sınıfın 
  // benzersiz olmalıdır
  String tag,
  // isteğe bağlı: Get.put() ile aynı, bu örneği tüm uygulamada canlı tutmanız gerektiğinde kullanılır 
  // varsayılan olarak false 
  bool permanent = false
) 
`` ` 
### Get.create 

This one is tricky. A detailed explanation of what this is and the differences between the other one can be found on [Differences between methods:](#differences-between-methods) section

```dart
Get.Create<SomeClass>(() => SomeClass());
Get.Create<LoginController>(() => LoginController());
```

Create kullanırken ayarlayabileceğiniz seçeneklerin tümü budur: 

```dart 
Get.create<S>( 
  // gerekli: her // zaman "fabrikasyon" olacak bir sınıf döndüren bir işlev `Get.find( )` adı 
  // Örnek: Get.create<YourClass>(() => YourClass()) 
  FcBuilderFunc<S> builder,

  // isteğe bağlı: Get.put() gibi, ancak birden çok örneğe ihtiyacınız olduğunda kullanılır 
  // aynı sınıftan 
  // Her öğenin kendi denetleyicisine ihtiyaç duyduğu bir listeniz varsa kullanışlıdır 
  // benzersiz bir dize olması gerekir. Sadece etiketten isme değiştirin 
  String name,

  // isteğe bağlı: tıpkı int`Get.put()` gibi , tüm uygulamada 
  // örneğini canlı 
  tutmanız gerektiğinde kullanılır . Fark Get.create'dedir 
  // kalıcı, varsayılan olarak true 
  bool permanent = true
``` 

## Örneklenmiş yöntemleri/sınıfları kullanma 

Çok sayıda rotada gezindiğinizi ve denetleyicinizde geride bırakılmış bir veriye ihtiyacınız olduğunu hayal edin, Sağlayıcı veya Get_it ile birleştirilmiş bir durum yöneticisine ihtiyacınız olacak, doğru mu? Get ile değil. Denetleyiciniz için Get to "bul" komutunu istemeniz yeterlidir, ek bağımlılığa ihtiyacınız yoktur: 

```dart 
final controller = Get.find<Controller>(); 
// VEYA 
Controller controller = Get.find();

// Evet, Magic'e benziyor, Get kontrol cihazınızı bulacak ve size teslim edecek. 
// 1 milyon denetleyiciyi somutlaştırabilirsiniz, Get her zaman size doğru denetleyiciyi verecektir. 
``` 

Ve sonra oradan elde ettiğiniz kontrolör verilerinizi kurtarabileceksiniz: 

```dart 
Text(controller.textFromApi); 
``` 

Döndürülen değer normal bir sınıf olduğundan, istediğiniz her şeyi yapabilirsiniz: 
```dart 
int count = Get.find<SharedPreferences>().getInt('counter'); 
yazdır(sayım); // çıktı: 12345 
``` 

Get örneğini kaldırmak için: 

```dart 
Get.delete<Controller>(); //Genellikle bunu yapmanız gerekmez çünkü GetX zaten kullanılmayan denetleyicileri siler
``` 

## Alternatif bir örnek belirtme 

Halihazırda eklenmiş bir örnek, "replace" veya "lazyReplace" yöntemi kullanılarak benzer veya genişletilmiş bir sınıf örneğiyle değiştirilebilir. Bu daha sonra orijinal sınıf kullanılarak alınabilir. 

```dart
abstract class BaseClass {}
class ParentClass extends BaseClass {}

class ChildClass extends ParentClass {
  bool isChild = true;
}


Get.put<BaseClass>(ParentClass());

Get.replace<BaseClass>(ChildClass());

final instance = Get.find<BaseClass>();
print(instance is ChildClass); //true


class OtherClass extends BaseClass {}
Get.lazyReplace<BaseClass>(() => OtherClass());

final instance = Get.find<BaseClass>();
print(instance is ChildClass); // false
print(instance is OtherClass); //true
```

## Yöntemler arasındaki farklar 

Önce Get.lazyPut'un 'fenix'i ve diğer yöntemlerin 'kalıcı'larından bahsedelim. 

"Kalıcı" ve "fenix" arasındaki temel fark, örneklerinizi nasıl depolamak istediğinizdir. 

Güçlendirme: Varsayılan olarak GetX, kullanımda değilken örnekleri siler. 
Bunun anlamı: Ekran 1'de denetleyici 1 varsa ve ekran 2'de denetleyici 2 varsa ve ilk rotayı yığından kaldırırsanız (gibi 'Get.off()' veya 'Get.offNamed()' kullanıyorsanız) denetleyici 1 kaybolur kullanımı silinecektir.

Ancak "kalıcı:doğru" kullanmayı tercih etmek istiyorsanız, bu geçişte denetleyici kaybolmaz - bu, tüm uygulama boyunca canlı tutmak istediğiniz hizmetler için çok yararlıdır. 

'fenix' ise ekran değişiklikleri arasında kaybetme endişesi duymadığınız ancak o hizmete ihtiyaç duyduğunuzda canlı olmasını beklediğiniz hizmetler içindir. Temel olarak, kullanılmayan denetleyiciyi/hizmeti/sınıfı elden çıkaracak, ancak ihtiyacınız olduğunda yeni bir örneği "küllerden yeniden yaratacaktır". 

Yöntemler arasındaki farklarla devam etmek:

- Get.put ve Get.putAsync, ikincisinin eşzamansız bir yöntem kullanması farkıyla aynı oluşturma sırasını takip eder: bu iki yöntem örneği oluşturur ve başlatır. Bu, "permanent: false" ve "isSingleton: true" parametreleriyle "insert" dahili yöntemi kullanılarak doğrudan belleğe eklenir (bu isSingleton parametresinin tek amacı, "bağımlılık" bağımlılığını kullanıp kullanmayacağını söylemektir. veya `FcBuilderFunc` bağımlılığını kullanacaksa). Bundan sonra, bellekteki örnekleri hemen başlatan `Get.find()` çağrılır.

- Get.create: Adından da anlaşılacağı gibi, bağımlılığınızı "yaratacak"! "Get.put()"a benzer şekilde, örneklemeye "insert" dahili yöntemini de çağırır. Ancak 'kalıcı' doğru oldu ve 'isSingleton' yanlış oldu (bağımlılığımızı "yarattığımızdan", bunun tek bir örnek olmasının bir yolu yok, bu yüzden yanlış). Ve "kalıcı: doğru" olduğu için, varsayılan olarak ekranlar arasında kaybetmeme avantajına sahibiz! Ayrıca `Get.find()` hemen çağrılmaz, çağrılacak ekranda kullanılmayı bekler. "Permanent" parametresini kullanmak için bu şekilde yaratılmıştır, o zamandan beri, fark edilmeye değer, "Get.create()", örneğin bir bu liste için benzersiz bir örnek istediğinizi bir listView'deki düğme - bu nedenle, Get.

- Get.lazyPut: Adından da anlaşılacağı gibi tembel bir işlemdir. Örnek yaratılır, ancak hemen kullanılmak üzere çağrılmaz, çağrılmayı bekler. Diğer yöntemlerin aksine burada `insert` denilmez. Bunun yerine, instance, hafızanın başka bir bölümüne, örneğin yeniden oluşturulup oluşturulamayacağını söylemekle sorumlu bir kısma eklenir, buna "fabrika" diyelim. Daha sonra kullanılmak üzere bir şey yaratmak istersek, şu anda kullanılanlarla karıştırılmayacak. Ve işte burada 'fenix' büyüsü devreye giriyor: 'fenix: false' bırakmayı seçerseniz ve 'smartManagement'ınız 'keepFactory' değilse, o zaman 'Get.find' kullanılırken örnek bellekteki yeri değiştirecektir. "fabrika"dan ortak örnek bellek alanına. Bundan hemen sonra, varsayılan olarak "fabrikadan" kaldırılır. Şimdi, `fenix'i seçerseniz:

## Bağlamlar 

Bu paketin en büyük farklılıklarından biri, belki de, yolların, durum yöneticisinin ve bağımlılık yöneticisinin tam entegrasyonu olasılığıdır. 
Yığından bir rota kaldırıldığında, onunla ilgili tüm denetleyiciler, değişkenler ve nesne örnekleri bellekten kaldırılır. Akışlar veya zamanlayıcılar kullanıyorsanız, bunlar otomatik olarak kapatılır ve bunların hiçbiri için endişelenmenize gerek yoktur. 
2.10 sürümünde Bindings API'sini tamamen uygulayın. 
Artık init yöntemini kullanmanıza gerek yok. İstemiyorsanız kontrol cihazlarınızı yazmanız bile gerekmez. Bunun için uygun yerde kontrolörlerinizi ve servislerinizi başlatabilirsiniz.
Binding sınıfı, durum yöneticisine ve bağımlılık yöneticisine giden yolları "bağlarken", bağımlılık enjeksiyonunu ayıracak bir sınıftır. 
Bu, belirli bir denetleyici kullanıldığında hangi ekranın görüntülenmekte olduğunu ve bunun nerede ve nasıl imha edileceğini bilmenizi sağlar. 
Ayrıca Binding sınıfı, SmartManager yapılandırma kontrolüne sahip olmanızı sağlar. Yığından bir rota kaldırılırken veya onu kullanan pencere öğesi düzenlendiğinde veya hiçbirini yapmadığında düzenlenecek bağımlılıkları yapılandırabilirsiniz. Sizin için çalışan akıllı bağımlılık yönetimine sahip olacaksınız, ancak buna rağmen istediğiniz gibi yapılandırabilirsiniz. 

### Bindings sınıfı 

- Bir sınıf oluşturur ve Binding uygular

```dart
class HomeBinding implements Bindings {}
```

IDE'niz otomatik olarak "bağımlılıklar" yöntemini geçersiz kılmanızı isteyecektir ve sadece lambaya tıklamanız, yöntemi geçersiz kılmanız ve o rotada kullanacağınız tüm sınıfları eklemeniz yeterlidir: 

```dart
class HomeBinding implements Bindings {
  @override
  void dependencies() {
    Get.lazyPut<HomeController>(() => HomeController());
    Get.put<Service>(()=> Api());
  }
}

class DetailsBinding implements Bindings {
  @override
  void dependencies() {
    Get.lazyPut<DetailsController>(() => DetailsController());
  }
}
```

Şimdi sadece rotanızı, rota yöneticisi, bağımlılıklar ve durumlar arasında bağlantı kurmak için bu bağlamayı kullanacağınızı bildirmeniz gerekiyor. 

- Adlandırılmış yolları kullanma: 

```dart
getPages: [
  GetPage(
    name: '/',
    page: () => HomeView(),
    binding: HomeBinding(),
  ),
  GetPage(
    name: '/details',
    page: () => DetailsView(),
    binding: DetailsBinding(),
  ),
];
```

- Normal yolları kullanma: 

```dart 
Get.to(Home(), bağlama: HomeBinding()); 
Get.to(DetailsView(), bağlama: DetailsBinding()) 
```

Orada, artık uygulamanızın bellek yönetimi konusunda endişelenmenize gerek yok, Get bunu sizin için yapacak. 

Bir rota çağrıldığında Binding sınıfı çağrılır, oluşturulacak tüm bağımlılıkları eklemek için GetMaterialApp'ınızda bir "initialBinding oluşturabilirsiniz. 

```dart 
GetMaterialApp( 
  initialBinding: SampleBind(), 
  home: Home(), 
); 
``` 

### BindingsBuilder 
Bağlam oluşturmanın varsayılan yolu, Bindings uygulayan bir sınıf oluşturmaktır. 
Ancak alternatif olarak, istediğiniz her şeyi başlatmak için bir işlevi kullanabilmeniz için `BindingsBuilder` geri çağrısını kullanabilirsiniz. 

Örnek: 

```dart 
getPages: [ 
  GetPage( 
    name: '/',
    sayfa: () => HomeView(), 
    binding: BindingsBuilder(() { 
      Get.lazyPut<ControllerX>(() => ControllerX()); 
      Get.put<Service>(()=> Api()); 
    } ), 
  ), 
  GetPage( 
    name: '/details', 
    sayfa: () => DetailsView(), 
    binding: BindingsBuilder(() { 
      Get.lazyPut<DetailsController>(() => DetailsController()); 
    }), 
  ) , 
]; 
``` 

Bu şekilde, her rota için bir Binding sınıfı oluşturmaktan kaçınarak bunu daha da basitleştirebilirsiniz. 

Her iki şekilde de gayet iyi çalışıyor ve zevkinize en uygun olanı kullanmanızı istiyoruz. 

### Akıllı Yönetim

GetX, bir hata oluşsa ve onu kullanan bir pencere öğesi düzgün şekilde atılmamış olsa bile, varsayılan olarak kullanılmayan denetleyicileri bellekten atar. 
Bu, "tam" bağımlılık yönetimi modu olarak adlandırılan şeydir. 
Ancak GetX'in sınıfların imhasını kontrol etme şeklini değiştirmek istiyorsanız, farklı davranışlar ayarlayabileceğiniz `SmartManagement' sınıfınız var. 

#### Nasıl 

değiştirilir Bu yapılandırmayı değiştirmek istiyorsanız (ki genellikle buna ihtiyacınız olmaz) şu şekildedir: 

```dart
void main () {
  runApp(
    GetMaterialApp(
      smartManagement: SmartManagement.onlyBuilder //here
      home: Home(),
    )
  )
}
```
#### SmartManagement.full


Varsayılan olanıdır. Kullanılmayan ve kalıcı olacak şekilde ayarlanmayan sınıfları atın. Çoğu durumda, bu yapılandırmaya dokunulmamasını isteyeceksiniz. GetX'te yeniyseniz, bunu değiştirmeyin. 

#### SmartManagement.onlyBuilder 
Bu seçenekle, yalnızca 'init:' ile başlatılan veya 'Get.lazyPut()' ile bir Binding'e yüklenen denetleyiciler atılır. 

`Get.put()` veya `Get.putAsync()` veya başka bir yaklaşım kullanırsanız, SmartManagement bu bağımlılığı dışlamak için izinlere sahip olmayacaktır. 

Varsayılan davranışla, SmartManagement.onlyBuilder'ın aksine "Get.put" ile örneklenen widget'lar bile kaldırılacaktır. 

#### SmartManagement.keepFactory

SmartManagement.full gibi, artık kullanılmadığında bağımlılıklarını kaldıracaktır. Ancak, fabrikalarını koruyacak, yani bu örneğe tekrar ihtiyacınız olursa bağımlılığı yeniden yaratacaktır. 

### Başlık altında bağlamalar nasıl çalışır 
Bağlamalar, başka bir ekrana gitmek için tıkladığınız anda oluşturulan ve ekran değiştirme animasyonu gerçekleşir gerçekleşmez yok edilecek olan geçici fabrikalar oluşturur. 
Bu o kadar hızlı gerçekleşir ki analizör onu kaydedemez bile. 
Bu ekrana tekrar gittiğinizde, yeni bir geçici fabrika çağrılır, bu nedenle SmartManagement.keepFactory kullanmak yerine bu tercih edilir, ancak Bindings oluşturmak istemiyorsanız veya tüm bağımlılıklarınızı aynı Binding üzerinde tutmak istiyorsanız, mutlaka size yardımcı olacaktır.
Fabrikalar çok az bellek kaplar, örnekleri tutmazlar, ancak istediğiniz sınıfın "şekli" olan bir işleve sahiptirler. 
Bunun bellekte maliyeti çok düşüktür, ancak bu kitaplığın amacı, minimum kaynakları kullanarak mümkün olan maksimum performansı elde etmek olduğundan, Get fabrikaları bile varsayılan olarak kaldırır. 
Hangisi sizin için daha uygunsa onu kullanın. 

## Notlar 

- Birden çok Bağlantı kullanıyorsanız SmartManagement.keepFactory KULLANMAYIN. Bindings olmadan veya GetMaterialApp'in initialBinding'inde bağlantılı tek bir Binding ile kullanılmak üzere tasarlanmıştır. 

- Bindings kullanmak tamamen isteğe bağlıdır, isterseniz belirli bir denetleyiciyi kullanan sınıflarda `Get.put()` ve `Get.find()` kullanabilirsiniz.
Ancak, Hizmetler veya başka bir soyutlama ile çalışıyorsanız, daha iyi bir organizasyon için Bindings'i kullanmanızı öneririm.
