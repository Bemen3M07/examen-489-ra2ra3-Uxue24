# EXAMEN · MÒDUL 489

## Programació de Dispositius Mòbils i Multimèdia

**Unitats Formatives:** RA2 i RA3  
**Curs:** 2n DAM · Videojocs  
**Data:** 23/03/2026  
**Durada:** 2 hores  

**Alumne/a:** Uxue Esteve  
**Grup:** 2n DAM  

---

## Posada en marxa de l'entorn

Consulta el fitxer **`README.md`** del projecte per a les instruccions completes d'instal·lació i arrencada (Node.js, servidor mock i Flutter).

---

> **Instruccions generals**
>
> - Respon cada pregunta en l'espai indicat (substitueix el text `[Escriu la teva resposta aquí]`).
> - Per a la part de codi, escriu directament en bloc `dart`. No és necessari que el codi compili, però ha de reflectir coneixement real de Flutter/Dart.
> - Tens el codi dels projectes **Cars** i **Phone** com a referència en el teu ordinador. **No pots accedir a internet** durant l'examen.
> - Desa el fitxer i lliura'l amb el nom: `EXAMEN_M489_[el_teu_nom].md`
> - Fes commit i push del .md modificat i de tots els arxius que hagis modificat

---

## BLOC 1 · ARQUITECTURA I CICLE DE VIDA *(RA 2)*

### Pregunta 1.1 – Comunicació entre Widgets *(12 punts)*

Al projecte **Cars**, el widget `CarsPage` gestiona el número de pàgina actual (`_currentPage`) i el passa a `CarsList1`. El widget `ButtonPanel` conté els botons "Anterior" i "Següent".

**a)** A `cars_page.dart`, el widget utilitza el mètode `setState` per gestionar la paginació. Explica:

- Quina és la funció de `setState` i per què cridar-lo fa que la UI es torni a dibuixar.
- Per quin motiu `_loadPage()` fa servir dos crides a `setState` separades (una a l'inici i una al final) en lloc d'una sola.

**Resposta:**

```
setState serveix per dir-li a Flutter que alguna cosa ha canviat en el widget i que cal tornar-lo a dibuixar. Així, la pantalla es refresca amb les noves dades. 
A _loadPage() es fan dues crides a setState perquè passen dues coses diferents:  
Al començar, per indicar que s’estan carregant les dades i mostrar un indicador de càrrega.  
Al final, quan les dades ja han arribat, per actualitzar la llista i amagar l’indicador.  
Si només es fes una crida al final, l’usuari no veuria que la informació s’està carregant, i l’experiència seria pitjor.
```

---

### Pregunta 1.2 – Cicle de vida d'un widget amb recursos *(13 punts)*

Al projecte **Camera**, el widget `CameraScreen` utilitza un `CameraController` per gestionar la càmera del dispositiu. Aquest controlador ocupa recursos del sistema (càmera, memòria) i cal alliberar-los correctament.

**a)** Quin mètode del cicle de vida de `State` s'usa a `CameraScreen` per alliberar el `CameraController` quan el widget és destruït? Escriu com es fa i explica per quina raó és imprescindible cridar-lo.

**Resposta:**

```
@override
void dispose() {
  _cameraController.dispose();
  super.dispose();
}
Es necesario llamar a dispose() para liberar la cámara y la memoria que usa. Si no se hace, la app podría usar recursos de más o bloquear la cámara para otras apps.
```

---

**b)** El `CameraController` s'inicialitza de forma asíncrona a `initState()` i el resultat es guarda a `_initializeControllerFuture`. Respon les preguntes següents:

- Per quin motiu no es pot fer `await` directament a `initState()`?
- Quina millora aporta a l'usuari usar `FutureBuilder` en lloc de bloquejar el fil?
- Com treballen junts `_initializeControllerFuture` i `FutureBuilder`?

**Resposta:**

```
No es pot fer await a initState() perquè aquest mètode ha de ser ràpid i no pot ser asíncron.
S’utilitza FutureBuilder per no bloquejar la pantalla mentre es prepara la càmera. Així es pot mostrar un loading (CircularProgressIndicator) fins que tot estigui llest.
_initializeControllerFuture guarda l’estat de la inicialització. El FutureBuilder mira aquest futur i reconstrueix la UI: mostra el loading mentre espera, la càmera quan acaba i un missatge si hi ha un error.

```

---

## BLOC 2 · COMUNICACIÓ, PERSISTÈNCIA I PROVES *(RA 2 — 35 punts)*

### Pregunta 2.1 – Consum d'API i robustesa *(18 punts)*

Analitza el mètode `getCarsPage(int page, int limit)` de `car_http_service.dart`.

Què passaria si el servidor de l'API trigués 60 segons a respondre? L'aplicació quedaria bloquejada per a l'usuari? Per què? Escriu com implementaries un *timeout* de 10 segons a la petició HTTP.

**Resposta:**

```dart
// Escriu la modificació al getCarsPage aquí:
Future<List<CarsModel>> getCarsPage(int page int limit) Future<List<CarsModel>> getCarsPage(int page int limit) async
  try 
    // Construïm la URI amb els paràmetres de pàgina i límit
    final uri = _buildUri('/v1/cars', {
      'page' page.toString() // Número de pàgina
      'limit' limit.toString() // Màxim de resultats per pàgina
    })
    // Fem la petició HTTP amb un timeout de 10 segons
    final response = await http.get(uri).timeout(Duration(seconds 10))
    // Comprovem si la resposta és correcta
    if response.statusCode = 200
      // Decodifiquem el JSON rebut
      final data = jsonDecode(response.body)
      // Obtenim la llista de cotxes del camp 'data'
      final List carsJson = data['data']
      // Convertim cada mapa en un objecte CarsModel
      return carsJson.map((e) => CarsModel.fromMapToCarObject(e)).toList()
    else 
      // Si el servidor retorna error, llençar excepció
      throw Exception('Error del servidor ${response.statusCode}')
  // Captura de TimeoutException si la petició triga massa
  on TimeoutException
    throw Exception('La petició ha trigat massa')
  // Captura de qualsevol altre error
  catch(e)
    throw Exception('Error en la petició $e')
```
La app no se queda bloqueada porque await deja que otras cosas sigan funcionando mientras se esperan los datos. Si tarda demasiado, usamos un límite de 10 segundos para que aparezca un error y el usuario no quede esperando sin saber qué pasa.
---

### Pregunta 2.2 – Models de dades  *(17 punts)*

Analitza el constructor `factory CarsModel.fromMapToCarObject(Map<String, dynamic> json)` de `car_model.dart`.

**a)** Imagina que l'API retorna per error el camp `year` com a `String` en lloc d'`int` (per exemple, `"2021"` en lloc de `2021`). El codi actual fallaria. Escriu com resoldries el problema.

**Resposta:**

```
year: int.tryParse(json['year'].toString()) ?? 0
  ```
---

**b)** Al fitxer `class_model_test.dart`, el test utilitza un `const jsonString` amb un JSON escrit a mà en lloc de fer una petició real a l'API de RapidAPI. Explica per quin motiu és millor simular el JSON en un test unitari.

**Resposta:**

```
És millor simular el JSON en un test perquè així els tests funcionen sempre, són més ràpids i no depenen que l’API estigui en línia. També permet provar situacions concretes, com errors o dades estranyes, de manera segura.
```

---

## BLOC 3 · IMPLEMENTACIÓ PRÀCTICA *(RA 3 — 30 punts)*

### Exercici – Widget de detall amb dades remotes

Imagina que volem crear una pantalla de detall per a cada cotxe del projecte Cars. Implementa el mètode `build` d'un widget `StatelessWidget` anomenat `CarDetailPage` que compleixi els requisits següents:

1. Rep un paràmetre `final CarsModel car` al constructor.
2. Mostra el **make** i el **model** del cotxe com a títol destacat (`Text` amb estil gran i negreta).
3. Mostra una **icona diferent** depenent del `type` del cotxe:
   - Si el `type` és `'SUV'`, mostra `Icons.directions_car`.
   - Per qualsevol altre tipus, mostra `Icons.car_rental`.
4. Afegeix un botó `ElevatedButton` que, quan es premi, mostri un `SnackBar` amb el text: `"Cotxe seleccionat: [make] [model]"`.

```dart
// Escriu el teu codi aquí:

class CarDetailPage extends StatelessWidget
  // Paràmetre que rep el cotxe a mostrar
  final CarsModel car
  // Constructor amb la clau i el cotxe obligatori
  const CarDetailPage({super.key required this.car})
  @override
  Widget build(context) 
    // Scaffold principal de la pàgina
    return Scaffold(
      // Barra superior amb títol
      appBar AppBar(title Text('Detall del cotxe'))
      // Cos de la pàgina amb padding
      body Padding(
        padding EdgeInsets.all(16)
        child Column(
          crossAxisAlignment CrossAxisAlignment.start
         children [
            // Títol amb marca i model del cotxe
            Text('${car.make} ${car.model}'
              style TextStyle(fontSize 24, fontWeight FontWeight.bold))
            // Espai vertical
            SizedBox(height 16)
            // Icona condicional segons tipus de cotxe
            Icon(car.type = 'SUV' ? Icons.directions_car : Icons.car_rental size 50)
            // Nou espai vertical
         SizedBox(height 16)
         // Botó per seleccionar el cotxe
       ElevatedButton(
          onPressed () {
            // Mostrem un SnackBar amb informació del cotxe seleccionat
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(content Text('Cotxe seleccionat: ${car.make} ${car.model}'))) }
          // Text del botó
          child Text('Seleccionar')),),],),),);}}

```

---

**Ampliació (nivell Expert):** Afegeix un `FutureBuilder` que cridi al mètode `CarHttpService().getCarsPage(1, 5)` i mentre espera les dades mostri un `CircularProgressIndicator`. Quan les dades estiguin llestes, mostra un `ListView.builder` amb el `make` de cada cotxe. Si hi hagués un error, mostra un `Text` en color vermell amb el missatge de l'error.

```dart
//Escriu la teva ampliació aquí:
FutureBuilder<List<CarsModel>>(
  // Futur que obté la pàgina 1 amb 5 cotxes
  future: CarHttpService().getCarsPage(1, 5),
  // Builder que reconstrueix la UI depenent de l'estat del futur
  builder: (BuildContext context, AsyncSnapshot<List<CarsModel>> snapshot) {
    // Si encara no hi ha dades
    if (!snapshot.hasData) {
      // Mostrem indicador de càrrega
      return Center(child: CircularProgressIndicator());}
    // Si hi ha un error
    if (snapshot.hasError) {
      // Mostrem missatge d'error en vermell
      return Text(
        'Ha succeït un error: ${snapshot.error}',
        style: TextStyle(color: Colors.red),);}
    // Guardem les dades retornades pel futur
    final cars = snapshot.data!;
    // Si la llista està buida
    if (cars.isEmpty) {
      // Mostrem missatge indicant que no hi ha cotxes
      return Text('No s’han trobat cotxes');}
    // Mostrem la llista de cotxes
    return ListView(
      // Convertim cada cotxe en un ListTile amb el seu 'make'
      children: cars.map((car) => ListTile(title: Text(car.make))).toList(),);},)
```

---

## BLOC 4 · EXTENSIÓ DEL SERVEI HTTP *(RA 2 — 10 punts)*

### Exercici 4.1 – Mètode parametritzat a `CarHttpService` *(10 punts)*

El servidor mock local té disponible un  endpoint de cerca:

```
GET http://localhost:8080/v1/cars/search?make=Toyota&model=Corolla
```

- El paràmetre `make` filtra per marca (coincidència parcial, insensible a majúscules).
- El paràmetre `model` filtra per model (coincidència parcial, insensible a majúscules).
- Tots dos paràmetres són opcionals: si no s'envien, retorna tots els cotxes.

Exemples vàlids:

- `/v1/cars/search?make=Toyota` → tots els Toyota
- `/v1/cars/search?model=X5` → tots els cotxes amb "X5" al model
- `/v1/cars/search?make=BMW&model=X` → BMW amb "X" al model

**Implementa** el mètode `getCarsByFilter` a la classe `CarHttpService` existent, seguint el mateix patrons que `getCarsPage`:

```dart
// Afegeix aquest mètode a car_http_service.dart:
Future<List<CarsModel>> getCarsByFilter({String? make, String? model}) async
  try 
    // Creem un mapa buit per guardar els paràmetres de la URL
    final params = <String, String>{}
    // Afegim el paràmetre 'make' si no és nul ni buit
    if make != null && make.isNotEmpty params['make'] = make
    // Afegim el paràmetre 'model' si no és nul ni buit
    if model != null && model.isNotEmpty params['model'] = model
    // Construïm la URI amb el path i els paràmetres
    final uri = _buildUri('/v1/cars/search', params)
    // Fem la petició HTTP GET amb un timeout de 10 segons
    final response = await http.get(uri).timeout(Duration(seconds: 10)
    // Comprovem si la resposta del servidor és correcta
    if response.statusCode == 200
      // Decodifiquem el JSON i extraiem la llista de cotxes
      final data = jsonDecode(response.body)['data'] as List
      // Convertim cada element a un objecte CarsModel
      return data.map((c) => CarsModel.fromMapToCarObject(c)).toList()
    else
      // Llancem una excepció si el servidor retorna un codi diferent de 200
      throw Exception('Error del servidor (${response.statusCode}')
  // Capturem el TimeoutException si la petició triga massa
  on TimeoutException
    throw Exception('La petició ha trigat massa')
  // Capturem qualsevol altre error inesperat
  catch e
    throw Exception('Ha succeït un error: $e')
```

Requisits:

1. Utilitza el mètode privat `_buildUri(String path, Map<String, String> queryParams)` ja existent.
2. Només afegeix els paràmetres `make` i/o `model` al mapa si el valor no és `null` ni buit (`isEmpty`).
3. Gestiona errors i timeout amb el mateix mecanisme que `getCarsPage`.

**Resposta:**

```dart
// Escriu aquí la teva implementació completa del mètode:

FFuture<List<CarsModel>> getCarsByFilter({
  String? make,
  String? model,
}) async {
  try
    // Creem un mapa buit per als paràmetres de la URL
    final params = <String, String>{}
    // Si make té valor, l’afegim als paràmetres
    if make?.isNotEmpty ?? false params['make'] = make
    // Si model té valor, l’afegim als paràmetres
    if model?.isNotEmpty ?? false params['model'] = model
    // Construïm la URI completa amb els paràmetres
    final uri = _buildUri('/v1/cars/search', params)
    // Fem la petició GET amb un timeout de 10 segons
    final response = await http.get(uri).timeout(Duration(seconds: 10)
    // Si la resposta és correcta (codi 200)
    if response.statusCode == 200 {
      // Decodifiquem el JSON i extraiem la llista de cotxes
      final carsData = jsonDecode(response.body)['data'] as List
      // Convertim cada element a un objecte CarsModel
      return carsData.map((c) => CarsModel.fromMapToCarObject(c)).toList()
    } else
      // Llancem excepció si el servidor retorna un altre codi
      throw Exception('Servidor va respondre amb codi ${response.statusCode}')
  // Capturem el timeout i llancem excepció personalitzada
  on TimeoutException 
    throw Exception('La petició ha trigat massa timeout')
  // Capturem qualsevol altre error
  catch e {
    throw Exception('Ha succeït un error: $e')}
```

---

## Resum de l'examen

| Bloc | RA | Punts màxims |
|------|----|:------------:|
| Bloc 1 – Arquitectura i Cicle de vida | RA 2 | 25 |
| Bloc 2 – Comunicació, Persistència i Proves | RA 2  | 35 |
| Bloc 3 – `CarDetailPage` (base) | RA 3 | 20 |
| Bloc 3 – Ampliació `FutureBuilder`  | RA 3 | 10 |
| Bloc 4 – Extensió del servei HTTP | RA 2 | 10 |
| **TOTAL** | | **100** |

---
