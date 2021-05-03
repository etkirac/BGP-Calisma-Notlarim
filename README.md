# BGP (Border Gateway Protocol) 
BGP, genellikle Internet Service Provider (İnternet Servis Sağlayıcıları) tarafından kullanılan gelişmiş bir yönlendirme protokolüdür. BGP’de routerlara otonom sistem (AS) numarası tanımlanır. BGP, yönlendirme tablosunu oluşturmak için metrik hesaplar, bu hesaplama hedefe giderken üzerinden geçilen otonom sistem sayısıdır. Yani, BGP Path Vector (Yol Vektörü) algoritmasını kullanır. Fakat, Distance Vector algoritmasını kullanan EIGRP’nin aksine farklı otonom sistemler arasında da çalışabilmektedir. Ayrıca BGP, IP adreslerinin özetlenmesini sağlayan CIDR’i (Classless Inter Domain Routing) destekler.

### Path Attributes (Yol Özellikleri)
Path Attributes kısaca PAs, BGP içindeki yönlendirme politikalarının ayrıntı düzeyini ve denetimini sağlar ve BGP ilişkili her network yolu için PAs kullanır. PAs aşağıdaki gibi sınıflandırılır:
- Well-known mandatory
- Well-known discretionary
- Optional transitive
- Optional non-transitive

Well-konown mandatory, tüm BGP uygulamaları tarafından tanımlanması zorunludur. Well-known discretionary, her prefix advertisementlarına (reklam) dahil edilmelidir. Optional diye belirtilen isteğe bağlı özellikler prefix advertisementlara dahil edilebilir veya edilmeyedebilir. Non-transitive olan PAs otonom sistemden otonom sisteme advertisementlarla paylaşılmaz.

BGP’de belirli bir yol için prefix, prefix uzunluğu ve BGP PA’larından oluşan yönlendirme güncellemesine NLRI yani _Network Layer Reachability Information_ denir.

### Loop Prevention (Döngü Önleme)
BGP, link-state yönlendirme protokellerinde olduğu gibi ağın tam bir topolojisini içermez. BGP distance vector protokolleri gibi davranarak bir yolun döngü içermediğini garanti eder.

BGP özelliklerinden _AS_Path_ bir well-known mandatory özelliktir. AS_Path, BGP’de döngü önleme mekanizması olarak kullanılır. Kaynaktan hedefe giderken geçtiği otonom sistemlerin (AS’lerin) otonom sistem numaralarını (ASN’larını) taşıyan bir prefix advertisement üretilir. Bu advertisement sayesinde daha önce geçilen otonom sistem üzerinden tekrar geçilmek istenirse, router bu advertisementın bir döngü olduğunu düşünür ve düşündüğü için bu prefixi atar.

### Address Families
BGP, normalde IPv4 prefixlerini rotalamak için keşfedilmiştir. Fakat daha sonra, address family identifier (AFI) denilen uzantı ile birlikte Multi-Protocol BGP (MP-BGP) özelliği eklendi. Her address family BGP’deki her protokol için ayrı veritabanı ve konfigürasyon sağlar. Böylelikle farklı address family kullanarak aynı BGP oturumu içerisinde farklı policyler uygulamaya izin verir.

### Routerlar Arasındaki İletişim
BGP, IGP protokolleri gibi komşuları keşfetmek için “hello” paketlerini kullanmaz, komşuları dinamik olarak keşfedemez. BGP komşuları IP adresleriyle tanımlanır.

BGP komşuluğunun kurulabilmesinin birinci şartı, underlayde ağdaki cihazların birbirine erişimi olması gerekmektedir. Buna kavramsal olarak “next hop reachability” denir. İkinci şartı ise, TCP 179.portunun açık olması gerekir. Çünkü BGP, iletişim için TCP 179.portu kullanır.

BGP kendi arasında ikiye ayrılır:

##### 1.EBGP (Exterior Border Gateway Protocol) :
Farklı otonom sistemdeki routerların birbirleriyle komşuluk kurabilmesi için kullanılmaktadır. EBGP sayesinde öğrenilen rotalar, yönlendirme tablosuna administrative distance değeri 20 olacak şekilde eklenir.
##### 2.IBGP (Interior Border Gateway Protocol):
Aynı otonom sistemdeki routerların birbirleriyle komşuluk kurabilmesi için kullanılmaktadır. IBGP sayesinde öğrenilen rotalar, yönlendirme tablosuna administrative distance değeri 200 olacak şekilde eklenir.

### BGP Mesajları
BGP iletişim için 4 mesaj tipi kullanır. Bunlar:

##### 1.OPEN: 
OPEN mesajları BGP komşuluğu kurmak için kullanılır. Bu mesaj BGP versiyon numarası, otonom sistem numarası, hold time(bekleme süresi) vb. içerir.
##### 2.UPDATE: 
UPDATE mesajları, olası rotaları bildirir, önceden ilan edilen rotaları geri çekebilir veya her ikisini birden yapabilir.
##### 3.NOTIFICATION: 
NOTIFICATION mesajları, BGP oturumunda hold time değişmesi, komşuluk bilgilerinin değişmesi veya BGP oturumu sıfırlama talep edilmesi gibi bir hata algılandığında gönderilir. Bu, BGP bağlantısının kapanmasına neden olur.
##### 4.KEEPALIVE: 
KEEPALIVE mesajları, BGP komşuluğunun devam ettiğinden emin olmak için 60 saniyede gönderilen mesajlardır.

### BGP Komşuluk Durumları
BGP komşu routerlarla bir TCP oturumu oluşturur, buna “peer” denir. BGP peer’larını ve onların operasyonel durumlarını belirten bir tabloyu tutmak için finite-state machine (FSM) kullanılır. BGP oturumlarında aşağıdaki durumlar mevcuttur:

##### 1.IDLE: 
IDLE durumu, BGP FSM’nin ilk aşamasıdır. BGP, bir başlangıç olayı algılar ve BGP peer’ına bir TCP bağlantısı başlatmaya çalışır.
##### 2.CONNECT: 
CONNECT durumunda, BGP, TCP bağlantısını başlatır. Eğer three-way TCP handshake tamamlanırsa, BGP OPEN mesajını komşuya gönderir ve daha sonra OPENSENT durumuna geçer. Eğer belirlenen sürede bu handshake tamamlanmazsa yeni bir TCP bağlantısı denenir ve durum ACTIVE olur.
##### 3.ACTIVE: 
ACTIVE durumunda, yeni bir three-way TCP handshake başlatılır. Bir bağlantı kurulursa OPEN mesajı gönderilir ve durum OPENSENT olur. Bu TCP bağlantısı başarısız olursa CONNECT durumuna geri döner.
##### 4.OPENSENT: 
OPENSENT durumunda, kaynak routerdan bir OPEN mesajı gönderilmiştir ve diğer routerdan OPEN mesajı beklenmektedir. Bu mesaj alındığında, her iki OPEN mesajı karşılıklı kontrol edilir. Aşağıdaki öğeler incelenir:

-	BGP versiyonları eşleşmelidir.
-	OPEN mesajındaki otonom sistem numarası komşu için yapılandırılanla eşleşmelidir.
-	Güvenlik parametreleri (şifre ve TTL gibi) uygun şekilde ayarlanmalıdır.
-	Eğer OPEN mesajlarında herhangi bir hata yoksa, KEEPALIVE mesajı gönderilir ve daha sonra bağlantı durumu OPENCONFIRM’e çekilir. Eğer OPEN mesajında bir hata bulunursa NOTIFICATION mesajı gönderilir ve bağlantı durumu IDLE konumuna geri alınır. Bir de TCP bağlantı kesme mesajı alırsa bağlantı durumu ACTIVE olarak tanımlanır.

##### 5.OPENCONFIRM: 
OPENCONFIRM durumunda, BGP, KEEPALIVE veya NOTIFICATION mesajını bekler. Bir komşudan KEEPALIVE mesajı alınması durumunda bağlantı durumu ESTABLISHED olur. Hold time süresi dolarsa veya bir NOTIFICATION mesajı alınırsa durum IDLE konumuna taşınır.                                                                           
##### 6.ESTABLISHED: 
ESTABLISHED durumunda, BGP oturumu kurulur. BGP komşuları UPDATE mesajlarını kullanarak rotaları değiştirir.

### BGP Multihoming
Yedekliliği sağlamanın en iyi ve en basit yollarından biri ikinci bir yol sağlamaktır. İkinci bir yol eklemek ve bu peer bağlantısı üzerinden ikinci bir BGP oturumu kurmaya “_Multihoming_” denir. Çünkü yolları öğrenmek ve bağlantı kurmak için birden fazla oturum vardır. BGP varsayılan olarak sadece en iyi yolu yayınlar. Bu, ağ trafiğinin hedefe iletilmesi için sadece bir tane yol kullanması demektir.

### Regular Expressions (REGEX)
Regular expression, Türkçe anlamıyla Düzenli veya Kurallı İfadeler, genellikle harf ve işaretlerden oluşan karakterler dizisinin bazı kurallar çerçevesinde daha kısa bir ifadeyle belirlenmesini sağlayan yapıdır. Regular Expression ifadeleri ve anlamları aşağıdaki tabloda belirtilmiştir.

         . (Period)	          ->       Herhangi bir karakteri ifade etmek için kullanılır.
         [] (Brackets)	  ->       Dizideki karakterlerden biri ile eşleşmesi için kullanılır.
         ^ (Caret)	          ->       Dizinin başlangıcındaki karakteri ifade etmek için kullanılır.
         ? ( Question Mark)	  ->       Karakterin var olup olmadığından emin olunmadığı zaman kullanır.(Karakter ya yok ya da 1 kere var)
         $ (Dollar Sign)	  ->       Dizinin sonundaki karakteri ifade etmek için kullanılır.
         * (Asteriks)	  ->       Karakterin sıfır veya sıfırdan fazla kere kullanıldığından emin olunmadığı zaman    kullanılır. (Karakter ya yok ya da 1,2,3,.. kere olabilir.) 
        + (Plus Sign)	  ->       Karakterin bir kereden fazla var olup olmadığından emin olunmadığı zaman kullanılır. (Karakter 1,2,3,4,.. kere olabilir.)
         _ (Underscore)       ->       Karakterle direkt eşleştirmek istediğimiz zaman kullanılır.
         | (Pipe)	          ->       OR fonksiyonu gibi çalışır.
         -  (Hyphen)          ->	   Parantez içinde sayı aralığını belirtmek için kullanılır.
         () (Parentheses)	  ->      Bir dizi belirtmek için kullanılır.
      [^] (Caret in brackets) ->	   Parantez içinde listelenen karakterleri içermez.
      
Regular Expression, BGP’de filtreleme yaparak gözlem yapılabilmesi için ya da policy uygularken kullanılır. BGP özelinde bu ifadelerden bazılarının nasıl kullanıldığını örneklerde gösterilmiştir. 

Örneğin; 
1.	Bir policy uygulanırken “ permit ^200_ “ komutu kullanılırsa, sadece otonom sistem 200’den gelen rotalar görülür.
2.	Bir policy uygulanırken “ permit _200$ “ komutu kullanılırsa, sadece otonom sistem 200’den üretilen prefixler görülür.

### Route Maps
Route map, yönlendirme protokellerine çok sayıda farklı özellikler sağlar. Kısaca Route map, aynı ACL (Access List) gibi ağı filtrelemek için kullanılır fakat ACL’ye göre ek özellikler barındırır. Route map’ler BGP için kritik bir önem taşır.
	Bir route map’te 4 bileşen vadır. Bunlar:
-	Sequence Number: Route map’in işlem sırasını belirler.
-	Conditional Matching Criteria: Belirli bir sıra için prefix özellikleri tanımlar.
-	Processing Action: Prefix’e izin verir veya reddeder.
-	Optional Action: Router’da route-map’e nasıl başvurulduğuna bağlı olarak manipülasyonlara izin verir. Aksiyonlar, rota özelliklerinin değiştirilmesi, eklenmesi veya kaldırılmasını içerebilir.

Bir route-map oluşturmak için “ route-map {route-map-name} [permit|deny]  [sequence number] “ komutu kullanılır. Bazı kurallar uygulanır. Bunlar:
-	 Eğer Processing Action belirtilmediyse, varsayılan olarak prefix’lere izin verir. 
-	Eğer Sequence Number belirtilmediyse, varsayılan olarak 10’dan itibaren otomatik arttırılır.

##### _Conditonal Matching  (Koşullu Eşleştirme)_
Route-map bileşenleri ve işlem sırası belirlendikten sonra, bu bölümde rotanın nasıl eşleştirilebileceği ifade edilir. Koşullu eşleştirme için bazı örnekler tabloda açıklamalarıyla birlikte belirtilmiştir.

    match as-path 	             ->  Prefix’ler regex isteklerine bağlı olarak eşleşir.
    match ip address             ->  Prefix’ler ACL’de tanımlanan ip adresleriyle eşleşir. verir.
    match ip address prefix-list ->  Prefix’ler prefix-list ile tanımlanan ip adresleriyle eşleşir.
    match local-preference 	     ->  Prefix’ler BGP özelliği olan local-preference’a göre eşleşir.
    match metric 	             ->  Prefix’ler metric değerine bağlı eşleşir.
    match tag                    ->	 Prefix’ler sayısal bir etiketlemeye bağlı eşleşir.
    
Bu komutlar çoklu eşleşme izin verir.

##### _Optional Action (İsteğe Bağlı Aksyionlar)_
Ek olarak, prefix’in geçmesine izin verildiğinde, route map rotaların bazı özelliklerini değiştirebilir. İsteğe bağlı aksiyonlar için bazı örnekler aşağıda tabloda verilmiştir.

    set as-path prepend     ->	Prefix için otonom sistem yolunu öne ekler.
    set ip next-hop	        ->  Herhangi bir eşleşme için bir sonraki adımda olan IP adresini ekler.
    set local-preference	->  BGP özelliği olan local preference ekler.
    set metric              ->  Rota için belirlenen metric değerini değiştirir.
    set tag	                ->  Prefix’ler sayısal bir etiketlemeye bağlı eşleşir.
    set weight              ->	BGP özelliği olan weight değerini ekler.
    
Varsayılan olarak bir route map davranışı, route map’te sequence number’a göre işler ve ilk eşleşmede belirtilen aksiyonları yürütür, daha sonra işlemi durdurur. Bu, birden fazla sequence number olduğunda işlenmesini engeller. Bu durumu önlemek için “continue” komutunun eklenmesi gerekmektedir. 

### BGP Rota Filtreleme (BGP Route Filtering)
Rota filtreleme, komşu routerlardan tanıtılan ve alınan rotaları seçerek belirleme yöntemidir. Rota filtreleme, trafik akışlarını değiştirmek, bellek kullanımını azaltmak veya güvenliği artırmak için kullanılır. 

BGP rota filtreleme, spesifik olarak gelen ve giden trafik için kullanılabilir. Bu filtrelemeyi uygularken bazı metotlar kullanılır. Bunlar:
-	Distribute list: Distribute list, standart veya genişletilmiş bir Access list ile prefixlerin filtrelenmesini içerir.
-	Prefix list: Prefix list, Access list’e benzer şekilde yukardan aşağıya prefixlere izin verir veya reddeder.
-	AS path ACL/filtering: Regex komutları kullanılarak belirtilen otonom sistemlerinin prefixlere izin verilmesine veya reddedilmesine izin verir.
-	Route map: Route map, çeşitli prefix nitelikleri üzerinde koşullu eşleştirme ve çeşitli aksiyonlar gerçekleştirme yöntemi sağlar. Bu aksiyonlar, basit bir izin veya reddetme olabilir; veya BGP yol niteliklerinin değiştirilmesini içerebilir.

### BGP Toplulukları (BGP Communities)
BGP community, rotaları etiketlemek ve routerlarda BGP yönlendirme politikasını değiştirmek için ek yetenek sağlar. Bir rota routerdan routera ilerlerken, BGP community her bir niteliğe eklenebilir, kaldırılabilir veya değiştirilebilir. 

###### _Well-Known Communities (İyi Bilinen Topluluklar)_
-	Internet: İnternette reklamı yapılması gereken yolları belirlemek için standartlaştırılmış bir topluluktur.
-	No_Advertise: Bu topluluğa sahip rotalar, herhangi bir BGP komşusuna (iBGP veya eBGP) tanıtılmamalıdır.
-	No_Export: Bu toplulukla bir rota alındığında, rota herhangi bir eBGP eşine bildirilmez. Bu topluluğa sahip rotalar iBGP eşlerine ilan edilebilir.

### BGP Yol Seçimi (BGP Path Selection)
BGP en iyi yol seçimi, trafiğin otonom sisteme nasıl girdiğinden ve çıktığından etkilenir. Bazı routerlarda BGP nitelelikleri modifiye edildiğinde gelen ve giden trafik de etkilenir. Routerlar en iyi yol olarak her zaman prefix uzunluğunun en çok eşleştiği rotaları seçer. Buna “longest prefix match” denir. Eğer bu rotalarda eşleşme eşitse, sırayla aşağıdaki kriterlere bakılarak seçim yapılır.
1.	Weight (Yüksek olan tercih edilir.)
2.	Local Preference (Yüksek olan tercih edilir.)
3.	Lokal kaynaklı rotalar
4.	AS_Path ( En kısa otonom sistem yolu tercih edilir.)
5.	Rotanın kökeni (Tercih sırası IGP, Incomplete)
6.	MED değeri (En düşük metrik tercih edilir.)	
7.	eBGP, iBGP’e göre tercih edilir.
8.	En düşük IGP metric
9.	Eğer ikisi de eBGP ise, eski olan rota tercih edilir.
10.	En düşük router-id
11.	En düşük IP adresine sahip komşu

