# Stambeni kredit kalkulator – simulacija otplate, prijevremenih uplata i refinanciranja

> ⚠ Ovaj kalkulator i README generirani su uz pomoć LLM asistenta (GPT-5 Thinking).  
> Kod je demo / edukativan i ne predstavlja financijski savjet.

---

## 🏠 Što je ovo?

Jednostrana (single-file) web aplikacija koja:
- modelira stambeni kredit kroz mjesece,
- simulira da svaki mjesec odvajaš određeni iznos novca (`mjesečno odvajanje`),
- svake godine radi prijevremenu uplatu te štednje u glavnicu,
- pokazuje kako to utječe na:
  - ukupno plaćenu kamatu,
  - preostalu glavnicu,
  - trajanje kredita,
  - visinu budućih rata.

Aplikacija ti pomaže odgovoriti na vrlo praktično pitanje:

> "Isplati li mi se godišnje ubacivati dodatnu uplatu u kredit — i je li mi bolje da banka smanji buduću ratu ili da mi skrati rok otplate?"

Sve se računa u pregledniku (čisti HTML/CSS/JS). Nema backend-a, nema slanja podataka.

---

## 🔁 Dva modela otplate koje možeš usporediti

Aplikacija nudi dvije strategije nakon godišnje prijevremene uplate:

### 1. 📉 "Smanji ratu"
- Svakih 12 mjeseci uzme se akumulirana štednja i jednokratno uplati u glavnicu kredita.
- Nakon te uplate banka *smanjuje buduću mjesečnu ratu* (re-amortizacija), a nominalni kraj kredita ostaje otprilike isti.
- Efekt: plaćaš manju mjesečnu ratu kroz ostatak trajanja.

Modelira situaciju "želim rasteretiti mjesečni cash flow".

### 2. ⏳ "Skrati rok"
- Isto: svaka godina štednja ide u glavnicu.
- Ali nakon toga rata ostaje ista kao i prije.
- Zbog toga se kredit otplati brže nego originalno planirano — rok (trajanje u mjesecima/godinama) se skraćuje.
- Efekt: ukupno manje kamata, raniji izlaz iz duga.

Modelira situaciju "želim završiti kredit što ranije, rata mi je ok kakva je".

U sučelju imaš tabove za prebacivanje između ova dva modela.  
Za svaki model dobiješ svoj sažetak i detaljan otplatni plan.

---

## 🧮 Ulazni parametri

U formi na vrhu unosiš:

- **Iznos kredita (€)**  
  Početna glavnica kredita.

- **Godišnja kamatna stopa (%)**  
  Fiksna nominalna kamatna stopa na godišnjoj razini.  
  Pretvaramo je u mjesečnu stopu `r = (godišnja_stopа / 100) / 12`.

- **Rok otplate (godina)**  
  Koliko je kredit inicijalno planiran (npr. 30 godina → 360 mjeseci).

- **Vrsta otplate**
  - `Anuitet`: fiksna mjesečna uplata (anuitet). S vremenom udio kamate pada, udio otplate glavnice raste.
  - `Fiksna glavnica`: svaki mjesec vraćaš isti dio glavnice + kamatu na preostali dug. Rata je veća u početku, pa s vremenom pada.

- **Mjesečno odvajanje (€)**  
  Koliko maksimalno mjesečno možeš izdvojiti za stan:
  - dio ide na stvarnu ratu kredita,
  - ostatak ide u internu "štednju" koja se ne troši odmah,
  - ta štednja se jednom godišnje (svakih 12 mjeseci) odjednom uplati u kredit kao prijevremena otplata.

Drugim riječima: ako ti je rata npr. 700 €, a ti si spreman izdvajati 900 €, tih dodatnih 200 € mjesečno se gomila i jednom godišnje spusti glavnicu.

---

## 📊 Što dobiješ natrag

### 1. Sažetak scenarija
Za aktivni model (Smanji ratu / Skrati rok) prikazuje se:

- **Efektivno trajanje kredita**  
  Koliko mjeseci/godina treba dok glavnica ne padne na 0, nakon svih godišnjih uplata.

- **Početna mjesečna rata (mjesec 1)**  
  Koliko iznosi rata u prvom mjesecu plana.

- **Ukupno plaćena kamata**  
  Zbroj svih mjesečnih kamata koje si platio do potpunog zatvaranja kredita u tom modelu.

- **Prijevremene uplate (1× godišnje)**  
  Koliko € ukupno si uplatio iz štednje direktno u glavnicu kroz godine.

- **Upozorenje ako tvoj `mjesečno odvajanje` < rata**  
  Ako se dogodi da je planirana rata veća nego što si rekao da možeš izdvojiti, kalkulator to označi.  
  Račun i dalje ide dalje (jer kredit *moraš* platiti), ali u tim mjesecima nema dodatne štednje.

### 2. Detaljna tablica po mjesecima
Za svaki mjesec vidiš:
- redovna rata tog mjeseca,
- koliko od toga je kamata, a koliko ide na glavnicu,
- stanje privatne štednje na kraju mjeseca,
- iznos "godišnje uplate na glavnicu" (vidljivo svakih 12 mj.),
- preostala glavnica nakon uplate.

Redovi u kojima se dogodila godišnja prijevremena uplata (lump sum) posebno su vizualno označeni.

---

## 🧠 Kako simulacija radi pod haubom

### Mjesečni koraci
Za svaki mjesec:
1. Izračuna se kamata na preostali dug.
2. Izračuna se redovna uplata tog mjeseca (ovisno o vrsti otplate).
3. Ažurira se preostala glavnica.
4. Izračuna se koliko od tvojeg `mjesečno odvajanje` ostaje nakon plaćanja rate → to ide u virtualnu štednju.

### Svakih 12 mjeseci
- Cijela akumulirana štednja uplati se u kredit kao jednokratna uplata (prijevremena otplata).
- Ostatak štednje se resetira (postaje 0 nakon uplate, osim ako je uplata bila veća od ostatka duga pa nešto ostane).

### Razlika između modela
- **Model "Smanji ratu":**  
  Nakon te godišnje uplate radi se *re-amortizacija*: ponovo se izračuna nova buduća rata tako da se sadašnji ostatak duga raspodijeli preko PREOSTALIH mjeseci do prvotnog kraja.  
  → Rata padne, rok ostaje ~isti.

- **Model "Skrati rok":**  
  Nakon godišnje uplate ne mijenja se rata.  
  → Ako stalno plaćaš istu (relativno veću) ratu na sve manju glavnicu, dug nestane ranije, pa se rok skraćuje.

Kod obje strategije zbrajaju se:
- ukupno plaćena kamata,
- ukupno uplaćeno izvanredno u glavnicu,
- broj mjeseci do potpune otplate.

---

## 🖥️ Tehnička arhitektura

- **Front-end samo**  
  - `index.html` sadrži sve: HTML, CSS (tamni theme), JavaScript logiku.
  - Nije potreban build alat, server, backend.

- **UI elementi**  
  - Forma za ulazne parametre.
  - Dva taba za prebacivanje strategije (📉 smanji ratu / ⏳ skrati rok).
  - Sažetak rezultata i upozorenja.
  - Tablica mjesečnih podataka.

- **State u JS-u**  
  - `activeTab` određuje koji model je trenutno aktivan.
  - `lastResultRatu` i `lastResultRok` drže rezultate simulacije za oba modela.
  - Kad promijeniš tab, UI se rerenda bez ponovnog računanja.

- **Bez vanjskih biblioteka**  
  - Nema frameworka, nema jQueryja, nema grafova zasad.

---

## 🚧 Ograničenja modela

Ovo je edukativni simulacijski alat, ne bankarski izračun.

Konkretno:
- Pretpostavlja fiksnu kamatnu stopu kroz cijeli period.
- Pretpostavlja da banka dopušta godišnje izvanredne uplate bez naknade i bez ograničenja.
- Pretpostavlja idealizirano re-amortiziranje:
  - u modelu "smanji ratu" – rata se odmah prilagodi savršeno matematički nakon uplate,
  - u modelu "skrati rok" – rata ostaje identična, skraćuje se samo rok.
- Ne uračunava poreze, inflaciju, promjene EURIBOR-a, policu osiguranja, trošak života, naknade banke itd.
- Za "fiksna glavnica" model uzima ravnomjerno linearan pad glavnice; neke banke rade malo drugačije zaokruživanje.

Drugim riječima: cilj je razumjeti trendove (koliko rata pada ili koliko se rok skraćuje), a ne dobiti službeni otplatni plan banke.

---

## 🏁 Kako koristiti

1. Otvori HTML datoteku u pregledniku (Chrome, Firefox, Safari…).
2. Unesi:
   - Iznos kredita (npr. 123456),
   - Kamatnu stopu (npr. 3.5),
   - Rok (npr. 30),
   - Tip otplate (anuitet ili fiksna glavnica),
   - Koliko mjesečno možeš stvarno izdvojiti (npr. 800 €).
3. Klikni **"Izračunaj plan"**.
4. Pregledaj:
   - tab **📉 Smanji ratu**,
   - tab **⏳ Skrati rok**.
5. U sažetku vidiš:
   - kolika je ukupna kamata po svakom scenariju,
   - koliko brzo se kredit zatvara u godinama/mjesecima,
   - koliko si ukupno ubacio prijevremeno.
6. U tablici ispod možeš vidjeti ponašanje kredita mjesec po mjesec.

Tipično korištenje:
- promijeni `mjesečno odvajanje` i vidi koliko to zapravo ubrzava otplatu.
- prebaci se između dva modela da vidiš što tebi ima više smisla: rasterećenje rate ili agresivno skraćenje roka.

---

## 📌 Napomena o autorstvu

Ovaj kalkulator i ovaj README generirani su uz pomoć LLM asistenta (GPT-5 Thinking), na temelju korisničkih uputa.  
Kod treba pregledati, validirati i prilagoditi prije stvarne financijske odluke ili pregovora s bankom.

---
