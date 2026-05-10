# Napredni Relacioni Model Baze Podataka za Skauting Sistem

Ovaj repozitorijum sadrži **sveobuhvatnu arhitekturu baze podataka** dizajniranu za upravljanje kompletnim ekosistemom skautinga u vrhunskom fudbalskom klubu. Model je visoko normalizovan (3NF) i omogućava precizno praćenje performansi, administracije i analitike.

## Arhitektura sistema

Sistem je podeljen na nekoliko logičkih modula koji zajedno čine celinu:

### 1. Centralni registar (Core Entities)
*   **Igraci i Klubovi:** Detaljna evidencija o fudbalerima i klubovima širom sveta.
*   **Stadioni i Gradovi:** Geografska hijerarhija i infrastruktura gde se mečevi održavaju.
*   **Agenti i Skauti:** Upravljanje ljudskim resursima i eksternim saradnicima.

### 2. Modul za performanse i analitiku
*   **Utakmice i Sezone:** Praćenje mečeva kroz različite vremenske periode i takmičenja.
*   **Statistika utakmice:** Detaljni kvantitativni podaci o učinku igrača na terenu (minuti, golovi, asistencije, kartoni).
*   **Vestine i Pozicije:** Standardizovani šifarnik fudbalskih veština i taktičkih uloga koje igrač može da pokriva.
*   **Igrac_Vestina:** Istorijat ocenjivanja specifičnih atributa igrača sa datumima procene.

### 3. Skauting proces i Izveštavanje
*   **Skauting nalozi:** Upravljanje radnim zadacima, prioritetima i statusima skautiranja.
*   **Izvestaji:** Finalni produkt skautinga koji integriše opšti utisak, prednosti, mane i konačne preporuke.
*   **Analize izvestaja:** Kvalitativna analiza podeljena po tipovima (tehnička, taktička, psihološka) sa ocenama i komentarima.

### 4. Administracija, ugovori i zdravstvo
*   **Ugovori:** Praćenje finansijskih detalja, plata i otkupnih klauzula uz istorijat ugovora.
*   **Pregledi:** Poseban medicinski modul za praćenje sistematskih pregleda i istorije povreda (osetljivi podaci).
*   **Medijski sadrzaj:** Povezivanje video snimaka (Wyscout/Instat klipovi), članaka i eksternih linkova sa profilom igrača.

### 5. Klasifikacioni sistem (Lookup Tables / Šifarnici)
Baza koristi veliki broj šifarnika kako bi se osigurala konzistentnost podataka:
*   `statusi_igraca`, `statusi_ugovora`, `statusi_naloga`
*   `tipovi_analize`, `tipovi_preporuka`, `tipovi_vestina`, `tipovi_medijskog_sadrzaja`
*   `rangovi_takmicenja`

## Ključne karakteristike modela
*   **Visoka granulacija:** Podaci su razbijeni na atomske vrednosti, što omogućava kompleksne SQL upite i duboku analitiku (Data Science ready).
*   **Istorizacija:** Preko datumskih polja u asocijativnim tabelama (`igrac_vestina`, `ugovori`), sistem omogućava praćenje razvoja igrača i njegovih finansija kroz vreme.
*   **Skalabilnost:** Dizajn podržava lako dodavanje novih tipova analiza ili statističkih parametara bez promene osnovne strukture.

## Tehnički detalji
*   **Tip modela:** Relacioni (RDBMS)
*   **Broj tabela:** 30+
*   **Veze:** Implementirane 1:1, 1:N i M:N relacije uz strogo definisan referencijalni integritet.


### Relacije između tabela (Relationship Architecture)

Model koristi strogo definisane relacije kako bi osigurao integritet podataka i omogućio kompleksno izveštavanje:

#### **Relacija Jedan-na-Jedan (1:1)**
Ovaj tip veze se koristi za entitete koji su direktno i isključivo povezani:
*   **Stadioni i Klubovi:** Svaki klub iz registra ima jedan primarni (matični) stadion, dok se stadion u sistemu vodi kao sedište jednog specifičnog kluba.
*   **Igraci i Pregledi:** (U zavisnosti od poslovne logike) Svaki igrač je povezan sa svojim najsvežijim medicinskim kartonom radi brze provere zdravstvenog statusa.

#### **Relacija Jedan-na-Više (1:N)**
Najčešći tip veze u sistemu koji omogućava hijerarhijsku organizaciju:
*   **Gradovi → Klubovi:** U jednom gradu može biti sedište više različitih klubova.
*   **Klubovi → Igraci:** Jedan klub poseduje više igrača pod ugovorom.
*   **Agenti → Igraci:** Jedan agent obično zastupa veći broj fudbalera.
*   **Skauti → Izvestaji:** Jedan skaut tokom sezone kreira na stotine različitih izveštaja.
*   **Takmicenja → Utakmice:** U okviru jednog takmičenja (npr. Superliga) igra se veliki broj utakmica.
*   **Sezone → Utakmice:** Jedna sezona obuhvata sve odigrane utakmice u tom periodu.
*   **Igraci → Ugovori:** Jedan igrač kroz karijeru može imati istorijat od više potpisanih ugovora.

#### **Relacija Više-na-Više (M:N)**
Ove veze su realizovane preko **asocijativnih tabela** (tabela koje spajaju dva entiteta) i one su ključne za analitiku:
*   **Igraci ↔ Pozicije (preko `igrac_pozicija`):** Jedan igrač može da pokriva više pozicija (npr. desni bek i desno krilo), a na jednoj poziciji može da igra mnogo različitih igrača.
*   **Igraci ↔ Vestine (preko `igrac_vestina`):** Svaki igrač se ocenjuje za veći broj veština (brzina, pas, šut), dok se svaka veština ocenjuje kod svih igrača u bazi.
*   **Igraci ↔ Utakmice (preko `statistika_utakmice`):** Na jednoj utakmici učestvuje više igrača, a jedan igrač tokom karijere odigra mnogo utakmica. Ova tabela čuva i konkretne učinke (golove, asiste) za taj spoj.
*   **Skauting nalozi ↔ Igraci (preko `nalog_igrac`):** Jedan radni nalog skautu može obuhvatiti praćenje više igrača na jednom turniru, dok jedan igrač može biti predmet više različitih naloga tokom vremena.
*   **Izvestaji ↔ Tipovi analize (preko `analize_izvestaja`):** Jedan sveobuhvatan izveštaj sadrži više različitih tipova analiza (tehničku, taktičku, fizičku), a svaki tip analize se koristi u hiljadama različitih izveštaja.

---

### Kako koristiti ovaj model?
1.  **SQL Upiti:** Zahvaljujući M:N vezama, možete lako izvući upite tipa: *"Prikaži sve igrače koji igraju na poziciji 'Centralni vezni', imaju ocenu za 'Pas' veću od 80 i kojima ugovor ističe za manje od 6 meseci."*
2.  **Praćenje razvoja:** Kroz asocijativne tabele sa datumima, možete generisati grafikone napretka igrača kroz sezone.
3.  **Finansijsko planiranje:** Povezanost ugovora i izveštaja omogućava upravi da vidi da li je cena igrača na tržištu opravdana njegovim skauting ocenama.

**Autor:** Mihailo Đurđević
