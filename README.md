**! Dôležité !**
Pre spustenie je potrebné spustiť program index.php v priečinku public

```sh
    cd public
    php -S localhost:8000
```

# Finax

Finax je webová aplikácia navrhnutá na správu a sledovanie finančných aktív. Umožňuje používateľom prihlásiť sa, pridávať, upravovať a mazať aktíva a zobrazovať celkovú hodnotu aktív podľa kategórie.

## Funkcie

- Autentikácia používateľov (prihlásenie/odhlásenie)
- Pridávanie, úprava a mazanie finančných aktív
- Zobrazenie celkovej hodnoty aktív podľa kategórie
- Responzívny dizajn pre mobilné zariadenia a počítače

## Spustenie programu

Pred spustením je potrebné:

1. Mať nainštalované

- PHP 7.4 alebo vyšší
- Webový server (napr. Apache, Nginx)

2. Mať správne nastavenú cestu pre php v environment variables

**Pre spustenie je potrebné spustiť program index.php v priečinku public**

```sh
    cd public
    php -S localhost:8000
```

Otvorte prehliadač a prejdite na `http://localhost:8000`

## Štruktúra programu

Projekt je organizovaný nasledovne:

```
finax/
├── config/
│   ├── config.php                      # Konfigurácia konštánt
├── data/                               # Automaticky vygenerované na štarte
│   ├── assets.json                     # Uložené dáta o aktívoch
│   ├── users.json                      # Uložené dáta o používateloch
├── public/
│   ├── index.php                       # Hlavný vstupný bod aplikácie
│   ├── routes.php                      # Správa ciest
│   ├── scripts/
│   │   ├── formDataManager.js          # Ukladanie dát formulárov, hide/show heslo
│   │   ├── formUtils.js                # Pomocné funkcie pre formuláre
│   │   ├── formValidation.js           # Overenie údajov formulárov
│   │   └── profileScripts.js           # Funkcie využité v karte profil
│   ├── styles/
│   │   ├── mediaQuery.css              # Štýly pre responzivitu
│   │   ├── nav-footer.css              # Štýly nav a footeru
│   │   ├── mormalize.css               # Štýly pre resetovanie predvolených štýlov
│   │   └── styles.css                  # Štýlovanie stránky
├── src/
│   ├── router.php                      # Deklarácia routeru
│   ├── controllers/
│   │   ├── assetsController.php        # Kontrolér pre správu aktív
│   │   ├── authController.php          # Kontrolér pre karty s autentifikáciu
│   │   ├── errorController.php         # Kontrolér pre správu chýb
│   │   ├── pageController.php          # Kontrolér pre správu kariet
│   │   └── sessionController.php       # Kontrolér pre správu session
│   ├── models/
│   │   ├── assetModel.php              # Model pre správu aktív
│   │   └── userModel.php               # Model pre správu používateľov
│   ├── views/                          # Html componentov a kariet
│   │   ├── components/
│   │   │   ├── footer.php
│   │   │   ├── header.php
│   │   │   └── navbar.php
│   │   ├── pages/
│   │   │   ├── error.php
│   │   │   ├── home.php
│   │   │   ├── login.php
│   │   │   ├── profile.php
│   │   │   └── register.php
├── README.md
```

## Vysvetlenie štruktúry aplikácie a jej častí

### Spracovanie requestu od klienta

1. Klient posiela request na hlavný vstupný bod aplikácie = index.php
2. Index.php inicialize konštanty v config

```php
// index.php

require_once '../config/config.php';
```

3. Globálne inicializuje objekt Router aby bol k nemu jednoduchý prístup neskôr (pri redirectoch)

```php
require_once BASE_DIR . '/src/Router.php';

global $router;
$router = new Router();
```

4. Inicializuje routes.php, ktoré pridá validné cesty do routeru s im prideleným kontrolerom

```php
require_once BASE_DIR . '/public/routes.php';
```

5. Router prepošle request do príslušného kontróleru ktorý ho spracuje, stiahne potrebné dáta z databázy (pomocou Modelov - o tomto viac v ďalšej sekcii), pomocou Views vygeneruje html stránky a pošle ako response klientovi späť

### Kontroller, Model, View štruktúra

Jadro aplikácie sa skladá z 3 častí:

- Controllers
- Models
- Views

### Prečo sa používa štruktúra MVC

1. **Oddelenie zodpovedností**:

   - **Model**: Spravuje dáta a obchodnú logiku aplikácie. Priamo spracováva ukladanie, načítanie a manipuláciu s dátami.
   - **View**: Spravuje prezentačnú vrstvu. Je zodpovedný za zobrazovanie dát poskytnutých modelom v užívateľsky prívetivom formáte.
   - **Controller**: Pôsobí ako sprostredkovateľ medzi Modelom a View. Spracováva prichádzajúce požiadavky, manipuluje s dátami pomocou Modelu a aktualizuje View.

2. **Udržiavateľnosť**:

   - Oddelením aplikácie do troch prepojených komponentov je jednoduchšie spravovať a udržiavať kód. Každý komponent môže byť vyvíjaný, testovaný a ladený nezávisle.

3. **Škálovateľnosť**:

   - Štruktúra MVC umožňuje škálovateľný vývoj. Ako aplikácia rastie, nové funkcie môžu byť pridané s minimálnym dopadom na existujúci kód.

4. **Znovupoužiteľnosť**:

   - Komponenty v štruktúre MVC môžu byť znovu použité v rôznych častiach aplikácie. Napríklad rovnaký Model môže byť použitý rôznymi Kontrolermi a View.

5. **Testovateľnosť**:

   - Oddelenie zodpovedností uľahčuje písanie jednotkových testov pre každý komponent. Modely môžu byť testované na dátovú logiku, View na prezentáciu a Kontrolery na spracovanie požiadaviek.

6. **Paralelný vývoj**:
   - Rôzni vývojári môžu pracovať na rôznych komponentoch súčasne. Napríklad jeden vývojár môže pracovať na Modeli, zatiaľ čo iný pracuje na View a tretí na Kontroleri.

### Controllers

- V tejto aplikácii je kotrolér rozdelený na viacej
- Ukážka funkcie AssetsController

Validácia sesiion (viac o session v neskorsej sekcii)

```php
global $router;
$this->sessionController->resetSessionTimeout();
if (!$this->sessionController->isLoggedIn()) {
    $router->redirect('/login');
}
```

Inicializácia premenných

```php
$error = null;
$success = null;
```

Spracovanie POST requestu

```php
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    try {
        // ...
```

Spracovanie a validácia dát klienta

```php
$owner = $this->sessionController->getEmail();
$category = trim($_POST['category']);
$name = trim($_POST['name']);
$value = trim($_POST['value']);

if (empty($category)) {
    throw new Exception('Kategória nemôže byť prázdna. Tento údaj je povinný.');
}

if (empty($name)) {
    throw new Exception('Názov nemôže byť prázdny. Tento údaj je povinný.');
}

if (empty($value)) {
    throw new Exception('Hodnota nemôže byť prázdna. Tento údaj je povinný.');
}

if (!is_numeric($value)) {
    throw new Exception('Hodnota musí byť číslo.');
}

if ($value <= 0) {
    throw new Exception('Hodnota musí byť väčšia ako 0.');
}
```

Dezinfekcia vstupu

- je nevyhnutná pre zachovanie bezpečnosti, integrity a funkčnosti webových aplikácií. Vstup používateľa môže pochádzať z rôznych zdrojov (formuláre, parametre dopytu, súbory cookie atď.) a ak nie je správne vyčistený, môže viesť bezpečnostným zraniteľnostiam a chybám.

```php
$category = filter_var($category, FILTER_SANITIZE_SPECIAL_CHARS);
$name = filter_var($name, FILTER_SANITIZE_SPECIAL_CHARS);
$value = filter_var($value, FILTER_SANITIZE_NUMBER_INT);
```

Posunutie dát do Model ktorý pridá aktíva do databázy.. alebo teda JSON súboru.

```php
$this->assetModel->addAsset($owner, $category, $name, $value);
```

Spracovanie výnimiek

```php
} catch (Exception $e) {
    $error = $e->getMessage();
}
```

Presmerovanie s chybovými a úspešnými správami

```php
$error = $error ?? '';
$success = $success ?? '';

$router->redirect('/profile?error=' . urlencode($error) . '&success=' . urlencode($success));
```

Podobným štýlom fungujú aj ostatné funkcie v kontróleroch.

### Sessions

- Niektoré údaje na stránke nemôžu byť verejne dostupné (napr. osobné údaje používateľov)
- Môžu sa k nim dostať iba autorizovaní používatelia
- Preto je potrebný kľúč = heslo aby sa k týmto údajom dostal
- Avšak keby musí pri každom requestu používatel napísať svoje heslo bolo by to nepraktické a tak je potrebné si zapamätať prihlásenie používateľa (vytvoriť session)
- Údaje klienta môžemu ukladať na serveri alebo u klienta v prehliadači (cookies, local storage, ...)
- kľúč je potrebné uložiť u používateľa aby ho nemusel znovu a znozu písať ale uložíme vygenerovaný kľúč u používateľa v cookies aby sa pri každom requeste poslal
- Na serveri ho treba overiť a ak je platný tak poskutnúť dáta používateľa

- Sessions v tejto aplikácii sú spravované objektom SessionController
- Session v tejto aplikácii má životnosť 15 min (nastavené v /config/config.php) - pre bezpečnosť
- Pri každom requeste sa skontroluje či je session validná a ak nie používateľ je odhlásený. V opačnom prípade sa obnoví životnosť

### Models

- Obsahujú metódy na zapisovanie a čítanie dát z databáz
- Rozdelené do 2 objektov na spravovanie dát aktív a používateľov
- Pri inicializácii týchto objektov sa JSON súbor vytvorý ak vytvorený nie je

### Views

- Viuzálna časť aplikácie
- obsahuje html ktoré sa vygeneruje a pošle klientovi
- Opakované html je vytvorené do komponentov (priečinok components)
- css, js je v /public a nie vo /views aby sa k tymto assetom dalo dostať

### Public

- obsahuje css a js pre stránku

---

kód je aj pokomentovaný čiže viac info tam
