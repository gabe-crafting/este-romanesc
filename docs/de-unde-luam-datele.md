# De unde luăm datele

Patru lucruri pe care le arătăm despre o firmă. Pentru fiecare: ce e, de unde îl luăm și cât
de sigur e.

---

## 1. Compania și țara în care e declarată

**Ce e.** Firma care stă în spatele produsului, cu CUI-ul ei, și țara în care e înregistrată.
E cel mai simplu lucru dintre toate patru: ori are CUI românesc, ori nu are.

**De unde.**

- **ANAF**, serviciul public de verificare după CUI — nume, adresă, dacă firma e activă sau
  inactivă. Gratuit, fără cont.
- **Ministerul Finanțelor** — bilanțurile depuse: cifră de afaceri, număr de angajați, cod CAEN.
  Gratuit.

**Cât de sigur.** Foarte sigur, și avem practic toate firmele din România. Aici nu ghicim nimic.

> Atenție: „declarată în România" nu înseamnă „a românilor". Înseamnă doar că plătește taxele
> aici. Cine o deține e punctul 3.

---

## 2. Grupul din care face parte

**Ce e.** Cine o controlează. Urcăm din acționar în acționar până sus: firma românească poate
fi deținută de un holding din Olanda, care e deținut de o firmă din Germania, și așa mai
departe. Tot lanțul ăsta e „grupul".

Grupul nu e o firmă în sine — e un set de firme separate, fiecare cu CUI-ul și contabilitatea
ei, care au același stăpân. De aceea firma din România rămâne contribuabil român chiar dacă
grupul e străin.

**De unde.**

- **Registrul Comerțului (ONRC)** — asociații și administratorii. Se obțin prin portal și prin
  extrase, nu printr-o listă publică descărcabilă.
- **Furnizori specializați** (termene.ro, listafirme.ro, risco.ro) — aceleași date, dar
  împachetate ca API. Contra cost.
- **Registrul beneficiarilor reali** — ar fi cea mai bună sursă, dar **nu este public**.
  Accesul e permis autorităților și celor care justifică un interes legitim.

**Cât de sigur.** Parțial. La firmele mari, de obicei putem urca tot lanțul. La firmele mici,
de multe ori ne oprim după un nivel sau două. Când ne oprim, spunem că ne-am oprit.

---

## 3. Cât e românesc ownership-ul

**Ce e.** Un procent: cât din firmă ajunge, la capătul lanțului, la oameni sau firme din
România. Se calculează înmulțind cotele pe fiecare drum. Dacă un român are 40% dintr-o firmă
care are 50% din fabrică, el deține 20% din fabrică.

Ies **trei numere**, nu unul:

| | |
| --- | --- |
| **românesc** | cât am urmărit până la proprietari din România |
| **străin** | cât am urmărit până la proprietari din afară |
| **necunoscut** | cât nu am putut urmări |

Cele trei adună 100%. `Necunoscut` nu devine niciodată zero și nu se împarte între celelalte
două — dacă nu știm, scriem că nu știm. O firmă 30% românească, 10% străină și 60% neurmărită
nu e „30% românească", e „cel puțin 30%, restul nu se știe".

**De unde.** Din același graf de la punctul 2. Nu e o sursă nouă, e un calcul peste aceleași
date.

**Cât de sigur.** Exact cât de sigur e punctul 2. De asta arătăm și procentul necunoscut — ca
să se vadă cât de mult din răspuns e de fapt o necunoscută.

---

## 4. Cât e românească producția

**Ce e.** Dacă firma chiar produce în România și la ce scară — nu doar dacă are birou aici.

**De unde.**

- **Codul CAEN** (Ministerul Finanțelor / ONRC) — arată dacă activitatea principală e producție
  sau doar comerț.
- **Punctele de lucru** (ONRC) — unde are fabrici și depozite, nu doar sediul social.
- **Numărul de angajați** din bilanț (Ministerul Finanțelor) — o firmă cu 800 de oameni și cod
  de producție nu e o firmă de hârtie.
- **Marca ovală de pe ambalaj**, la alimente de origine animală — „RO" plus un număr de
  autorizare, pe care îl verificăm în listele **ANSVSA**. Asta chiar spune unde s-a procesat
  produsul.
- **Textul de pe etichetă**, fotografiat de utilizator. Atenție: *Ambalat în România* nu
  înseamnă *Produs în România*.

**Cât de sigur.** Bun pentru firma din România, dar cu o limită pe care e cinstit s-o spunem:
putem spune dacă se produce **în România**, nu ce procent din fabricile unui grup străin sunt
aici. Registrele românești văd doar firma din România — câte fabrici are grupul în Polonia sau
Vietnam nu scrie nicăieri unde avem noi acces.

---

**Ce nu folosim ca dovadă:** codul de bare. Prefixul 594 arată doar că firma s-a înscris la GS1
România, nu unde s-a fabricat produsul. Îl folosim ca să găsim produsul, atât.

**Fiecare informație afișată are sursa și data la care am văzut-o.** Dacă un număr pare greșit,
se poate vedea de unde vine.
