# Lab 06 — Category-Partition ба PICT

- Оюутны нэр: Э. Мөнх-Очир
- Оюутны код: B232270022
- Хичээл: F.CSA313 — Программ хангамжийн чанарын баталгаа ба тест

## Зорилго

Category-Partition аргаар банкны шилжүүлгийн тестийн спецификаци боловсруулах,
Microsoft PICT ашиглан pairwise тест үүсгэх, гараар боловсруулсан тестүүдтэй
харьцуулах.

## A. Category-Partition

### A1. Сонголт ба төлөөлөх утгууд

| Category | Choice | Тайлбар |
|---|---|---|
| From | Valid | Эх данс байна, царцаагдаагүй |
| From | Frozen [ERROR] | Эх данс царцаагдсан |
| From | Missing [ERROR] | Эх данс байхгүй |
| To | Valid | Хүлээн авах данс байна, эх данснаас өөр |
| To | Missing [ERROR] | Хүлээн авах данс байхгүй |
| To | SameAsFrom [ERROR] | Эх болон хүлээн авах данс ижил |
| Amount | Normal | Эерэг, үлдэгдэл болон лимитэд багтах дүн |
| Amount | Zero [ERROR, SINGLE] | Дүн тэг; бусад amount алдаатай утгатай хослуулахгүй |
| Amount | OverBalance [ERROR, SINGLE] | Дүн хөрвүүлсний дараа үлдэгдлээс хэтэрсэн |
| Amount | OverLimit [ERROR, SINGLE] | Дүн хөрвүүлсний дараа өдрийн лимитээс хэтэрсэн |
| Currency | Same | Дансны валюттай ижил |
| Currency | Different | Дансны валютаас өөр, хөрвүүлэлт хийнэ |

Далд орчны нөхцөлд дансны үлдэгдэл, өдрийн лимитийн үлдсэн хэмжээ,
дансны валют болон ханш орно. Эдгээрийг Amount-ийн төлөөлөх утгуудын
setup-д тодорхойлно.

### A2. Хязгаарлалт ба тооцоо

Олон алдаа зэрэг илэрвэл дараах давуу эрэмбийг хэрэглэнэ:

From → To → Amount → Balance → Daily limit.

Энэ нь тестийн oracle гаргахад зориулсан таамаглал бөгөөд бодит системийн
хэрэгжүүлэлтийн баталгаа биш.

| Хязгаарлалт | Тэмдэглэгээ | Үндэслэл |
|---|---|---|
| From = Missing үед Amount = Normal | IF/property | Эх дансны алдааг тусад нь шалгаж, amount-ийн алдаатай утгаар далдлуулахгүй |
| To = Missing үед Amount = Normal | IF/property | Хүлээн авах дансны алдааг тусад нь шалгана |
| Zero | ERROR, SINGLE | amount <= 0 алдааг дангаар шалгана |
| OverBalance | ERROR, SINGLE | Үлдэгдэл хүрэлцэхгүй алдааг дангаар шалгана |
| OverLimit | ERROR, SINGLE | Өдрийн лимит хэтэрсэн алдааг дангаар шалгана |
| Frozen, Missing, SameAsFrom | ERROR | Дансны төлөв болон дансны холбоотой алдаанууд |

Хязгаарлалтгүй бүрэн олонлог:

N = 3 × 3 × 4 × 2 = 72.

IF нөхцөлүүдийн дараа From болон To-ийн Missing утга байгаа үед Amount
зөвхөн Normal байж болно.

- From = Missing: 3 To × 1 Amount × 2 Currency = 6
- From ≠ Missing, To = Missing: 2 From × 1 To × 1 Amount × 2 Currency = 4
- From ≠ Missing, To ≠ Missing: 2 From × 2 To × 4 Amount × 2 Currency = 32

Иймээс хязгаарлалтын дараах бүрэн олонлог:

72 → 6 + 4 + 32 = 42.

Энэ тооцоо нь дээрх дөрвөн category болон хоёр IF нөхцөлд үндэслэсэн.

### A3. Тестийн спецификаци

| № | From | To | Amount | Currency | Setup | Хүлээгдэх үр дүн |
|---:|---|---|---|---|---|---|
| 1 | Valid | Valid | Normal | Same | Эх данс царцаагдаагүй, үлдэгдэл 100000, лимит хүрэлцээтэй, amount=10000 | HTTP 201, OK |
| 2 | Missing | Valid | Normal | Same | Эх данс үүсгээгүй | HTTP 200, ERROR_NO_FROM |
| 3 | Frozen | Valid | Normal | Same | Эх данс царцаагдсан | HTTP 200, ERROR_FROZEN |
| 4 | Valid | Missing | Normal | Same | Хүлээн авах данс үүсгээгүй | HTTP 200, ERROR_NO_TO |
| 5 | Valid | SameAsFrom | Normal | Same | from болон to ижил дансны ID-тай | HTTP 200, ERROR_SAME_ACCOUNT |
| 6 | Valid | Valid | Zero | Same | amount=0 | HTTP 200, ERROR_BAD_AMOUNT |
| 7 | Valid | Valid | OverBalance | Same | Үлдэгдэл 1000, amount=2000 | HTTP 200, ERROR_INSUFFICIENT |
| 8 | Valid | Valid | OverLimit | Same | Өдрийн лимитийн үлдэгдэл 1000, amount=2000 | HTTP 200, ERROR_LIMIT |
| 9 | Valid | Valid | Normal | Different | Валют өөр, ханш тохируулсан, хөрвүүлсэн дүн хүрэлцээтэй | HTTP 201, OK |
| 10 | Missing | Valid | Zero | Same | Эх данс байхгүй, amount=0 | HTTP 200, ERROR_NO_FROM |
| 11 | Frozen | Missing | Normal | Same | Эх данс царцаагдсан, хүлээн авах данс байхгүй | HTTP 200, ERROR_FROZEN |
| 12 | Valid | Valid | Normal | Same | amount нь дансны үлдэгдэлтэй яг тэнцүү, лимит хүрэлцээтэй | HTTP 201, OK |

Давхар алдааны 10, 11-р тестийн oracle нь тогтоосон давуу эрэмбэд тулгуурласан.
Эдгээр нь бодит API дээр ажиллуулсан үр дүн биш, спецификацийн хүлээгдэх үр дүн юм.
