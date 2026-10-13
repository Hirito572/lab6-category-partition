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

## B. PICT — Pairwise Covering Array

### B1. Browser модель

Browser модель нь 6 параметртэй:

- Content: Allow, Restrict, Block
- Cookies: Allow, Restrict, Block
- Popups: Yes, No
- Addons: Yes, No
- Attacks: Yes, No
- Forgeries: Yes, No

Бүрэн боломжит олонлог: 3 × 3 × 2 × 2 × 2 × 2 = 144.

| Хувилбар | Үүссэн тестийн мөр | Үр дүнгийн файл |
|---|---:|---|
| 2-way | 10 | results/browser-2way.txt |
| 3-way | 20 | results/browser-3way.txt |

Лекцийн гараар боловсруулсан 9 мөртэй харьцуулахад PICT-ийн 2-way гаралт 10 мөр байна.
PICT нь greedy алгоритм ашигладаг тул тухайн алхам бүрд хамрагдаагүй хослолыг үр ашигтай
нэмдэг боловч хамгийн цөөн мөртэй багцыг заавал олохгүй. Иймээс 10 мөр гарсан нь
моделийн алдаа гэсэн үг биш. 3-way хувилбар 20 мөртэй бөгөөд бүрэн 144 хослолын
олонлогоос цөөн тестээр гурвалсан утгуудын хамралтыг хангана.

### B2. Transfer моделиудын харьцуулалт

| Модель | Хязгаарлалт | Тестийн мөр | Үр дүн |
|---|---|---:|---|
| transfer-nc.model | Хязгаарлалтгүй | 13 | results/transfer-nc.txt |
| transfer.model | Хоёр IF нөхцөл | 13 | results/transfer.txt |
| transfer-neg.model | Сөрөг утгуудыг ~ тэмдэглэсэн | 16 | results/transfer-neg.txt |

transfer.model-ийн нөхцөлүүд:
- IF [From] = "Missing" THEN [Amount] = "Normal";
- IF [To] = "Missing" THEN [Amount] = "Normal";

Хязгаарлалттай болон хязгаарлалтгүй хувилбарын мөрийн тоо хоёулаа 13 боловч
тестийн агуулга өөрчлөгдсөн. Сөрөг утгын тэмдэглэгээтэй хувилбар 16 мөр гаргасан.
PICT-ийн ~ тэмдэглэгээ нь нэг мөрөнд олон сөрөг утгыг зэрэг оруулахгүй байхад тусалдаг.
Энэ нь алдааны нөхцөлүүдийг тусад нь шалгаж, error masking-ийг бууруулахад хэрэгтэй.

### B2.1. Бодит тестийн кейсүүд

Доорх нь PICT-ийн гаралтаас сонгосон тестийн кейсийн загвар юм.
Oracle нь лабораторийн тодорхойлолт болон бидний тогтоосон давуу эрэмбэд тулгуурлана.
Бодит API дээр ажиллуулсан үр дүн гэж үзэхгүй.

#### Test case 1 — Happy path

- Модель: transfer-neg.model
- PICT мөр: Valid, Valid, Normal, Same
- Setup: Эх болон хүлээн авах данс байгаа, эх данс царцаагдаагүй.
  Үлдэгдэл 100000, өдрийн лимитийн үлдэгдэл 50000, amount 10000.
  Хоёр дансны валют ижил.
- Input:
  {"from":"ACC001","to":"ACC002","amount":10000,"currency":"MNT"}
- Expected oracle: HTTP 201, {"result":"OK"}

#### Test case 2 — Missing эх данс

- Модель: transfer-neg.model
- PICT мөр: ~Missing, Valid, Normal, Same
- Setup: ACC001 байхгүй, ACC002 хүчинтэй. Amount 10000.
- Input:
  {"from":"UNKNOWN","to":"ACC002","amount":10000,"currency":"MNT"}
- Expected oracle: HTTP 200, {"result":"ERROR_NO_FROM"}

#### Test case 3 — Amount тэг

- Модель: transfer-neg.model
- PICT мөр: Valid, Valid, ~Zero, Same
- Setup: ACC001 болон ACC002 хүчинтэй, эх данс царцаагдаагүй.
- Input:
  {"from":"ACC001","to":"ACC002","amount":0,"currency":"MNT"}
- Expected oracle: HTTP 200, {"result":"ERROR_BAD_AMOUNT"}

### B3. Category-Partition ба PICT-ийн харьцуулалт

Category-Partition арга нь оролтын параметр, төлөөлөх утга, хязгаарлалтыг системтэй тодорхойлоход тохиромжтой.
Энэ арга нь алдааны утга болон хүлээгдэх үр дүнг хүн ойлгомжтойгоор тодорхойлох боломж олгодог.
Гэхдээ бүх хослолыг гараар сонгох нь цаг их шаарддаг.
PICT нь параметрийн хослолуудыг автоматаар үүсгэдэг тул pairwise тест боловсруулах ажлыг хурдасгадаг.
PICT-ийн гаралт нь загварын хязгаарлалтыг дагаж мөрдөхөд тусалдаг.
Сөрөг утгын тэмдэглэгээ нь нэг мөрөнд олон алдаатай утга зэрэг орохоос сэргийлдэг.
Гэсэн хэдий ч PICT нь хамгийн цөөн тестийн мөрийг заавал олдоггүй.
Мөн PICT-ийн гаралт дангаараа хүлээгдэх үр дүнг тодорхойлохгүй тул oracle-ийг хүн боловсруулах шаардлагатай.
Энэ лабораторид хамгийн их бодол шаардсан хэсэг нь олон алдааны давуу эрэмбэ болон утгуудын хязгаарлалтыг зөв тодорхойлох байв.
Иймээс Category-Partition болон PICT-ийг хамтад нь ашиглах нь тестийн хамралт, ойлгомжтой байдлыг сайжруулдаг.
