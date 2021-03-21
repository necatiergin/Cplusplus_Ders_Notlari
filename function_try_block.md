#### işlev try bloğu (function try block)

Bir sınıfın kurucu işlevi görevini yerine getiremeyeceğini anladığında ne yapmalı? Bunu herkes biliyor, bir hata nesnesi göndermeli. 
Peki ya eğer kurucu işlevin  öğe ilk değer verme listesiyle _(member initializer list)_ hayata getirilen sınıf öğelerinin kurucu işlevlerinin birinden hata nesnesi 
gönderilirse ne olacak? Bu durumda programın akışı sınıfımızın kurucu işlevinden çıkacak değil mi? Aşağıdaki koda bir bakalım:
```
class Member {
	//
public:
	Member(int);
	//
};


class Owner {
	Member m;
public:
	Owner() : m(0) 
	{
		///
	}
};
```
_Owner_ sınıfının _Member_ sınıfı türünden bir veri elemanı var ve _Owner_ sınıfının varsayılan kurucu işlevi bu elemanı, öğe ilk değer verme listesiyle hayata getiriyor. 
_Owner_ sınıfının kurucu işlevi çağrıldığında _Member_ sınıfının _int_ parametreli kurucu işlevinden bir hata nesnesi gönderilirse ne olur? Öncelikle _C++_ dilinin çok net bir kuralını bilmemiz gerekiyor. Eğer bir sınıf nesnesinin kurucu işlevinin kodu tamamen yürütülmez ise derleyici bu sınıf nesnesi için sonlandırıcı işlevi çağırmaz. Bu kuralın gerekliliği çok açık: Sınıfın kurucu işlevi içinden, hata nesnesinin gönderildiği kaynak kod noktasından sonra birtakım kaynakların elde edildiğini düşünelim. 
Bu durumda sonlandırıcı işlev çağrılsaydı, hata nesnesinin gönderilmesi durumunda  sonlandırıcı işlev edinilmemiş kaynakları geri verme girişiminde bulunur bu da çalışma zamanı hatalarına neden olurdu. Yani en azından şöyle bir güvencemiz var: _Member_ sınıfının kurucu işlevinden bir hata nesnesinin gönderilmesi durumunda, ne _Member_ ne de _Owner_ sınıfının kurucu işlevinin kodu tamameen yürütülmüş olacak ve her iki sınıfın da sonlandırıcı işlevi çağrılmayacak.
<br>

Sorumuza geri dönelim: 
Sınıfımızın ilk değer verme listesinin yürütülmesi sürecinde bir hata nesnesi gönderilirse ne yapmalıyız? Eğer yanıtınız hiç bir şey yapmamalıyız ise bu yanıt çoğunlukla yanlış. En azından yapmamız gereken gönderilen hata nesnesini yakalayarak kurucu işlevimizin hata durumunu loglamak ve yakalanan hata nesnesini yeniden göndermek _(rethrow)_. 
Bazı durumlarda _rethrow_ işlemiyle yakalanan hata nesnesini yeniden göndermek yerine, yukarıdaki daha yüksek seviyeli kodlara durumu daha iyi anlatacak, kendi belirlediğimiz bir türden hata nesnesi göndermeyi de tercih edebiliriz.

Peki hata nesnesini yakalamak için ne yapacağız? Kurucu işlevimizin kodunun tamamını bir _try_ bloğu içine alsak bile öğe ilk değer verme listesinden gönderilecek hata nesnelerini yakalayamayız. Çünkü öğe ilk değer verme listesi kodları _try_ bloğu içinde yer almıyor:

```
class Member {
	//
public:
	Member(int);
	//
};


class Owner {
	Member m;
public:
	Owner(int x) : m(x) 
	{
		try{
			//kurucu işlevin kodu
		}
		catch (...) {
			///
		}
	}
};
```

Yukarıdaki kodda _Owner_ sınıfının kurucu işlevinin tüm kodlarının _try_ bloğu içinde yer aldığını düşünelim. _Member_ sınıfının kurucu işlevinden bir hata nesnesi gönderilirse programın akışı _catch_ bloğuna girmeyecek. _C++_ dili öğe ilk değer verme listesinden gönderilebilecek hata nesnelerinin yakalanmasına olanak verecek özel bir _try_ bloğu oluşturmaya olanak sağlıyor.  Böyle _try_ bloklarına _"işlev try bloğu"_ deniyor. İşlev _try_ bloğu hem kurucu işlevin öğe ilk değer verme listesini hem de ana bloğunu kapsayan bir _try_ bloğu olarak düşünülebilir:

```
class Member {
	//
public:
	Member(int);
	//
};


class Owner {
	Member m;
public:
	Owner(int x) try : m(x)  
	{
		//kurucu işlevin kodu
	}
	catch (...) {
		///
	}
};
```
Şimdi de işlev _try_ bloklarıyla ilgili kodlama açısından önemli olabilecek noktalara değinelim:

- Bir sınıfın öğe ilk değer verme listesinden bir hata nesnesi gönderilirse, ilk değer listesiyle hayata getirilmiş (yani kurucu işlev kodunun tamamı çalıştırılmış) öğe nesnelerin sonlandırıcı işlevleri, programın akışının işlev _try_ bloğuna ilişkin _catch_ bloğuna girmesinden önce çağrılır. 
- Kurucu işlevin parametre değişkenleri sınıf türlerinden ise bu parametre değişkenleri için çağrılacak olan kopyalayan kurucu işlev, taşıyan kurucu işlev ya da sonlandırıcı işlevlerden gönderilen hata nesneleri işlev _try_ bloğu ile yakalanamazlar. 
- Programın akışı işlev _try_ bloğunu izleyen bir _catch_ bloğuna girdiğinde bu catch bloğu içinde program sonlandırılmaz ya da _catch_ bloğundan bir hata nesnesi (yeniden) gönderilmez ise derleyicinin ürettiği kod ile otomatik olarak hata nesnesi yeniden gönderilir _(rethrow)_.

İşlev _try_ bloklarının genel kullanım amacı kurucu işlevlere yönelik olsa da herhangi bir global işlev ya da bir sınıfın üye işlevi de işlev _try_ bloğuna sahip olabilir. Örneğin dilersek _main_ işlevi için bir işlev _try_ bloğu oluşturabiliriz:

```
int main() 
try {
	//
}
catch (...) {
	///
}
```

Normal _try_ bloklarında olduğu gibi işlev _try_ bloğu içinde tanıtılan isimler de _try_ bloğunu izleyen catch blokları içinde bilinmezler. 
Ancak işlevin parametre değişkenleri _catch_ bloklarında da bilinirler ve programın akışının bir _catch_ bloğuna girmesi durumunda parametre değişkenleri _catch_ bloğunun kodunun çalışması bitene kadar hayatta kalırlar:

```
#include <iostream>

void func(int x)
try {
	throw 1;
}
catch (...) {
	std::cout << "x = " << x << "\n";
	++x;
	std::cout << "x = " << x << "\n";
}

int main()
{
	func(10);

	return 0;
}
```

Bir işlev _try_ bloğunu izleyen _catch_ bloğu içinde bir _return_ deyimi yer alabilir. Böyle bir _return_ deyimi _try_ bloğuna konu işlevin return deyimidir. _catch_ bloğu içinde bir _return_ deyimi yok ise programın akışı _catch_ bloğunun sonuna geldiğinde işlevin çalışan kodu sonlanmış olur. Eğer işlev _void_ değilse yani işlevin bir geri dönüş değeri türü var ise programın akışının _catch_ bloğunun sonuna kadar gelmesi tanımsız davranıştır.

İsim alanı kapsamında  _(namespace scope)_ oluşturulmuş statik ömürlü bir nesnenin kurucu işlevinden gönderilecek bir hata nesnesi _main_ işlevi için oluşturulacak olan bir işlev _try_ bloğu içinde yakalanamaz. statik ömürlü global nesnelerin kurucu işlevleri _main_ çağrılmadan önce çağrılmaktadır.
