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
