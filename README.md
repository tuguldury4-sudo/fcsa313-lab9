# Лаборатори №9 — Код хамралт: JaCoCo-гоор тестээ хөтлөх

**Оюутан:**  Я. Төгөлдөр
**Оюутны код:** B242270806
**Хичээл:** F.CSA313 — Программ хангамжийн чанарын баталгаа ба тест (2026)

---

Maven төсөл [starter/](starter/) хавтаст байрлана. Тооцооллын эх кодыг [ScholarshipCalculator.java](starter/src/main/java/mn/edu/must/sqat/ScholarshipCalculator.java), тестүүдийг [ScholarshipCalculatorTest.java](starter/src/test/java/mn/edu/must/sqat/ScholarshipCalculatorTest.java), тохиргоог [pom.xml](starter/pom.xml)-ээс үзнэ.

## 1. Ажлын орчин

A, B, C үе шат болон эцсийн JaCoCo тайланг 2026-11-24-нд доорх орчинд дахин гаргасан. Төсөл Java 17-д хөрвөдөг бөгөөд JUnit Jupiter 5.10.2, JaCoCo 0.8.12 ашигладаг.

```text
$ java -version
openjdk version "25.0.3" 2026-04-21 LTS
OpenJDK Runtime Environment Temurin-25.0.3+9 (build 25.0.3+9-LTS)
OpenJDK 64-Bit Server VM Temurin-25.0.3+9 (build 25.0.3+9-LTS, mixed mode, sharing)

$ mvn -version
WARNING: A restricted method in java.lang.System has been called
WARNING: java.lang.System::load has been called by org.fusesource.jansi.internal.JansiLoader in an unnamed module (file:/C:/Program%20Files/JetBrains/IntelliJ%20IDEA%202025.2.5/plugins/maven/lib/maven3/lib/jansi-2.4.1.jar)
WARNING: Use --enable-native-access=ALL-UNNAMED to avoid a warning for callers in this module
WARNING: Restricted methods will be blocked in a future release unless native access is enabled

Apache Maven 3.9.9 (8e8579a9e76f7d015ee5ec7bfcdc97d260186937)
Maven home: C:\Program Files\JetBrains\IntelliJ IDEA 2025.2.5\plugins\maven\lib\maven3
Java version: 25.0.3, vendor: Eclipse Adoptium, runtime: C:\Program Files\Eclipse Adoptium\jdk-25.0.3.9-hotspot
Default locale: en_US, platform encoding: UTF-8
OS name: "windows 11", version: "10.0", arch: "amd64", family: "windows"
```

Төслийн үндсэн хавтаст `mvn verify` ажиллуулна. Энд ашигласан JDK 25 дээр `mvn "-Djacoco.includes=mn.edu.must.sqat.*" verify` командаар JaCoCo агентыг төслийн багцад чиглүүлсэн. Шинэ локал clone-оос 31 тест амжилттай ажиллаж, мөр болон салааны хамралт 100% гарсан. HTML тайлан `target/site/jacoco/index.html` файлд үүснэ.

| Үе шат | Тест | Maven лог | JaCoCo CSV |
|---|---:|---|---|
| A | 5 | [Лог](starter/results/mvn-verify-A.txt) | [CSV](starter/results/jacoco-A.csv) |
| B | 21 | [Лог](starter/results/mvn-verify-B.txt) | [CSV](starter/results/jacoco-B.csv) |
| C | 24 | [Лог](starter/results/mvn-verify-C.txt) | [CSV](starter/results/jacoco-C.csv) |
| D — засварын өмнө | 31, үүнээс 1 унасан | [Лог](starter/results/mvn-test-D-fail.txt) | — |
| Эцсийн | 31 | [Лог](starter/results/mvn-verify-final.txt) | [CSV](starter/results/jacoco-final.csv) |


## 2. Хэсэг A — Happy path тестүүд ба эхний хэмжилт

5 хэвийн урсгалын тест: `letterGrade_score95_returnsA`, `letterGrade_score85_returnsB`,
`gpa_scores95And85_returns3point5`, `scholarship_gpa39Credits16NoViolation_returns100`,
`scholarship_gpa35Credits12NoViolation_returns70` → `results/mvn-verify-A.txt`:
`Tests run: 5, Failures: 0`, BUILD SUCCESS.

`results/jacoco-A.csv` (ScholarshipCalculator):

| | Missed | Covered | Хамралт |
|---|---|---|---|
| LINE | 10 | 15 | **15/25 = 60.0%** |
| BRANCH | 21 | 20 | **20/41 = 48.8%** |
| INSTRUCTION | 50 | 102 | 102/152 = 67.1% |
| METHOD | 0 | 4 | 4/4 |

Тайлангийн **шар** (хагас хамрагдсан) мөрүүд нь бүгд нөхцөлийн шалгалтууд: `if (score < 0 || score > 100)`
(2/4 салаа дутуу), `if (score >= 80)`, gpa-гийн `scores == null || scores.length == 0`,
`switch` (C, D салаа дутуу), scholarship-ийн эхний `gpa < 0 || gpa > 4 || credits < 0`
(3/6 дутуу), `if (hasViolation)`, хоёр `&&` нөхцөл — өөрөөр хэлбэл зөвхөн "үнэн" эсвэл зөвхөн
"худал" тал нь шалгагдсан. **Улаан** (огт хүрээгүй) мөрүүд нь яг алдааны замууд: гурван
`throw new IllegalArgumentException`, хамгийн доод дүнгүүд (`C`, `D`, `F`) ба тэтгэлгийн 50%,
0% буцаах мөрүүд — happy path тестүүд хүчингүй оролт, муу дүн, бага голчийн замыг огт дайраагүй.

## 3. Хэсэг B — CFG ба branch 100%

![CFG — scholarship()](starter/cfg.png)

`cfg.png` — `scholarship()`-ийн CFG: нөхцөл бүрийг тусдаа шийдвэрийн зангилаа болгосон
(`gpa < 0 || gpa > 4 || credits < 0` → 2, 3, 4-р зангилаа; `&&` бүр → 8–9, 11–12), ирмэг бүрийн
хажууд түүнийг дайрдаг тестийн **бодит метод нэр**. Зургийн байрлал, зангилаа, ирмэгийг SVG-д гараар тодорхойлж PNG болгосон. Ирмэгүүдэд `ScholarshipCalculatorTest.java` дахь бодит тестийн нэрийг ашигласан.

CFG дээрх шийдвэрийн 9 зангилаа тус бүр үнэн/худал хоёр ирмэгтэй тул нийт **18 шийдвэрийн салаа** байна. Шийдвэрийн бус ирмэгүүдийг дараах тестүүд дайрна.

| Ирмэг | Тестийн метод |
|---|---|
| 1 → 2 | `scholarship_gpa39Credits16NoViolation_returns100` |
| 5 → 17 | `scholarship_negativeGpa_throws`, `scholarship_gpaAbove4_throws`, `scholarship_negativeCredits_throws` |
| 7 → 17 | `scholarship_hasViolation_returns0` |
| 10 → 17 | `scholarship_gpa39Credits16NoViolation_returns100` |
| 13 → 17 | `scholarship_gpa35Credits12NoViolation_returns70` |
| 15 → 17 | `scholarship_gpa32Credits10_returns50` |
| 16 → 17 | `scholarship_gpa25Credits20_returns0` |

Хэсэг A-ийн дараа хамрагдаагүй ирмэгүүд (2T, 3T, 4T, 6T, 9F, 12F, 14T, 14F) ба letterGrade/gpa-ийн
улаан мөрүүдэд 16 тест нэмэв:

| Функц | Нэмсэн тест | Хамрах салаа |
|---|---|---|
| letterGrade | `letterGrade_score75_returnsC`, `letterGrade_score65_returnsD`, `letterGrade_score30_returnsF` | C, D, F (`score > 60` худал) |
| letterGrade | `letterGrade_negativeScore_throws`, `letterGrade_scoreAbove100_throws` | `score < 0`, `score > 100` тус бүрийн үнэн (`assertThrows`) |
| gpa | `gpa_nullScores_throws`, `gpa_emptyScores_throws` | `scores == null`, `scores.length == 0` тус бүрийн үнэн |
| gpa | `gpa_scoresCDandF_returns1` | switch-ийн C, D ба **default** (F) салаа |
| scholarship | `scholarship_negativeGpa_throws`, `scholarship_gpaAbove4_throws`, `scholarship_negativeCredits_throws` | эхний `if`-ийн 3 нөхцөл тус бүрийн үнэн — 3 тусдаа `assertThrows` |
| scholarship | `scholarship_hasViolation_returns0` | 6T |
| scholarship | `scholarship_gpa39ButCredits14_returns70` | 9F — голч ≥ 3.8 ч кредит < 15 |
| scholarship | `scholarship_gpa35ButCredits11_returns50` | 12F — голч ≥ 3.4 ч кредит < 12 |
| scholarship | `scholarship_gpa32Credits10_returns50`, `scholarship_gpa25Credits20_returns0` | 11F, 14T, 14F |

**Шаардагдсан тест: 21** (A-ийн 5 + B-ийн 16) — `results/mvn-verify-B.txt`:
`Tests run: 21, Failures: 0, Errors: 0, Skipped: 0`, BUILD SUCCESS.

`results/jacoco-B.csv`: `BRANCH_MISSED = 0`, `BRANCH_COVERED = 41` (**41/41 = 100%**),
`LINE_MISSED = 0`, `LINE_COVERED = 25` (**25/25 = 100%**), INSTRUCTION 152/152.

## 4. Хэсэг C — Давталтын заагийн тестүүд (gpa)

| Давталт | Тест | Шалгалт |
|---|---|---|
| 0 удаа | `gpa_loopZeroIterations_emptyOrNullScores_throws` | хоосон массив **ба** `null` → `assertThrows(IllegalArgumentException)` (`assertAll`-аар хоёуланг) |
| 1 удаа | `gpa_loopOneIteration_singleScore95_returns4` | `gpa([95]) == 4.0` |
| 2+ удаа | `gpa_loopManyIterations_scores95_85_75_65_returns2point5` | `gpa([95, 85, 75, 65]) == 2.5` |

`results/mvn-verify-C.txt`: `Tests run: 24, Failures: 0` (21 + 3), `results/jacoco-C.csv` нь
`jacoco-B.csv`-тэй яг ижил (Branch 41/41, Line 25/25) — хамралт **нэг ч өссөнгүй**.

Хамралт өсөөгүй ч эдгээр тест хэрэгтэй: JaCoCo зөвхөн салаа *дайрагдсан эсэхийг* тоолдог болохоос
давталт 0, 1, олон удаа эргэх үед `total / scores.length` зөв гарах эсэхийг (жишээ нь хуваагчийг
`scores.length - 1` гэж буруу бичих, эсвэл эхний элементийг алгасах off-by-one) шалгадаггүй —
ийм алдаа салааны бүтцийг өөрчлөхгүй тул 100% хамралттай байхад ч амьд үлдэнэ. 0/1/2+ дүрэм нь
давталтын хамгийн эмзэг зааг (хоосон, ганц, олон) бүрд *гаралтын утгыг* шалгаж ийм алдааг барина.

## 5. Хэсэг D — Хамралт 100%, гэхдээ алдаа амьд

Branch 41/41, Line 25/25 байхад letterGrade-д спецификацийн зааг утгуудыг нэг
`@ParameterizedTest`-ээр (`letterGrade_boundaryScores_matchSpec`) нэмэв: 60→D, 59.99→F,
69.99→D, 0→F, 100→A, 89.99→B, 90→A. [Засварын өмнөх Maven лог](starter/results/mvn-test-D-fail.txt) дараах үр дүнг бүртгэсэн.

```
[ERROR] mn.edu.must.sqat.ScholarshipCalculatorTest.letterGrade_boundaryScores_matchSpec(double, String)[1]
org.opentest4j.AssertionFailedError: expected: <D> but was: <F>
[ERROR]   ScholarshipCalculatorTest.letterGrade_boundaryScores_matchSpec:164 expected: <D> but was: <F>
[ERROR] Tests run: 31, Failures: 1, Errors: 0, Skipped: 0
[INFO] BUILD FAILURE
```

Унасан нь зөвхөн `[1]` мөр — `letterGrade(60)`; бусад 6 зааг утга зөв. Харьцуулах операторуудыг
спецификацийн мөр бүртэй тулгахад `60-69.99 -> "D"` гэсэн ч код `if (score > 60) return "D";` —
60 оноог "F" болгож байсан. **Засвар тусдаа commit** (`dc54c56`, *"Fix letterGrade to return D for score 60"*): `score > 60` → `score >= 60`. Дараа нь `results/mvn-verify-final.txt`:
`Tests run: 31, Failures: 0, Errors: 0, Skipped: 0`, BUILD SUCCESS; `results/jacoco-final.csv`:
BRANCH_MISSED = 0 (41/41), LINE_MISSED = 0 (25/25).

**Гол дүгнэлт.** `score > 60` ба `score >= 60` хоёулаа яг ижил хоёр салаа (үнэн → "D", худал → "F")
үүсгэдэг тул Хэсэг B-ийн `letterGrade_score65_returnsD` (үнэн) ба `letterGrade_score30_returnsF`
(худал) тестүүд хоёр салааг хоёуланг нь дайрч JaCoCo 100% гэж харуулсан — алдаатай кодоор ч,
зассан кодоор ч хамралтын тоо яг ижил (jacoco-B.csv = jacoco-final.csv). Хоёр хувилбарын ялгаа
зөвхөн нэг цэг дээр — яг 60 оноо дээр гардаг тул тэр утгыг өгсөн тест л алдааг илрүүлнэ. Өөрөөр
хэлбэл хамралт нь "аль мөр, аль салаа ажилласан бэ"-г хэмждэг болохоос "гарсан үр дүн
спецификацитай таарч байна уу"-г хэмждэггүй; oracle-ийн чанар, зааг утгын сонголтыг огт харуулахгүй.
Хамралт 100% нь "туршаагүй код алга" гэсэн доод баталгаа болохоос "алдаа алга" гэсэн нотолгоо биш —
зааг утгын шинжилгээтэй хослуулж байж утгатай.

## 6. Нэмэлт даалгавар (Сонголт 2) — AI өнцөг

AI-д Хэсэг A-ийн дараах (100%-иас өмнөх) төслийг өгч "хамралтыг 100% болгох тестүүд бич" гэв;
28 тест ([`ai/AiScholarshipCalculatorTest.java.txt`](starter/ai/AiScholarshipCalculatorTest.java.txt)), ажиллуулсан
үр дүн — [`ai/ai-results.md`](starter/ai/ai-results.md). (1) Хамралт: AI-ийн тестүүд дангаараа Line 25/25, Branch
41/41 хүрсэн — миний 21 тесттэй адил. (2) Зааг утгын алдаа: AI спецификацийн тайлбарыг уншаад
`letterGrade_score60_returnsD`, `letterGrade_score59point99_returnsF`, `letterGrade_score90_returnsA`
зэрэг зааг тест бичсэн тул алдаатай код дээр `expected: <D> but was: <F>` гэж **барьсан** — миний
Хэсэг B-ийн тестүүд (65 ба 30-аар салаа хамарсан) үүнийг барьж чадаагүй. Энэ нь "100% хамралт" гэсэн
даалгавар биелэхэд хүрэлцэх тест биш, спецификацийн хил дээр oracle бичсэн тест л алдааг олдгийг дахин
харуулж байна — AI хамралт гэдэг зорилгоос илүү спецификацийг уншсан учраас барьсан. Гэхдээ AI 69.99
(D-ийн дээд зааг) ба давталтын 2+ тохиолдлын утгыг шалгаагүй, `letterGrade_boundaries0And100_valid` нэг
тестэд хоёр зүйл шалгадаг тул унавал алийг нь гэдэг нь тодорхойгүй. Мөн AI өөрөө "тест ажиллуулаагүй"
гэж хэлсэн тул түүний "100%" гэсэн мэдэгдлийг би `mvn verify`-ээр нотолсны дараа л итгэсэн.

## 7. AI туслах ашигласан тухай

Энэ лабын ажлыг AI туслах (Claude Code) ашиглан гүйцэтгэсэн: тестийн кодын төсөл, JaCoCo-г
ажиллуулах, CFG зургийн байршил, README-ийн бичвэр. Бүх тоо (тестийн тоо, хамралтын %), алдааны мөр
нь бодитоор ажиллуулсан `results/` файлуудаас авсан бөгөөд дээрх 6-р хэсгийн AI-ийн тестийг тусад нь
ажиллуулж шалгасан.
