{%include styles.html%}

<br/>
<span style="background-color: LightYellow;">ПРИМЕЧАНИЕ: Информация на этой странице является [информативным содержанием](https://hl7.org/fhir/versions.html#std-process).</span>
<br/>

FHIR Shorthand (FSH) — это доменно-специфический язык для определения артефактов FHIR, используемых при создании руководств по внедрению FHIR (IG). Язык специально разработан для этой цели, он прост и компактен, и позволяет автору выражать свои намерения с меньшим беспокойством о базовой механике FHIR. FSH можно создавать и обновлять с помощью любого текстового редактора, и поскольку это текст, он позволяет вести распределенную, командную разработку с использованием инструментов управления исходным кодом, таких как GitHub.

### Основы языка FHIR Shorthand

Полный язык FSH формально описан в [Справочнике по языку FHIR Shorthand](reference.html). Здесь мы представляем лишь основы, чтобы познакомиться с FSH.

* **Грамматика**: [FSH имеет формальную грамматику](https://github.com/FHIR/sushi/tree/v3.8.0/antlr/src/main/antlr), определенную в [ANTLR](https://www.antlr.org/).
* **Типы данных**: Примитивные и сложные типы данных и форматы значений в FSH идентичны примитивным типам и форматам значений в [FHIR R4](https://hl7.org/fhir/R4/datatypes.html#2.24.0) и [FHIR R5](https://hl7.org/fhir/R5/datatypes.html#2.1.28.0). Доступные для использования типы при создании зависят от версии FHIR.
* **Пробельные символы**: Повторяющиеся пробельные символы имеют значение в файлах FSH только внутри строковых литералов и при использовании для [отступов в правилах](reference.html#indented-rules). Во всех остальных контекстах повторяющиеся пробельные символы не имеют значения.
* **Комментарии**: FSH использует `//` как начальный разделитель для однострочных комментариев и пару `/*` `*/` для разделения многострочных комментариев.
* **Символ звездочки**: Начальная звездочка используется для обозначения правил FSH. Например, вот правило для установки элемента с именем `active` в значение `true`:

  ```
  * active = true
  ```

* **Символ экранирования**: FSH использует обратную косую черту как символ экранирования в строковых литералах. Например, используйте `\"` для встраивания кавычки в строку.
* **Символ крышки**: FSH использует [синтаксис крышки](reference.html#caret-paths) для прямой ссылки на структуру определения, связанную с элементом. При определении профиля, символ крышки `^` позволяет ссылаться на элементы в StructureDefinition. Например, для установки элемента `StructureDefinition.experimental`:

  ```
  * ^experimental = false
  ```

* **Псевдонимы**: Для улучшения читаемости FSH позволяет пользователю определять псевдонимы для URL и идентификаторов объектов (OID). После определения, в любом месте проекта FSH псевдоним может использоваться в большинстве мест, где требуется или принимается URL или OID. См. [Определение псевдонимов](reference.html#defining-aliases) для подробностей. По соглашению псевдонимы часто начинаются с символа `$`, например:

  ```
  Alias: $SCT = http://snomed.info/sct
  ```

* **Типы кодированных данных**: Начальный знак решетки (`#`) используется в FSH для обозначения кода, взятого из формальной терминологии. FSH предоставляет специальную грамматику для выражения типов кодированных данных FHIR (code, Coding и CodeableConcept), которая объединяет систему кодов, код и (опционально) отображаемый текст. Вот код SNOMED-CT в этом синтаксисе, использующий ранее определенный псевдоним:

  ```
  $SCT#363346000 "Malignant neoplastic disease (disorder)"
  ```

### Определение элементов в FSH

Элементы FSH представляют артефакты FHIR, такие как профили, наборы значений и расширения. Они определяются в трех частях: (1) объявление, (2) набор ключевых слов и (3) набор правил.

#### Объявления

Объявления представляют и именуют новые элементы FSH. Объявления всегда являются первым оператором в элементе FSH. В FSH есть [одиннадцать объявлений](reference.html#declaration-statements). Часто используемые объявления включают `Profile`, `Extension`, `ValueSet` и `Instance`. Вот два примера операторов объявления:

```
Profile: CancerDiseaseStatus
```

```
ValueSet: ConditionStatusTrendVS
```

#### Ключевые слова

После объявления каждый элемент FSH имеет набор обязательных и необязательных ключевых слов, как подробно описано в [Справочнике по языку FSH](reference.html#keyword-statements). Этот пример использует ключевые слова `Parent`, `Id`, `Title` и `Description` после объявления `Profile`:

```
Profile: CancerDiseaseStatus
Parent:  Observation
Id:      mcode-cancer-disease-status
Title:   "Cancer Disease Status"
Description: "A clinician's qualitative judgment on the current trend of the cancer, e.g., whether it is stable, worsening (progressing), or improving (responding)."
```

#### Правила

Раздел ключевых слов сопровождается рядом правил. Правила являются механизмом для ограничения профиля, определения расширения, создания срезов и многого другого. Все правила FSH начинаются со звездочки. Вот неполное резюме некоторых из наиболее важных правил в FSH:

* **Правила присваивания** используются для установки фиксированных значений в экземплярах и обязательных шаблонов в профилях. Например:

  ```
  * bodySite.text = "Left ventricle"
  ```

  ```
  * onsetDateTime = "2019-04-02"
  ```

  ```
  * status = #arrived
  ```

  ```
  * valueQuantity = UCUM#mm "millimeters"
  ```

* **Правила привязки** используются для элементов с кодированными значениями для указания набора перечисленных значений для этого элемента. Правила привязки включают [одну из сил привязки FHIR](https://hl7.org/fhir/R5/valueset-binding-strength.html): `example`, `preferred`, `extensible` или `required`. Например:

  ```
  * gender from http://hl7.org/fhir/ValueSet/administrative-gender (required)
  ```

  ```
  * address.state from USPSTwoLetterAlphabeticCodes (extensible)  // USPSTwoLetterAlphabeticCodes это набор значений, определенный в US Core
  ```

* **Правила кардинальности** ограничивают количество вхождений элемента, либо как верхние, так и нижние границы, либо только верхние или нижние границы. Например:

  ```
  * note 1..1
  ```

  ```
  * note 1..
  ```

  ```
  * note ..1
  ```

* **Правила содержания** используются для нарезки и расширений. Оба случая включают указание типа элементов, которые могут появляться в массивах.

  Следующее правило нарезает `Observation.component` на два компонента артериального давления:

  ```
  * component contains systolicBP 1..1 and diastolicBP 1..1
  ```

  Синтаксис для расширений аналогичен, за исключением модифицированного синтаксиса, который присваивает локальное имя расширению:

  ```
  // Добавление стандартных расширений FHIR в профиль AllergyIntolerance:

  * extension contains
      allergyintolerance-certainty named substanceCertainty 0..1 and
      allergyintolerance-resolutionAge named resolutionAge 0..1
  ```

* **Правила флагов** добавляют биты информации об элементах, влияющие на то, как разработчики должны с ними обращаться. Флаги - это те, которые [определены в FHIR](https://hl7.org/fhir/R5/formats.html#table), за исключением того, что FSH использует `MS` для must-support и `SU` для summary. Например:

  ```
  * communication MS SU
  ```

  ```
  * identifier and identifier.system and identifier.value MS
  ```

* **Правила типов** ограничивают тип значения, которое может быть присвоено элементу. Например:

  ```
  * value[x] only CodeableConcept
  ```

  ```
  * onset[x] only Period or Range
  ```

  ```
  * recorder only Reference(Practitioner)
  ```

  ```
  * recorder only Reference(Practitioner or PractitionerRole)
  ```

* **Правила наборов значений** используются для заполнения наборов значений. Эти правила могут быть определены двумя способами:

  [Экстенсиональные](https://blog.healthlanguage.com/the-difference-between-intensional-and-extensional-value-sets) правила перечисляют отдельные коды для включения и/или исключения, например:

  ```
  * include $SCT#54102005 "G1 grade (finding)"
  ```

  ```
  * exclude $SCT#12619005 "Tumor grade GX"
  ```

  [Интенсиональные](https://blog.healthlanguage.com/the-difference-between-intensional-and-extensional-value-sets) правила определяют содержимое набора значений косвенно, например:

  ```
  * include codes from system http://www.nlm.nih.gov/research/umls/rxnorm
  ```

  ```
  * include codes from valueset ConditionStatusTrendVS
  ```

  ```
  * include codes from system $SCT where concept is-a #123037004 "BodyStructure"
  ```

  ```
  * exclude codes from valueset EndStageRenalDiseaseVS
  ```

### Пошаговый разбор

В этом разделе мы пройдем через реалистичный пример FSH, строка за строкой. Этот пример не показывает все возможности языка FSH.

```
1   Alias: $LNC = http://loinc.org
2   Alias: $SCT = http://snomed.info/sct
3
4   Profile:  CancerDiseaseStatus
5   Parent:   Observation
6   Id:       mcode-cancer-disease-status
7   Title:    "Cancer Disease Status"
8   Description: "A clinician's qualitative judgment on the current trend of the cancer, e.g., whether it is stable, worsening (progressing), or improving (responding)."
9   * ^status = #draft
10  * extension contains EvidenceType named evidenceType 0..*
11  * extension[evidenceType].valueCodeableConcept from CancerDiseaseStatusEvidenceTypeVS (required)
12  * status and code and subject and effective[x] and valueCodeableConcept MS
13  * bodySite 0..0
14  * specimen 0..0
15  * device 0..0
16  * referenceRange 0..0
17  * hasMember 0..0
18  * component 0..0
19  * interpretation 0..1
20  * subject 1..1
21  * basedOn only Reference(ServiceRequest or MedicationRequest)
22  * partOf only Reference(MedicationAdministration or MedicationStatement or Procedure)
23  * code = $LNC#88040-1
24  * subject only Reference(CancerPatient)
25  * focus only Reference(CancerCondition)
26  * performer only Reference(http://hl7.org/fhir/us/core/StructureDefinition/us-core-practitioner)
27  * effective[x] only dateTime or Period
28  * value[x] only CodeableConcept
29  * value[x] from ConditionStatusTrendVS (required)
30
31  Extension: EvidenceType
32  Id:  mcode-evidence-type
33  Title: "Evidence Type"
34  Description: "Categorization of the kind of evidence used as input to the clinical judgment."
35  * value[x] only CodeableConcept
36
37  ValueSet:   ConditionStatusTrendVS
38  Id: mcode-condition-status-trend-vs
39  Title: "Condition Status Trend Value Set"
40  Description:  "How patient's given disease, condition, or ability is trending."
41  * $SCT#260415000 "Not detected (qualifier)"
42  * $SCT#268910001 "Patient condition improved (finding)"
43  * $SCT#359746009 "Patient's condition stable (finding)"
44  * $SCT#271299001 "Patient's condition worsened (finding)"
45  * $SCT#709137006 "Patient condition undetermined (finding)"
46
47  ValueSet: CancerDiseaseStatusEvidenceTypeVS
48  Id: mcode-cancer-disease-status-evidence-type-vs
49  Title: "Cancer Disease Status Evidence Type Value Set"
50  Description:  "The type of evidence backing up the clinical determination of cancer progression."
51  * $SCT#363679005 "Imaging (procedure)"
52  * $SCT#252416005 "Histopathology test (procedure)"
53  * $SCT#711015009 "Assessment of symptom control (procedure)"
54  * $SCT#5880005   "Physical examination procedure (procedure)"
55  * $SCT#386344002 "Laboratory data interpretation (procedure)"
```

* Строки 1 и 2 определяют псевдонимы для систем кодов LOINC и SNOMED-CT.
* Строка 4 объявляет намерение создать профиль с именем `CancerDiseaseStatus`. Имя обычно в стиле PascalCase (также известном как UpperCamelCase) и согласно FHIR должно быть "[пригодным для использования приложениями машинной обработки, такими как генерация кода](https://hl7.org/fhir/structuredefinition.html#resource)".
* Строка 5 говорит, что этот профиль будет основан на Observation.
* Строка 6 дает идентификатор для этого профиля. Идентификатор используется для создания глобально уникального URL для профиля. URL состоит из канонического URL IG, типа экземпляра (всегда `StructureDefinition` для профилей) и id профиля.
* Строка 7 - это человекочитаемое название профиля.
* Строка 8 - это описание, которое появится в IG на странице профиля.
* Строка 9 - это начало раздела правил профиля. Тут используется [синтаксис крышки](reference.html#caret-paths) для установки атрибута `status` в StructureDefinition, созданном из этого профиля.
* Строка 10 добавляет расширение к профилю, используя автономное расширение `EvidenceType`, дает ему локальное имя `evidenceType` и назначает кардинальность 0..*. _EvidenceType определен в строке 31._
* Строка 11 привязывает `valueCodeableConcept` расширения `evidenceType` к набору значений с именем `CancerDiseaseStatusEvidenceTypeVS` с обязательной силой привязки. _CancerDiseaseStatusEvidenceTypeVS определен в строке 47._
* Строка 12 обозначает список элементов (унаследованных от Observation) как must-support.
* Строки 13-20 ограничивают кардинальность некоторых унаследованных элементов. FSH не поддерживает установку кардинальности нескольких элементов одновременно, поэтому это должны быть отдельные операторы.
* Строки 21 и 22 ограничивают выбор типов ресурсов для двух элементов, которые ссылаются на другие ресурсы.
* Строка 23 фиксирует значение атрибута code на конкретный код LOINC, используя псевдоним для системы кодов, определенной в строке 1.
* Строки 24-25 сокращают унаследованный выбор ссылок на ресурсы до экземпляров, соответствующих конкретным профилям (которые должны быть определены, но являются внешними для этого примера)
* Строка 26 аналогична строкам 24 и 25, но ссылка на внешний профиль.
* Строки 27 и 28 ограничивают тип данных для элементов, которые предлагают выбор типов данных в базовом ресурсе.
* Строка 29 привязывает оставшийся разрешенный тип данных для `value[x]`, CodeableConcept, к набору значений `ConditionStatusTrendVS` с обязательной привязкой. _ConditionStatusTrendVS определен в строке 37._
* Строка 31 объявляет расширение с именем `EvidenceType`.
* Строка 32 назначает id расширению.
* Строка 33 дает расширению человекочитаемое название.
* Строка 34 дает расширению описание, которое появится на главной странице расширения.
* Строка 35 начинает раздел правил для расширения и ограничивает тип данных элемента `value[x]` расширения до CodeableConcept.
* Строка 37 объявляет набор значений с именем `ConditionStatusTrendVS`.
* Строка 38 дает набору значений id.
* Строка 39 предоставляет человекочитаемое название для набора значений.
* Строка 40 дает набору значений описание, которое появится на главной странице набора значений.
* Строки 41-45 определяют коды, которые являются членами набора значений.
* Строки 47-55 создают другой набор значений, `CancerDiseaseStatusEvidenceTypeVS`, похожий на предыдущий.

Несколько замечаний об этом примере:

* Порядок элементов (псевдонимы, профиль, набор значений, расширение) не имеет значения. В FSH вы можете ссылаться на элементы, определенные до или после текущего элемента. По соглашению псевдонимы появляются в начале файла.
* Пример предполагает, что все элементы находятся в одном файле, но они могут быть в отдельных файлах. Распределение элементов по файлам - это выбор автора. Псевдонимы, определенные в одном файле, могут использоваться в других файлах.
* Большинство правил ссылаются на элементы по их именам FHIR, но когда правило ссылается на элемент, который не находится на верхнем уровне, требуются более сложные пути. Пример сложного пути встречается в строке 11, `extension[evidenceType].valueCodeableConcept`. В Справочнике по языку содержатся [дальнейшие описания путей](reference.html#fsh-paths).

### FSH на практике

Этот раздел представляет обзор того, как язык FSH применяется на практике с использованием [SUSHI](https://fshschool.org). [SUSHI](https://fshschool.org/docs/sushi/) (аббревиатура от "**S**USHI **U**nshortens **SH**orthand **I**nputs") - это эталонная реализация и _де-факто_ стандарт для компилятора FSH, который переводит FSH в артефакты FHIR, такие как профили, расширения и наборы значений.

Обсуждение в этом разделе относится к номерам на следующем рисунке:

<img src="Workflow.png" alt="Общий рабочий процесс FSH" width="800px" style="float:none; margin: 0px 0px 0px 0px;" />

#### Установка SUSHI

Процесс установки SUSHI описан [здесь](https://fshschool.org/docs/sushi/installation/). Также необходим текстовый редактор. [Visual Studio Code](https://code.visualstudio.com/) имеет полезный [плагин FSH](https://marketplace.visualstudio.com/items?itemName=MITRE-Health.vscode-language-fsh), который знает синтаксис FSH и соответственно окрашивает текст.

#### Создание нового проекта

Чтобы настроить структуру каталогов для вашего IG, запустите `sushi init` в командной строке. Это создаст [конкретную структуру проекта](https://fshschool.org/docs/sushi/project/), требуемую IG Publisher.

#### Создание файлов FSH

Содержание, написанное на FSH, хранится в простых текстовых файлах (ASCII или UTF-8) с расширением `.fsh` (1). Любой текстовый редактор может использоваться для создания файла FSH. SUSHI позволяет автору решать, как распределить определения FSH по файлам **.fsh**. Вот некоторые возможности:

* Один файл на элемент
* Все определения профилей в одном файле, все определения наборов значений в другом, все расширения в третьем и т.д.
* Группировка связанных элементов в одном файле, например, профиль вместе с его наборами значений, расширениями и примерами
* Создание подкаталогов для каждого типа элемента (профили, расширения, наборы значений) с отдельными файлами для каждого элемента соответствующего типа внутри этих подкаталогов.

#### Запуск SUSHI и/или HL7 IG Publisher

Перед запуском SUSHI у вас должен быть [файл конфигурации с именем **sushi-config.yaml**](https://fshschool.org/docs/sushi/configuration/) (2), содержащий основную информацию о проекте, такую как его канонический URL.

Когда SUSHI запускается (3), он собирает все файлы FSH из подкаталога **input/fsh** данного входного каталога (1) и записывает сгенерированные артефакты FHIR JSON в данный выходной каталог (4). Входной и выходной каталоги появляются как аргументы командной строки SUSHI. Если не указаны, входной каталог по умолчанию будет текущим каталогом, а выходы будут записаны в **./fsh-generated**.

SUSHI может запускаться из командной строки или вызываться как часть [HL7 FHIR IG Publisher](https://confluence.hl7.org/display/FHIR/IG+Publisher+Documentation) (6). Последнее требует дополнительной информации о конфигурации и других данных IG (5). Если в папке **[корень]/input/fsh** проекта нет файлов FSH (1), IG Publisher не будет запускать SUSHI. Сгенерированный вывод SUSHI находится в каталоге **[корень]/fsh-generated** (4), а само руководство по внедрению (7) расположено в **[корень]/output**.

Для получения дополнительной информации об использовании SUSHI и IG Publisher см. [Документацию SUSHI](https://fshschool.org/docs/sushi/).

**[Продолжить к спецификации языка FSH ->](reference.html)**
