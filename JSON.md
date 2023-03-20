# Mis on JSON
JSON (**J**ava**S**cript **O**bject **N**otation) on universaalselt kasutusel olev tekstipõhine andmevahetusformaat. JSON põhineb küll JavaScripti objekt süntaksil kuid on kasutatav kõikjal.

JSON-it kasutatake väga tihti veebirakendustes, näiteks kliendi ja serveri vahel info edastamiseks.

## JSON struktuur
JSON koosneb võtmete ja väärtuste paaridest (nagu Java HashMap ja Pythoni Dictionary). Väärtused võivad olla:
* Numbrid
* Tõeväärtused
* Stringid
* Massiivid
* Teised objektid
* null

Võtmed käivad jutumärkide sisse. Võti ja väärtus on eraldatud kooloniga. Võtme ja väärtuse paare võib olla mitu, eraldatud komaga.

Näide ühest JSON'ist, kus on kasutatud kõiki väärtuse tüüpe:

```json
{
  "Name": "Andres",
  "Adult": true,
  "Courses": ["AKT", "Keeletehnoloogia"],
  "Car": {
    "Color": "yellow",
    "Horsepower": 180
  }
}
```
# GSON
Selle kodutöö piires õpime GSON-i. GSON on Google poolt loodud lihtne teek JSON-i parsimiseks.

## Setup
Impordime dependency Gradle kaudu.

`implementation 'com.google.code.gson:gson:2.10'`

---
Näidetes kasutame lihtsat `Student` klassi, millel on vaid paar atribuuti ja konstruktor:
```java
public class Student {
    String firstname;
    String surname;
    int age;

    public Student(String firstname, String surname, int age) {
        this.firstname = firstname;
        this.surname = surname;
        this.age = age;
    }
}
```
## Serialiseerimine
Saame serialiseerida igasuguseid Java objekte väga lihtsalt `ToJson` meetodiga:
```java 
Gson gson = new Gson();
// Loome uue Student objekti
Student tudeng = new Student("Andres", "Paas", 24);
// Parsime objekti JSON stringiks
String json = gson.toJson(tudeng);
System.out.println(json);
```
*Vastus:*
```json
{"firstname":"Andres", "surname":"Paas", "age":24}
```
### Parseri kohandamine
GSON parseril on mitmeid seadistusi, mida saame muuta. Üldiselt pole seda vaja teha, aga on hea teada, et need on olemas. Selleks kasutame `GsonBuilder` klassi, mille peal saame välja kutsuda erinevaid konfiguratsiooni meetodeid. Kõige viimasena kutsume välja `create` meetodit, mis loob meie GSON instantsi.

Siin näites kasutame nimekonfiguratsiooni meetodit ja "ilusa" printimise meetodit.
```java
Gson gson = new GsonBuilder()
    .setFieldNamingPolicy(FieldNamingPolicy.UPPER_CAMEL_CASE) // Muudab atribuutide nimede esitähed suureks
    .setPrettyPrinting() // Prindib atribuudid välja eraldi ridadel
    .create();
// Loome uue isik objekti
Student tudeng = new Student("Andres", "Paas", 24);
// Parsime objekti JSON stringiks
String json = gson.toJson(tudeng);
System.out.println(json);
```
*Vastus:*
```json
{
  "Firstname": "Andres",
  "Surname": "Paas",
  "Age": 24
}
```
## Deserialiseerimine
Väga tõenäoline on, et kui meil on vaja näiteks mingi REST API kaudu päring teha, siis vastuse saame JSON stringina. Nüüd harjutamegi selle parsimist. Peamiselt on kaks viisi kuidas JSON-i deserialiseerida.

### 1. fromJson meetod otse Java objektiks
Kõige lihtsam meetod, seni kuni JSON-i atribuudid on samade nimedega mis meie Java klassi atribuudid, on lihtsalt kasutada `fromJson` meetodit. See meetod võtab esimeseks argumendiks kas `JsonReader` objekti või lihtsalt stringi milles on JSON. Teine argument on tüüp milleks me tahame parsida, antud juhul `Student`.

#### Loeme JSONi sisse failist
```java
Gson gson = new Gson();
JsonReader reader = new JsonReader(new FileReader("input.json"));
Student tudeng = gson.fromJson(reader, Student.class);
```
`input.json` faili sisu:
```json
{
  "firstname": "Pipi",
  "surname": "Pikksukk",
  "age": 20,
  "courses": ["Sissejuhatus Erialasse", "Programmeerimine", "Matemaatiline maailmapilt"]
}
```

Siinjuures tooks tähelepanu `courses` atribuudi juurde. Seda atribuuti meie `Student` objektis ei leidu, seega parser lihtsalt ignoreerib seda. See on oluline omadus, sest tänu sellele võime parsida väga suuri JSON faile, milles on palju ebaolulisi atribuute, millest siis parsime ainult need mis meile olulised.

**NB:** Kui mõni atribuut on puudu JSON-is siis parsides seda Java objektiks antakse sellele atribuudile lihtsalt `null` väärtus.

#### Annotatsioonid
GSON on küll loodud põhimõttega, et annotatsioone pole meie klassides vaja kasutada. Kuid mõnedel juhtudel on siiski objektide parsimist vaja kohandada ja seega on selleks olemas mitmeid annotatsioone.

Näiteks on olukord, kus meie JSON sisendandmed võivad olla mitmes eri keeles. Eelnevalt sai mainitud, et `fromJson` meetod toimib ainult siis kui klassi atribuudid on samade nimedega mis JSON-i atribuudid. Sellest piirangust on võimalik ümber saada kui kasutame `@SerialisedName` annotatsiooni, et defineerida võimalikud aliased meie atribuutidele.
```java
public class Student {
    @SerializedName(value = "firstname", alternate = {"nimi", "eesnimi"})
    String firstname;
    @SerializedName(value = "surname", alternate = {"perekonnanimi", "perenimi"})
    String surname;
    @SerializedName(value = "age", alternate = "vanus")
    int age;

    public Student(String firstname, String surname, int age) {
        this.firstname = firstname;
        this.surname = surname;
        this.age = age;
    }
}
```
Nüüd saab GSON ka parsida järgnevat JSON stringi ilma probleemideta.
```json
{
  "nimi": "Pipi",
  "perenimi": "Pikksukk",
  "vanus": 20,
}
```
### 2. JSON puu meetod
Kui me ei taha eraldi klassi luua (nagu meie näidetes kasutatud Student klass) selleks, et JSON-ist andmeid kätte saada, siis me võime ka individuaalseid atribuute JSON-ist välja noppida. 
Selleks peame esiteks parsima JSON-i `JsonObject`'iks. 
```java
String jsonString = Files.readString(Paths.get("input.json")); // Input faili sisu sama mis enne
JsonObject jsonTree = JsonParser.parseString(jsonString).getAsJsonObject();
```
Paneme tähele, et tahame individuaalseid atribuute JSON-ist kätte saada, ja selle jaoks on vaja käsitleda `JsonObject` tüüpi, mitte `JsonElement` tüüpi. Kuna `parseString` tagastab`JsonElement` tüübi, siis peame kutsuma sellel veel `.getAsJsonObject` meetodit.

Peale seda on võimalik kõiki atribuute käsitleda lihtsalt nende atribuutide nimede kaudu, kasutades `get` meetodit. Näiteks kui tahame saada tudengi eesnime ja kursuseid kätte, saame seda teha järgnevalt:
```java
String eesnimi = jsonTree.get("firstname").getAsString();
System.out.println(eesnimi);
// Kuna kursused on massiivis, peame neid veidi teistmoodi käsitlema
JsonArray kursused = jsonTree.getAsJsonArray("courses");
for (Object kursus : kursused) {
    System.out.println(kursus.toString());
}
```
Vastus:
```
Pipi
"Sissejuhatus Erialasse"
"Programmeerimine"
"Matemaatiline maailmapilt"
```

## Ülesanded
Ülesannete juures võite kasutada ükskõik millist eelpool näidatud meetoditest. Kui rakendate `fromJson` meetodit siis peate vastava Java klassi ise looma (nagu näidetes kasutatud Student klass). JSON fail on siin (link books.json).

1. Tagastage raamatu "Toomas Nipernaadi" ISBN kood. `public static String task1(String jsonString)`
2. Tagastage kõikide lasteraamatute hindade summa (ümardada pole vaja!). `public static double task2(String jsonString)`
3. Tagastage kui mitu lehekülge pikk on keskmine romaan (leidke aritmeetiline keskmine, ümardatud täisarvuks). `public static int task3(String jsonString)`
4. Tagastage `Set` kõikidest autoritest keda on kirjastanud kirjastus "Sinisukk". `public static Set<String> task4(String jsonString)`
5. Tagastage milline autor on kirjutanud kõige rohkem romaane. `public static String task5(String jsonString)`
6. Looge klass mis implementeerib järgnevat `Literature` interface. Parsige raamat "Kevade" selleks objektiks, ning tagastage see. `public static Literature task6(String jsonString)`
```java
public interface Literature {
    public String getTitle();
    public double getPrice();
    public List<String> getAuthors();
}
```

**NB:** Automaattestide andmed erinevad teile antud andmetest, seega teie lahendus peab töötama ka testandmete peal.
