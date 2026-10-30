Sunt medic internist în România. Construiesc o bază de cunoștințe (knowledge base) pentru practica
clinică. Mai jos ai regulile de lucru și conținutul complet al fișierelor create deja într-o sesiune
anterioară. Recreează-le exact ca fișiere în repo-ul meu privat, într-un branch nou, apoi continuă de acolo.

REGULI OBLIGATORII

1. Date de pacienți: NU introduc și NU trebuie să primești date de pacienți (nume, CNP, date de
   naștere, localități, imagini cu antet). Dacă văd că le pomenesc, oprește-mă. Baza de cunoștințe
   (ghiduri, protocoale) e separată de orice bază de pacienți, care stă doar local și criptat.

2. Surse și actualitate: pentru fiecare întrebare clinică caută pe web și citează sursa oficială.
   Prioritate: cnas.ro (protocoale terapeutice, liste de compensate), ms.ro, Monitorul Oficial,
   anm.ro (RCP, nomenclator), ghidurile societăților române, apoi ESC/ESH/ADA/EASD/KDIGO.
   Separă mereu două straturi: „ce spune ghidul" (ideal) și „ce poți prescrie și deconta în România".
   UpToDate e doar verificare în plus, nu sursă pentru decontare. Dacă nu ai căutare web în sesiune,
   spune-mi de la început: notele vor fi atunci doar din memorie, marcate ca atare.

3. Etichetare obligatorie a fiecărei afirmații clinice:
   [Verificat web, data] = confirmat în sesiune pe o sursă, cu link
   [Din memorie] = cunoștințele tale, de confirmat pe PDF
   [Nu știu] = nu completa cu presupuneri; scrie explicit că lipsește
   Dacă nu găsești protocolul românesc actual, spune-o direct și marchează secțiunea „LIPSĂ".
   Nu inventa numere de ordin, doze, praguri de decontare sau clase de recomandare.

4. Notele din kb/ au antet: specialitate, temă, status (ciornă până confirm eu), data verificării web,
   surse. Structură: 1) Strat RO, 2) Strat european, 3) Trepte de tratament, 4) Neverificat, 5) Surse.
   Fără doze și contraindicații din memorie: acelea se iau din RCP.

5. Fii consilier, nu asistent: dacă o cerere a mea e riscantă sau o presupunere e greșită, spune-mi asta
   în prima propoziție, cu motivul și o alternativă. Marchează-ți încrederea ([Certain]/[Likely]/[Guessing]).
   Nu ceda dacă îți contrazic poziția fără informații noi.

6. Lucrează într-un branch nou în repo-ul meu privat, fă commit-uri clare și nu crea pull request
   fără să-ți cer. Nu comite niciodată date de pacienți.

CONȚINUTUL FIȘIERELOR EXISTENTE (recreează-le identic, la aceleași căi)

===== ÎNCEPUT FIȘIER: medical-db/SETUP.md =====
# Setup baza de date + knowledge base — medic internist, România

> Document de arhitectură. NU conține și NU trebuie să conțină date de pacienți.
> Repo-ul acesta este pe GitHub (cloud): nimic identificabil nu se commit-uie aici.

## 0. Regula de bază: două sisteme separate

| | Knowledge base (KB) | Baza de pacienți (PHI) |
|---|---|---|
| Conținut | ghiduri, protocoale, doze, interacțiuni, note proprii | date identificabile, analize, diagnostice |
| Sensibilitate | publică / licențiată | date de sănătate, GDPR art. 9 |
| Unde stă | oriunde (Git, cloud OK) | local, criptat, cu backup criptat |
| AI/LLM | OK | doar local sau cu DPA + anonimizare |

Nu amesteca cele două. KB-ul poate fi în Git; PHI nu, niciodată.

## 1. Knowledge base

**Format:** folder de fișiere Markdown (+ PDF-uri sursă), versionat cu Git.
- Editor: Obsidian (gratuit, local, linkuri între note, căutare rapidă).
- Structură propusă:

```
kb/
  cardiologie/   (HTA, FiA, IC, SCA, dislipidemii)
  endocrino/     (DZ tip 2, tiroidă, obezitate)
  nefrologie/    (BCR, KDIGO)
  gastro/        pneumo/   hematologie/   reumato/   infectioase/
  urgente/       (algoritmi rapizi)
  medicamente/   (fișe proprii: doză, CI, interacțiuni, decontare CNAS)
  calculatoare/  (CKD-EPI, CHA2DS2-VASc, HAS-BLED, SCORE2, Wells, CURB-65)
  surse/         (PDF-uri originale, nume: societate_an_titlu.pdf)
```

- Fiecare notă are antet: `sursă`, `an`, `nivel de dovadă`, `data verificării`.
- Surse prioritare: ESC/ESH, ADA/EASD, KDIGO, GOLD, GINA, EASL, ghiduri
  Societatea Română de Cardiologie / Diabet / Nefrologie, protocoalele
  terapeutice CNAS (Ministerul Sănătății), Nomenclatorul ANMDMR.
- Căutare semantică (opțional, faza 2): indexare locală cu embeddings
  (ex. `sqlite-vec` sau Chroma) peste folderul `kb/`, cu citare obligatorie a sursei.

## 2. Baza de pacienți

**Recomandare pentru început:** nu construi de la zero un EHR dacă ai deja
soft de cabinet (cerut oricum pentru raportare SIUI/DES/CNAS). Construiește
doar un strat personal de ajutor (urmăriri, tendințe, reminders).

Dacă totuși îți faci propria bază:
- **SQLite + SQLCipher** (fișier unic, criptat AES-256, local) — suficient pentru un cabinet.
  Upgrade la PostgreSQL doar dacă ai nevoie de acces multi-utilizator.
- Disc criptat (BitLocker/FileVault/LUKS) + parolă puternică + 2FA unde există.
- Backup 3-2-1, criptat, testat la restaurare.
- Audit log: cine/când a citit sau modificat.

### Schemă minimă

```
pacient(id_intern, an_nastere, sex, cnp_hash*, note)
consultatie(id, pacient_id, data, motiv, anamneza, examen_obiectiv, evaluare, plan)
diagnostic(id, consultatie_id, cod_icd10, status)
medicatie(id, pacient_id, substanta, doza, frecventa, start, stop, motiv_stop)
analiza(id, pacient_id, data, parametru, valoare, unitate, ref_min, ref_max)
semne_vitale(id, pacient_id, data, TA_s, TA_d, puls, SpO2, greutate, inaltime)
alergie(id, pacient_id, substanta, reactie)
urmarire(id, pacient_id, data_scadenta, ce, efectuat)
```
\* CNP: doar hash cu sare, sau deloc; nume/contact într-un tabel separat criptat.

Valoarea reală: grafice de tendință (HbA1c, creatinină/eGFR, TA, LDL),
alerte de interacțiuni, listă „pacienți restanți la control".

## 3. Cadru legal (RO/UE) — de verificat cu un DPO sau juristul CMR

- GDPR art. 9 (categorii speciale) + Legea 190/2018; notificări ANSPDCP dacă e cazul.
- Registru de prelucrări, informare pacient, bază legală (îngrijire medicală, art. 9(2)(h)).
- Secret profesional (Codul deontologic CMR, Legea 95/2006).
- Raportare obligatorie: DES/SIUI/CNAS — folosește sistemele oficiale pentru asta.
- Incident de securitate: notificare ANSPDCP în 72h.

## 4. Folosirea AI (inclusiv Claude)

- Pe KB: da, liber.
- Pe cazuri: doar anonimizat (fără nume, CNP, date exacte de naștere, localitate,
  imagini cu antet). Atenție: un caz rar se poate re-identifica și fără nume.
- Orice instrument care procesează PHI: necesită contract de prelucrare (DPA)
  și stocare în UE. Verifică înainte, nu după.
- AI-ul este suport decizional; verifici tu orice doză sau recomandare față de ghid.

## 5. Plan pe etape

1. **Săptămâna 1:** Obsidian + structura `kb/`, 10-15 ghiduri esențiale.
2. **Săptămâna 2-3:** fișe medicamente + calculatoare clinice.
3. **Luna 2:** decizi: soft de cabinet existent vs. SQLite+SQLCipher pentru stratul personal.
4. **Luna 3:** căutare semantică locală peste KB.
5. Revizuire trimestrială a ghidurilor (versiuni noi ESC/ADA apar anual).

## 6. Întrebări deschise

- Lucrezi în spital, ambulatoriu sau cabinet privat? (schimbă cine e operator de date)
- Ai deja soft de cabinet? Care?
- Câți pacienți / ce specialitate dominantă (cardio-metabolic, nefro, etc.)?
- Vrei acces de pe telefon? (crește riscul; se rezolvă cu VPN/self-host, nu cu cloud public)
===== SFÂRȘIT FIȘIER: medical-db/SETUP.md =====

===== ÎNCEPUT FIȘIER: medical-db/kb/cardiologie/dislipidemii.md =====
---
specialitate: cardiologie
tema: dislipidemii
status: CIORNA - neverificata pe surse oficiale RO
verificat_web: 2026-10-30 (doar ghid ESC/EAS, prin rezumate secundare)
cunostinte_din_memorie: pana in iunie 2026
---

# Dislipidemii

## 1. Stratul RO (CNAS / MS) — LIPSA
- Protocolul terapeutic CNAS pentru hipolipemiante (inclusiv inhibitori PCSK9) NU a fost
  gasit la cautarea web din 2026-10-30. [Nu stiu] criteriile actuale de includere/decontare.
- DE FACUT: descarca de pe cnas.ro / Monitorul Oficial protocolul in vigoare, salveaza PDF-ul in
  `surse/` cu data si completeaza aici: DCI-uri compensate, liste (A/B/C), criterii LDL,
  prescriptori, durata.

## 2. Stratul european: ESC/EAS 2019 + Focused Update 2025
Sursa: Mach F. et al., Eur Heart J 2025, doi:10.1093/eurheartj/ehaf190 (de citit textul complet).
Verificat prin rezumate secundare: tintele LDL-C au ramas cele din 2019; s-a schimbat modul
de atingere (intensificare rapida, terapie combinata), SCORE2/SCORE2-OP pentru risc,
acid bempedoic Clasa I B la intoleranta la statine.

### Tinte LDL-C (ghid 2019) [din memorie, de confirmat pe PDF]
| Risc | Tinta LDL-C |
|---|---|
| Foarte mare | < 55 mg/dL si reducere >= 50% |
| Mare | < 70 mg/dL si reducere >= 50% |
| Moderat | < 100 mg/dL |
| Scazut | < 116 mg/dL |

### Treapta terapeutica
Verificat web (2026-10-30, rezumate secundare, nu textul ghidului):
- SCA, pacient naiv la tratament si improbabil sa atinga tinta cu statina singura: statina de
  intensitate mare + ezetimib inca din internarea index (Clasa IIa, Nivel B).
- SCA, pacient deja pe hipolipemiante: intensificare in spital (Clasa I, Nivel C).
- Acid bempedoic: intoleranta la statine (Clasa I, Nivel B); adaugat la statina maxim tolerata
  +/- ezetimib la risc inalt/foarte inalt (Clasa IIa, Nivel C - sursa de nivel scazut, produc. medicamentului).
- Reducere LDL-C aprox. 23% in monoterapie, 18% adaugat la statina (acid bempedoic) - aceeasi sursa secundara.
- Pozitionarea inhibitorilor PCSK9: NU am gasit-o in materialul consultat.

Din memorie (pana in iun. 2026), de confirmat pe PDF:
1. Statina potenta, doza maxim tolerata (+ modificarea stilului de viata).
2. Reevaluare la 4-6 saptamani; daca tinta nu e atinsa: + ezetimib.
3. Inca peste tinta, risc foarte mare / FH: + inhibitor PCSK9 (evolocumab, alirocumab).
4. Alternative: acid bempedoic (+/- ezetimib) la intoleranta; inclisiran (siRNA anti-PCSK9).
5. Hipertrigliceridemie persistenta la risc inalt sub statina: se discuta icosapent etil.
- Principiul actualizarii 2025: nu astepta luni intre trepte; combinatia se poate incepe devreme
  cand distanta pana la tinta e mare.

### Monitorizare [din memorie]
- Profil lipidic la 4-12 saptamani dupa initiere/schimbare, apoi anual la stabil.
- Transaminaze inainte de initiere si daca apar simptome; CK doar la simptome musculare.

## 3. Neverificat deloc
- Ghidul ACC/AHA 2026 pentru dislipidemii apare mentionat intr-un document extern; nu l-am citit.
- Orice doza, interactiune sau CI: de luat din RCP (anm.ro), nu din aceasta nota.

## 4. Surse
- https://www.escardio.org/Guidelines/Clinical-Practice-Guidelines/All-ESC-Practice-Guidelines/Dyslipidaemias
- https://www.clinicaltrialvanguard.com/?p=18411 (sursa promotionala - Esperion)
- https://www.escardio.org/Councils/Council-for-Cardiology-Practice-(CCP)/Cardiopractice/what-is-new-in-the-2025-focused-update-of-the-2019-esc-eas-guidelines-for-the-m
- https://www.metabolicum.com/news/2025-esc-eas-dyslipidaemia-update
- https://medicaldialogues.in/amp/cardiology-ctvs/guidelines/esceas-2025-update-brings-new-recommendations-on-cholesterol-management-and-risk-assessment-154653
===== SFÂRȘIT FIȘIER: medical-db/kb/cardiologie/dislipidemii.md =====

===== ÎNCEPUT FIȘIER: medical-db/kb/cardiologie/hipertensiune.md =====
---
specialitate: cardiologie
tema: hipertensiune arteriala
status: CIORNA - neverificata pe surse oficiale RO
verificat_web: 2026-10-30 (ghid ESC 2024, prin rezumate secundare)
cunostinte_din_memorie: pana in iunie 2026
---

# Hipertensiune arteriala

## 1. Stratul RO — LIPSA
- Nu am gasit ghidul SRC actual in format citibil, nici protocolul CNAS/MS pentru HTA.
  [Nu stiu] ce prevad pentru terapia combinata sau ce combinatii fixe sunt compensate.
- Context (hotnews.ro): ~6,6 mil. hipertensivi in RO, ~30% dintre cei tratati controlati;
  alianta RO-ANPDT-HTA tinteste >=70% control pana in 2030.
- Un articol din Romanian Journal of Cardiology (2026, reference-global.com, rjc-2026-0021) pare
  relevant; am vazut doar titlul. De citit integral.
- DE FACUT: ghidul SRC + lista medicamentelor compensate (CNAS) in `surse/`, cu data.

## 2. ESC 2024 (Eur Heart J 2024;45:3912-4018, doi 10.1093/eurheartj/ehae178)
Verificat web (rezumate secundare):
- HTA = TA de cabinet >=140/90 mmHg; gradarea in stadii 1-3 a fost eliminata; a aparut categoria "TA crescuta".
- Initiere la majoritatea pacientilor: combinatie de 2 medicamente in doza mica, ideal intr-un singur comprimat
  (blocant de sistem renina-angiotensina + blocant de calciu dihidropiridinic SAU diuretic).
- Daca nu e controlat: combinatie de 3 medicamente in doza mica, ideal intr-un singur comprimat,
  apoi titrare la doze maxim tolerate.
- Tinta pentru majoritatea adultilor tratati: TA sistolica 120-129 mmHg, daca e tolerata.
- Masuratori in afara cabinetului (MATA/MAPA) preferate pentru diagnostic.

Din memorie, de confirmat pe PDF:
- TA crescuta: 120-139 / 70-89 mmHg (intervalul NU a fost confirmat de cautare).
  Interventie pe stil de viata; farmacoterapie luata in calcul la risc CV inalt dupa evaluare.
- TA diastolica tinta 70-79 mmHg.
- Hipertensiune rezistenta (3 medicamente la doza adecvata, inclusiv diuretic): se adauga spironolactona;
  alternative eplerenona, amilorid, beta-blocant, alfa-blocant; excludere cauze secundare si non-aderenta.
- Beta-blocantele: nu prima linie generala; indicatii specifice (IC cu FE redusa, post-IM, FiA, angina).
- Nu se asociaza IECA + ARB.
- Pacient fragil / >85 ani: tinte individualizate, evitarea hipotensiunii ortostatice.

## 3. Trepte (rezumat de lucru) [verificat web + memorie]
1. Confirmare diagnostic (cabinet repetat + MATA/MAPA), evaluare risc CV, cauze secundare, afectare de organ tinta.
2. Stil de viata: sare redusa, scadere ponderala, activitate fizica, alcool, fumat.
3. Combinatie dubla in doza mica (SPC): RAS-blocant + CCB sau diuretic tiazidic-like.
4. Combinatie tripla (SPC): RAS-blocant + CCB + diuretic.
5. Rezistenta: + spironolactona; orientare catre centru specializat.
6. Tinta 120-129 mmHg sistolic daca e tolerata; reevaluare la 1-3 luni.

## 4. Neverificat
- Doze, CI, interactiuni: din RCP (anm.ro), nu din aceasta nota.
- Combinatiile fixe disponibile si decontate in RO: necunoscute.

## 5. Surse
- https://pcronline.com/Cases-resources-images/Tools-and-Practice/The-Essentials/Hypertension/A-review-of-the-2024-ESC-Guidelines-on-elevated-blood-pressure-and-hypertension
- https://academic.oup.com/ehjcvp/article/11/1/7/7831665
- https://medicaldialogues.in/cardiology-ctvs/guidelines/2024-esc-guidelines-essential-ten-commandments-for-managing-elevated-blood-pressure-and-hypertension-136915
- https://hotnews.ro/?p=1993640
- https://reference-global.com/article/10.2478/rjc-2026-0021
===== SFÂRȘIT FIȘIER: medical-db/kb/cardiologie/hipertensiune.md =====

SARCINI ACUM
a) Recreează cele 3 fișiere, comite-le și spune-mi ce lipsește sau e slab verificat în ele.
b) Cere-mi să încarc în medical-db/kb/surse/ PDF-urile oficiale (protocoale CNAS în vigoare, ghidul
   SRC/ESC) și completează straturile „RO" din ele.
c) Apoi continuă, în ordine, cu: fibrilație atrială (CHA2DS2-VASc, HAS-BLED, anticoagulare),
   insuficiență cardiacă, diabet zaharat tip 2, boală cronică de rinichi.
