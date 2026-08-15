# MVP

**Cauți o firmă și afli dacă e românească. Atât.**

Fără procente, fără grup, fără producție, fără scor. O întrebare, un răspuns.

## Ce înseamnă „românească" în MVP

**Înregistrată în România**: are CUI, e în registrul fiscal, plătește taxe aici.

Nu înseamnă „deținută de români" — aia e altă întrebare, mult mai greu de răspuns, și rămâne
pentru mai târziu ([plan-aplicatie.md](plan-aplicatie.md)). Aplicația trebuie să spună explicit
la ce răspunde, altfel oamenii vor citi altceva decât scrie.

## De ce tocmai asta

E singura dintre cele patru axe care nu are niciun blocaj:

| | |
| --- | --- |
| **Sursa** | ANAF, serviciu public, gratuit, fără cont |
| **Acoperire** | practic toate firmele din România |
| **Cost** | zero — nimic de cumpărat, niciun furnizor de ales |
| **GDPR** | fără probleme, sunt date de firmă, nu de persoană |
| **Timp** | o noapte de ingest pentru toată țara |

Nicio altă axă nu arată așa. Proprietatea are nevoie de date care nu sunt publice, producția are
nevoie de curare manuală. Asta merge acum.

## Faza 1 — baza de date și răspunsul

- Schema minimă: `companies`, `evidence`.
- Ingest ANAF: nume, CUI, adresă, activă sau inactivă. 100 de CUI-uri pe cerere, o cerere pe
  secundă, deci toată țara într-o noapte.
- Căutare în Postgres, cu potrivire fără diacritice — „napolact" să găsească „SC NAPOLACT SA".
- Verdictul: **da, e înregistrată în România** / **nu o găsim în registrul român**, plus CUI,
  județ, dacă e activă, și data la care am văzut datele.

**Gata când:** scrii un nume și primești răspunsul din baza noastră, fără niciun apel extern pe
drumul cererii.

## Faza 2 — aplicația de telefon

- Aplicația Android peste aceeași bază.
- Un singur ecran: câmp de căutare, listă de rezultate, pagina firmei cu răspunsul.
- Logica de căutare și verdict rămân în Postgres, nu în aplicație — ca site-ul de mai târziu să
  dea exact aceleași rezultate.

**Gata când:** poți căuta o firmă de pe telefon și primești același răspuns ca din faza 1.

## Faza 3 — scanarea

Codul de bare e doar o cale mai rapidă spre aceeași căutare.

- `products`: GTIN, nume, marcă, firma.
- Scanezi, se caută GTIN-ul, ajungi direct la firmă.
- Cod necunoscut: cade înapoi pe căutarea după nume. Nu e ecran de eroare.
- Prefixul 594 nu e folosit ca dovadă niciodată — doar ca să găsim produsul.

**Gata când:** scanarea unui produs cunoscut duce direct la pagina firmei, iar una necunoscută
duce la căutare.

## În afara MVP-ului

Proprietate și procente, grupul, producția, scorurile combinate, site-ul Next.js, iOS,
furnizorii plătiți, orice atinge registrul beneficiarilor reali.

## Un lucru de spus în interfață

Răspunsul „da, e înregistrată în România" va fi citit de mulți ca „e firmă a românilor". Nu e
același lucru — o fabrică germană din Cluj primește tot „da". O linie sub verdict care spune la
ce am răspuns, cu legătură către FAQ, e obligatorie, nu opțională.
