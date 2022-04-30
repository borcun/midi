# MIDI Tutorial

Bu MIDI dersi sana, MIDI protokolu kullanan herhangi bir cihazi kontrol etmek icin MIDI dilini nasil\
kullanacagini anlaman konusunda yardimci olacaktir.

## MIDI Tutorial Part 1 - MIDI Messages

MIDI dili, bir muzik parcasinin banttan calinmasi icin gercek zamanli bilgi gondermek amaciyla\
kullanilir.

Gercek zamanli terimi, bir mesajin, tam olarak hedef sentezleyici (donanimsal veya yazilimsal bir\
sentezleyici olabilir) tarafindan yorumlanmasi gereken anda gonderilmesi anlamina gelmektedir.

Bir muzigin banttan calinmasi icin gerekli olan bilgiyi gondermek icin farkli mesajlar tanimlanmistir.

Burada onemli olan nokta; MIDI dili sesin kendisini tanimlanamaz, ancak hedef sentezleyicideki sesi\
yaratmak icin gerekli olan komutlar serisini tanimlar.

MIDI mesajlari, zamanl-sirali bir veya daha fazla byte (8 bit) verisi olarak gonderilirler. Ilk byte\
**STATUS byte** verisidir, cogunlukla bu veriyi ek parametrelere sahip **DATA byte** verisi takip eder.\
Bir **STATUS byte** verisi 1'e ayarlanmis 7 adet bit'e, **DATA byte** verisi 0'a ayarlanmis 7 adet bit'e\
sahiptir.

**STATUS byte** verisi MIDI mesajinin tipini belirler. Onu takip eden **DATA byte** verisinin uzunlugu\
mesajin tipine baglidir.

Bazi sistem MIDI mesajlari disinda, **STATUS byte** verisi MIDI kanal sayisini icerir. Hexadecimal\
formatta ve 0'dan 15'e kadar numaralandirilmis 16 MIDI kanali vardir. Pratikte, muzisyenler ve\
yazilimlar, MIDI kanallarini 1'den 16'a kadar isimlendirirler, bu nedenle onlari hexadecimal olarak\
programladiginda aradaki 1 fark olacaktir. (kanal "1" "0" olarak, kanal "10" "9" olarak ve kanal "16"\
"F" olarak kodlanir.

Ayni MIDI kablosunda, 16 adete kadar enstrumani bagimsiz olarak kontrol etmek icin 16 adete kadar\
MIDI kanali olabilir.

### MIDI RUNNING STATUS

Bir MIDI mesaji okunurken, aslinda **STATUS byte** verisinin cikarilabilecegini bilmeniz gerekmektedir.\
(Ilk mesaj icerisinde gonderilen, mesajin tipini soyleyen **STATUS byte** verisi haric)

Boyle bir durumda, sadece **DATA byte** verisini iceren mesaj alabilirsiniz. Akabinde, **STATUS byte**\
verisinin, en son alinan **STATUS byte** verisi ile ayni oldugu kabul edilir.

Bu surec *MIDI RUNNING STATUS* olarak isimlendirilir. Ayni mesajin, uzun bir seri icerisinde gonderildigi\
durumlarda, veri transferini optimize etmek icin fayda saglamaktadir. Ornegin; pitch bend (zift kivrimi)\
veya crescendo volume curve (krisendo hacim egrisi).

*MIDI RUNNING STATUS* olayini, MIDI mesajlari urettiginiz zamanlarda da kullanabilirsiniz, ancak\
hedef synthesizer veya yazilimin bu durum bilgisini nasil aldigi ve onu iyi bir sekilde yorumladigi\
konularinda dikkatli ve emin olmalisiniz.

## MIDI Tutorial Part 2 - NOTE Messages

Temel mesajlar **NOTE ON** ve **NOTE OFF** mesajlaridir.

**NOTE ON** mesaji, muzisyen muzik klavyesinin bir tusuna vurdugu anda gonderilir. Mesaj nota hizinin\
(tusa basildigi andaki notanin yogunlugu) yani sira pitch bilgisini de saglayan parametreler icer.

Bir sentezleyici bu mesaji aldiginda, ilgili notayi dogru pitch ve guc seviyeleri ile birlikte calmaya\
baslar.

**NOTE OFF** mesaji alindiginda, ilgili mesajdaki nota, sentezleyici tarafindan kapatilir.

### NOTE ON - OFF

Her *NOTE ON* mesajinin bir *NOTE OFF* karsiligi olmalidir, aksi takdirde nota sonsuza kadar calacaktir.\
Perkusyon enstrumanlari icin bir istisna vardir, bir perkusyon enstrumani sadece *NOTE ON* gonderebilir,\
cunku perkusyon notasi kendi kendine otomatik olarak durur. Ancak, her durumda *NOTE OFF* gondermek\
daha iyi bir pratiktir, cunku mesaji alan sentezleyici tarafindan nasil yorumlanabilecegine emin olamazsiniz.

**NOTE ON** mesaj yapisi su sekildedir:

 1. Status byte : 1001 CCCC
 2. Data byte 1 : 0PPP PPPP
 3. Data byte 2 : 0VVV VVVV
 
 - "CCCC", MIDI kanali (0'dan 15'e kadar)
 - "PPP PPPP", pitch degeri (0'dan 127'e kadar)
 - "VVV VVVV", velocity degeri (0'dan 127'e kadar)
 
Pitch degeri, calinacak notanin frekansini belirler. 0'dan 127'e kadar bir araliktadir, \
60 degeri ile *Middle C* notasi gosterilir.

![*Middle C*](https://www.cs.cmu.edu/~music/cmsip/readings/MIDI%20tutorial%20for%20programmers_files/midi-note-c60.jpg)

Notanin degeri, yarim adimlarda da gosterilir, yani nota C# 61, nota D 62...

Bir notayi, bir oktav yukari tasimak icin, pitch degerine 12 eklenir. MIDI kullanarak, oktav tasima,\
mevcut degeri sabit sayilar ekleyip cikarma yaparak kolay bir sekilde yapilir.

0'dan 127'e kadar giden MIDI notalarinin araliklarinin nasi degistigi konusunda dikkat olmak gerekir.\
Ornegin; degeri 96 olan bir notaya 4 oktav (+48) ekleyerek, toplamda 144'e ulasilir ki bu deger aralik\
disindadir ve 144-128 = 16 olacak sekilde truncate edilir, 16 da cok dusuk bir notaya denk gelecektir.

Velocity degeri, pratikte duyulmayacak notalardan maksimum nota seviyelerine kadar araligi kapsayan\
1'den 127'e kadar bir sayisal araliktadir. Temel olarak, muzik notasyonunda bulunan farkin olcegine\
denk gelir, asagidaki gorselde bu olcek degerleri gosterilmistir:

![*Velocity Values*](https://www.cs.cmu.edu/~music/cmsip/readings/MIDI%20tutorial%20for%20programmers_files/nuance-velocity-table.jpg)

Temel sentezleyicilerde, velocity degeri sadece, calinacak nota icin bir siddet degeri belirler, bu\
siddet notanin daha guclu veya yumusak bir ses efektine sahip olacagini belirtir.

Daha gelismis sentezleyicide, bu deger ayni zamanda sesi kalitesini de belirler. Aslinda, gercek\
pianoda bir notaya daha sert basmak sadece, notanin gurultulu cikmasini saglamaz, bunun yani sira\
*timber* adi verilen sesin kendi kalitesini de saglar. Bu olay, pratikte herhangi bir enstrumanda olan\
durumdur.

Velocity degerinin sifir oldugu ozel bir durum vardir. Velocity degerinin 0 oldugu **NOTE ON** mesaji\
**NOTE OFF** mesaji ile ayni anlama gelir, yani nota calmayi durdurur.

**NOTE OFF** mesaj yapisi su sekildedir:

 1. Status byte : 1000 CCCC
 2. Data byte 1 : 0PPP PPPP
 3. Data byte 2 : 0VVV VVVV
 
"CCCC" ve "PPP PPPP" degerleri **NOTE ON** mesajindaki ile ayni anlamdadir. "VVV VVVV" degeri hizi\
serbest birakma anlamina gelir ve nadiren kullanilir. Default olarak, 0 degerine ayarlanir.

## MIDI Tutorial Part 3 - Playing notes and chords

Bir sentezleyiciye **NOTE ON** mesaji gonderdiginizde, bu nota calmaya baslar. Bu esnada, bir akor\
duymak icin farkli nota pitch degerleri iceren baska **NOTE ON** mesajlari gonderebilirsiniz. Ancak,\
calan notalarin kayitlarini tutmaya ihtiyaciniz olacaktir, bu sayede her notaya denk gelen **NOTE OFF**\
mesajlarini yollayabilirsiniz, aksi takdirde calan notalar sonsuza kadar durmadan calmaya devam\
edeceklerdir.

Bir ornege bakalim. Asagidaki notalari calmak icin gerekli olan MIDI mesajlari nelerdir?

![Example](https://www.cs.cmu.edu/~music/cmsip/readings/MIDI%20tutorial%20for%20programmers_files/Midi-example-1.jpg)

Muzigi duyabilmek icin zaman cizelgesine ihtiyac oldugu icin, kanal 1 uzerinden yukaridaki muzigi\
calabilmek icin sentezleyiciye gonderilecek MIDI mesajlarinin bir zaman siralamasi olmasi gerekir.

- t=0 : 0x90 - 0x40 - 0x40 (Start of E3 note, pitch = 64)
- t=0 : 0x90 - 0x43 - 0x40 (Start of G3 note, pitch= 67)
- t=1 : 0x80 - 0x43 - 0x00 (End of G3 note, pitch=67)
- t=1 : 0x90 - 0x45 - 0x40 (Start of A3 note, pitch=69)
- t=2 : 0x80 - 0x45 - 0x00 (End of A3 note, pitch=69)
- t=2 : 0x80 - 0x40 - 0x00 (End of E3 note, pitch=64)
- t=2 : 0x90 - 0x3C - 0x40 (Start of C3 note, pitch = 60)
- t=2 : 0x90 - 0x47 - 0x40 (Start of B3 note, pitch= 71)
- t=3 : 0x80 - 0x47 - 0x00 (End of B3 note, pitch= 71)
- t=3 : 0x90 - 0x48 - 0x40 (Start of C4 note, pitch= 72)
- t=4 : 0x80 - 0x48 - 0x00 (End of C4 note, pitch= 72)
- t=4 : 0x80 - 0x3C - 0x40 (End of C3 note, pitch = 60)

"t" saniye cinsinden zamani temsil eder. Dakikada 60 vurus ile calinir, bu nedenle her ceyrek nota 1\
saniye surer.
