# Modern C++ Çalışma Notları

Henüz tamamlanmamış olan bu çalışmada modern c++ notlarımı tutuyorum. c++11 ile başlayıp c++14 ve c++17 şeklinde devam etmeyi planlıyorum.

### Compiler Generated Default Constructor
Aşağıdaki gibi bir sınıfımız olduğunu düşünelim.

```c
class Daire{
    int yaricap;
    public:
        Daire(int r) {
            yaricap = r;
        }
}
```

Yukarıdaki sınıf için `Daire daire1` şeklinde nesne oluşturmaya çalıştığımızda hata alırız çünkü Daire sınıfını parametresiz constructor ile çağırmak istiyoruz ancak öyle bir constructor tanımlamamız yok. Bu sorunu aşmak için kodu aşağıdaki şekilde düzeltmek yeterli olacaktır.

```c
class Circle{
    int radius;
    public:
        Circle() = default;
        Circle(int r) {
            radius = r;
        }
}
```

### final Belirteci
Bir sınıfın başka bir sınıf tarafından miras alınmasını istemiyorsak veya bir methodun miras alındıktan sonra override edilmesini istemiyorsak **final belirtecini** kullanabiliriz.

```c
class Circle final{
    int radius;
    public:
        Circle() = default;
        Circle(int r) {
            radius = r;
        }
        void printRadius() final;
}
```

### override Belirteci
override belirteci derleyiciye ve kodu okuyan kişiye ilgili metodun miras alınan sınıftan override edildiğini bildirir. Kullanılması zorunlu olmayan bu belirtec kullanıldığında birçok hatayı önceden bize gösterebilir. Örneğin aşağıdaki durumları ele alalım.

```c
// Hata vermez
class Base{
  public:
  virtual void A(float a){};
};

class Derived: Base{
  public:
  void A(float a) override {std::cout << "Hello World!\n";};
};
```

```c
// A metodu virtual olmadığı için hata verir
class Base{
  public:
  void A(float a){};
};

class Derived: Base{
  public:
  void A(float a) override {std::cout << "Hello World!\n";};
};
```

```c
// Base class'ta void A(int) methodu olmadığı için hata verir
class Base{
  public:
  virtual void A(float a){};
};

class Derived: Base{
  public:
  void A(int a) override {std::cout << "Hello World!\n";};
};
```

Eğer override belirtecini kullanmamış olsaydık. Yukarıdaki durumlardan hiçbiri hata vermeyecekti ancak biz doğru yaptığımızı düşünüp programlamaya devam edecek ve ilerleyen kısımlarda bir sorunla karşılaştığımızda hatanın kaynağını bulmak için daha fazla çaba harcayacaktık.

### Constructor Delegation  
Birden fazla constructor bulunduran sınıflarda tüm contructor'ların belli bir kısmında aynı işlemler yapılıyor olabilir. Bu gibi durumlarda kodu daha düzgün bir hale sokmak için aşağıdaki yöntem kullanılabilir.

```c
class Dog{
    void init() {//ortak işlemler};
    dog() {init();}
    dog(int i) {init(); // diğer işlemler}
}
```

Bu yaklaşımda init motodu diğer tüm metotlar ile kullanılabildiğinden güvensizdir bu nedenle bu yöntem yerine yerine **C++11** ile birlikte aşağıdaki yöntem geldi. 

```c
class Dog{
    dog() {//ortak işlemler}
    dog(int i):dog() {// diğer işlemler}
}
```
### auto tipi
Bir değikenin tipinin otomatik olarak bulunması için auto tipi eklendi, aşağıdaki şekilde kullanılabiliyor ancak dezavantajı IDE'lerin değişkenin tipini anlayıp otomatik tamamlama yapmaları zorlaştı.

```c
auto a = 6;                 // int
auto b = 9.6                // double
auto c = a;                 // int

std::vector<int> vect;
auto it = vect.begin()    // std::vector<int>::iterator
```

### foreach Döngüsü
For döngüsü ile birlikte aşağıdaki şekilde liste içindeki elemanlara ulaşılabilir.

```c
    for(int i:arr)
        std::cout << i << ", ";    
```

### nullptr
NULL değeri 0 olduğundan iki anlamlıdır ve bu durum bazı yanlış çalışmalara yol açabilir. Aşağıdaki durumu ele alalım.

```c
void foo(int i) {};
void foo(char *p) {};

foo(NULL);          // NULL iki anlamlı olduğundan hata verir
foo(nullptr);       // foo(char*) fonksiyonunu çağırır
```

### enum class

Aşağıdaki örnek üzerinden ilerleyelim.

```c
enum color{"green", "red};
enum size{"big", "small"};

color g = green;
size s = big;
```

Yukarıdaki örnekte `g==s` sorgusu true değerini döndürür, bunu önlemek için enum class yapısı kullanılabilir.

```c
enum class color{"green", "red};
enum class size{"big", "small"};

color g = color::green;
size s = size::big;
```

### static_assert
Önceden assert metodu runtime zamanında çalışırdı. Buna ek olarak derleme sırasında kontrol edilen static_assert terimi eklendi.

```c
assert(myPointer != NULL);      // runtime assert
static_assert(sizeof(int)==4)   // compile time assert

static_assert ( bool_constexpr , message ) 		(since C++11)
static_assert ( bool_constexpr ) 		        (since C++17)
```

### delete ifadesi

```c
class Dog{
    Dog(int i) {};
}

Dog a(2);       // çalışır
Dog b(3.2);     // çalışır
a = b;          // çalışır
```

Yukarıdaki üç örnekte hatasız şekilde çalışır çünkü derleyici bizim yerimize ilgili fonksiyonları tanımlar. Eğer bu fonksiyonların tanımlanmasını istemiyorsak aşağıdaki şekilde **delete** ifadesinden yararlanabiliriz.

```c
class Dog{
    Dog(int i) {};
    Dog(double i) = delete;
    Dog& operator(const Dog&)=delete;
}
```

### constexpr belirteci
constexpr belitecinin amacı bazı hesaplamaların derleme esnasında yapılması böylece kullanıcının bu hesaplamalar için çalışma anında vakit kaybetmemesidir.

```c
constexpr cubed(int x) {return x*x*x};  
int y = cubed(9999);    // derleme esnasında hesaplandığı için hızlıdır
```

### Lambda İfadeleri
Genellikle çok büyük olmayan fonksiyonlar lambda ifadeleri kullanılarak yazılabilir. Bu programcıya okuma kolaylığı sağlar. Aşağıda genel yapısı verilen bu ifadeler için return type genellikle derleyici tarafından otomatik olarak bulunur.

```
[ capture clause ] (parameters) -> return-type  
{   
   definition of method   
}

capture clause için aşağıdakiler kullanılabilir

[]      lambda fonksiyonu scope içinedeki değişkenlere erişemez
[&]     lambda fonksiyonu scope içinedeki değişkenlere referans ile erişir
[=]     lambda fonksiyonu scope içinedeki değişkenlerin kopyasına erişir
[&, t]  t değişkeninin kopyasına diğer değişkenlerin referansına erişir
[=, &t] t değişkeninin referansına diğer değişkenlerin kopyasına erişir

```

Aşağıda ise basit bir lambda fonksiyonu bulunmaktadır.

```c
auto lambda = []() { cout << "Hello lambda" << endl; };
```

Aşağıda ise filter motodu ile birlikte kullanımı görünmektedir.

```c
vector<int> v = {1,2,3,4,5,6};
filter([](int x){return x>3}, v);   // 4, 5, 6

//eğer değişkenleri referasn olarak geçirmek istersek
```

### Sağ Değer Referansları

Sağ değerler genellikle atama operatörünün sağ tarafından bulunan herhangi bir tanımlayıcısı(identifier) olmayan değerlerdir. C++ diline move semantiği ve perfect forwarding gibi katkıları vardır. Örneğin `a=5` ifadesinde a **sağ değer**, 5 ise **sol değerdir.** Aşağıdaki örnekte sağ ve sol değerlerin fonksiyonlara nasıl parametre olarak geçirildiği görülmektedir.

```c
void print(int &x){cout << lvalue;}
void print(int &&x){cout << rvalue;}

a = 5;
print(a);       // lvalue
print(5);       // rvalue
```

`int &&` tanımı sağ değer referansı olarak adlandırılabilir. Yukarıdaki örnekte 5 değerinin kopyalanarak değil referans yoluyla print fonksiyonuna geçirilmesini sağlar.

Eğer yukarıdaki **print** methodlarına ek olara `void print(int x)` fonksiyonuda tanımlanmış olsaydı programımız hata verecekti çünkü derleyici hangi fonksiyonu çağıracağını bilemez. `void print(int x)` fonksiyonu parametre olarak hem sağ hem sol değerleri alabilir.

### Move Yapısı

Sağ değer referanslarını anlatırken bu yapının fonksiyonlara bir sağ değer geçirildiğinde kopyalanma maliyetinden kullanıcıyı kurtardığından bahsetmiştik. Modern C++ içinde tanımlı container nesneleri için bu yapıyı kullanmamızı sağlayan move constructor ve move operatörleri tanımlanmış ancak bizim oluşturduğumuz sınıflarda bu yapıları bizim kodlamamız gerekli, aşağıdaki örneği ele alalım.

```c
class MyVector{
    int size;
    double *arr;
  public:
    MyVector(const MyVector& value){
      //deep copy
    }    
    MyVector(const MyVector&& value){
      // move constructor
      size = value.size;
      arr = value.arr;
      // aynı veriye iki farklı nesnenin erişmemesi için
      // move edilen nesnenin önceki sahibi nullptr yapılır
      value.arr = nullptr;
    }
    ~MyVector(){delete arr;}
};

// Aşağıda aynı işi yapan 2 farklı fonksiyon vardır
void foo(MyVector v);
void foo_by_ref(MyVector &v);

int main{
  MyVector v;
  // v nesnesinin çeşitli verilerle doldurulduğunu varsayalım

  foo_by_ref(v);      // 1
  foo(v);             // 2
  // Aşağıdaki tanımlamadan sonra v nesnesi artık kullanılmaz çünkü move edilmiştir.
  foo(std::move(v));  // 3
}
```

**1** numaralı örnek nesneye refeans ile **2** numaralı örnek copy constructor üzerinden deep copy yaparak **3** numaralı örnek ise move constructor üzerinden nesneyi move ederek erişir. Performans yönünden değerlendirmeye kalkarsak en az iş yükü getiren yöntemden en fazla iş yükü getiren yönteme doğru sıralanmış hali **1,3,2** şeklinde olur.

Bir önceki paragrafta söylediğim gibi move yapısı c++11 ile birlikte tüm stl fonksiyonları için oluşturulduğu için eski bir kodu c++11 veya üstü bir derleyici ile derlediğimizde kodda hızlanma görülebilir. 

### Referans Yığılma Kuralı

```
  T& &        ---> T&
  T& &&       ---> T&
  T&& &       ---> T&
  T&& &&      ---> T&&
```
Yukarıda verilen tablo yardımıyla aşağıdaki örneğin nasıl çalıştığını anlamaya çalışalım.

```c
template<typename T>
void foo(T&& value){//yapilan islemler}

int main(){
  int c = 4;
  foo(9);   // T = int&& ----> value = int&& && = int&& = sağ değer
  foo(c);   // T = int & ----> value = int& &&  = int & = sol değer
}
```

Yukarıdaki örnekten görüldüğü gibi `T&&` template type'lar ile birlikte kullanılırken universal referans olarak görünür.

### Perfect Forwarding

```c
void foo(int value);
template<typename T>
void foo2(T value){
  foo(value);       // 1
}

int main(){
  int a = 5;
  foo2(a);           // 2
  foo2(7);           // 3
}
```

Yukarıdaki örnekte **2** ve **3** ile işaretlenen fonksiyonların ilgili fonksiyon içerisinde ki **foo**(**1** ile işaretli yer) fonksiyonunu rlavue mi yoksa lvalue şeklinde mi çağırması gerektiğini garanti etmemiz gerekir. Bunun için kullanılan aşağıdaki yapıya ise perfect forwarding denir. 

```c
void foo(int value);
template<typename T>
void foo2(T&& value){
  foo(std::forward<T>(value));       // perfect forwarding
}

int main(){
  int a = 5;
  foo2(a);          
  foo2(7);           
}
```

Kullanılan `std::forward` yapısı aldığı değişkenin tipine göre sağ veya sol değer döndürür, `std::move` yapısı ise sadece sağ değeri geri döndürür.

### Shared Pointer

C++11 öncesi kullanılan raw pointer'lar bellek yönetimini çeşitli yönlerden zorlaştırıyordu. Aynı bellek alanının birden fazla kez silinmeye çalışılması veya silinmesinin unutularak memory leak durumlarının ortaya çıkmasına sebep olabilir. Ayrıca silinmiş bir pointer aslında havada asılı duran bir nesne gibidir. Daha sonra bu pointerı kullanmak istediğimizde beklenmeyen sonuçlar alabiliriz, aşağıda bununla ilgili bir örnek mevcut.

```c
#include <iostream>
#include <vector>

using namespace std;
int main() {
  vector<int> *vect = new vector<int>;
  vect->push_back(7);

  delete vect;
  std::cout << vect->at(0);       // ekrana 0 yazar
}
```

Shared pointer kabaca bir sayaç sistemiyle çalışır. Eğer pointer'ı kullanan nesne sayısı 0 olursa ilgili pointer silinir. Aşağıya örnek kullanımını verelim.

```c
#include <iostream>
#include <memory>
#include <vector>

using namespace std;
int main() {
  // shared_ptr nesnemizi oluşturuyoruz
  // p pointer nesnesi int tutan bir vector'e işaret ediyor
  shared_ptr<vector<int>> p(new vector<int>);
  p->push_back(7);
  // shared_ptr nesnesinin metotlarına . ile
  // işaret ettiği vector<int> nesnesinin metotlarına ise -> ile erişiyoruz
  // use_count() metodu o an pointerı kullanan nesne sayısını döndürür
  std::cout << p->at(0) << " " << p.use_count() << endl;

  {
    // p pointer'ının işaret ettiği alana işaret eden yeni bir
    // pointer nesnesi oluşturuyoruz
    shared_ptr<vector<int>> p2 = p;
    std::cout << p2->at(0) << " " << p.use_count() << endl;
  }
  std::cout << p->at(0) << " " << p.use_count() << endl;
}
```

```
output
7 1
7 2
7 1
```

Çıktıdan görüldüğü gibi aynı alana işaret eden shared_ptr nesnelerinden biri silindiğinde counter değeri azalıyor. Counter sıfır olduğunda ise ilgili alan tamamen bellekten temizleniyor.

Aşağıda ise bu yapının nasıl kullanılmaması gerektiğiyle ilgili bir örnek var. Bu örneği tehlikeli yapan nokta **p** ve **p2** nesnelerinden direk ilgili bellek alanı üzerinden oluşturulması, **p** ve **p2** nesnelerinden biri silindikten sonra diğeri silinmeye çalışıldığında zaten silinmiş olan bellek alanı tekrar silinemeyeceği için hata verecektir.

```c
vector<int> *vect = new vector<int>;
shared_ptr<vector<int>> p(vect);
shared_ptr<vector<int>> p2(vect);
```

Son olarak ise **shared_ptr** oluşturmanın bir başka yolunu vereceğim. Bu yol ilk gösterdiğime göre daha hızlı ve güvenlidir. Güvenli olma sebebi, ilk yöntemde önce nesne oluşturulup ardından **shared_ptr** nesnesi oluşturuluyor. Bu işlemler sırasında nesnemiz başarıyla oluştuğu halde **shared_ptr** nesnemiz oluşturulurken hata verebilir. Bu durumda memory leak yaşamış oluruz.

```c
  shared_ptr<vector<int>> p = make_shared<vector<int>>();
```

### Weak Pointer

```c
class Dog{
  shared_ptr<Dog> friend;
  string name;
  public:
  Dog(string n){name = n;}
  void makeFriend(shared_ptr<Dog> f){friend = f;}
}

int main(){
  shared_ptr<Dog> d1(new Dog("1"));
  shared_ptr<Dog> d2(new Dog("2"));
  d1->makeFriend(d2);                 // 1
  d2->makeFriend(d1);                 // 2
}
```

Yukarıdaki örnekte **1** ve **2** nolu satırlar nedeniyle d1 ve d2 nesneleri bellekten silinemezler çünkü iki nesne içinde sayaç asla 0 olmaz. Bunu önlemek için weak pointer'lar kullanılabilir.

Weak pointer nesneleri tek başına bir anlam ifade etmezler, bir shared pointer ile ilişkilidir. Sadece **shared_ptr** veya **weak_ptr** nesnelerinden oluşturulabilir. Sadece okuma hakkına sahiptir ve `lock` metodu ile işaret ettiği **shared_ptr** elde edilir. Sayacı artırmazlar böylece yukarıda karşılaşılan problem weak pointer'lar ile yaşanmaz. Oluşturmak içinse aşağıdaki yöntem kullanılabilir.

```c
shared_ptr<int> p(new int(3));
weak_ptr<int> q(p); 
shared_ptr<int> r = q.lock()
```

### Unique Pointer

Birkaç **unique pointer** aynı adresi gösteremez, her `unique_ptr` eşsizdir ve tek sahipleri vardır. **Shared pointer'a** oranla daha az kaynak tüketirler.

```c
unique_ptr<int> p(new int(1));
// release metodu weak_ptr nesnesinin sahipliğine son verir ve raw pointer adresini geri döndürür
int *d = p.release();

unique_ptr<int> d1(new int(1));
unique_ptr<int> d2(new int(2));
// aşağıdaki metod 2 değerini taşıyan int* değişkenini siler,
// 1 değerini taşıyan int* değişkenini ise d2 nesnesi ile ilişkilendirir.
d2 = std::move(d1);
```

Bir **unique pointer** nesnesi aşağıdaki gibi bir fonksiyona parametre olarak geçirilirse, ilgili pointerın sahipliği fonksiyon içindeki **p** değişkenine devredilir(`std::move`). Bu sebeple `foo` metodundan çıkıldığında sayaç sıfırlandığı için pointer silinir. 

```c
void foo(unique_ptr<int> p){//işlemler}

int main(){
  unique_ptr<int> d1(new int(1));
  foo(d1);

  // Artık d1 pointerı kullanılamaz
}
```

**Unique pointer'lar** sınıf dışına paylaşılmayacak pointerlar için uygun bir seçimdir.