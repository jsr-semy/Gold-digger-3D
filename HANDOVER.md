# Gullgraveren: Dypet – overlevering

Første del av utviklingen skjedde i en samtale som også handlet om iPhone-spillet Gullgraveren (2D). Denne filen samler alt om VR-spillet, så arbeidet kan fortsette i et eget prosjekt.

## Hvor ting ligger

- **Spill (GitHub Pages):** https://jsr-semy.github.io/Gold-digger-3D/ (legg til `?v=N` for å unngå gammel buffer på Quest)
- **Repo:** https://github.com/jsr-semy/Gold-digger-3D (gren `main`, Pages fra rot)
- **Alt i én fil:** `index.html`. Three.js 0.160.0 fra jsDelivr via importmap, ingen byggesteg.
- **Versjon:** 0.12 (står under knappene på startskjermen). Øk versjonen ved hver publisering, så det er lett å se at Quest har lastet riktig fil.
- **Publisering:** commit og push til `main`. Pages-bygget tar rundt ett minutt. Status kan sjekkes med `https://api.github.com/repos/jsr-semy/Gold-digger-3D/actions/runs?per_page=1`.

## Konseptet

Førstepersons gruvespill for Meta Quest 3 (WebXR), inspirert av Lode Runner, men med eget navn, egne nivåer og egen grafikk.

1. Spillet starter i stua med kamera-gjennomsyn (`immersive-ar`). Det knaker, gløden rundt en luke flimrer, lukene slår opp, og spilleren faller ned i gruva (full VR fra da av).
2. Hvert nivå: samle alt gullet. Da åpnes et grønt hull i nederste etasje, og spilleren faller videre ned til neste nivå.
3. Først etter siste nivå (8) tar heisen spilleren opp til stua igjen.

## Mekanikk

- **Nivåer:** genereres med frø per nivå (`genLevel`). Labyrint per etasje (DFS pluss ekstra løkker), 2–4 etasjer, voksende størrelse (7×7 til 15×15 celler, CELL = 2 m).
- **Stiger:** mellom etasjene, plassert inntil en vegg (side der fjellet er tett på begge etasjer). På øverste etasje er det en gangbro av planker ved siden av hullet. Fjellet buler inn rundt stigen (nisje i `deform`).
- **Kløfter med tau:** 1–3 celler i rette korridorer fra nivå 2. Kløfter i øvre etasjer går rett ned til etasjen under, og spilleren kan slippe seg ned. Gull kan henge over kløfta.
- **Gruvetralle på skinner:** lengste rette korridor per etasje. Kjøres fra den ene enden til den andre og dytter troll av skinnene.
- **Graving med hakke:** gulvruta ved siden av spilleren. Hullet fylles igjen etter 8 s. Et troll som faller i, sitter fast i rundt 3,8 s, og spilleren kan gå over det. Fylles hullet mens trollet sitter der, blir det begravd og dukker opp på startplassen sin.
- **Tønner:** kan dyttes og bæres (grepknapp). Et troll som går inn i en tønne, banker på den i rundt 2,8 s til den knuses (bremser trollet).
- **Steiner:** plukkes opp med grep nær gulvet og kastes. Der steinen lander, blir det en lyd som lokker troll, også i etasjen under via sjakter og kløfter.
- **Liv:** 5 ved start, +1 per nivå, maks 9.

## Trollenes sanser (stealth)

Tre tilstander, vist med øyefarge og et tegn over hodet:

| Tilstand | Øyne | Tegn | Oppførsel |
|---|---|---|---|
| Patruljerer | gule | – | vandrer, også mellom etasjer |
| Undersøker | oransje | ? | går til lyden, ser seg rundt rundt 2,8 s |
| Jakter | røde | ! | følger spilleren, leter der den så spilleren sist i 6 s |

- **Syn:** samme etasje, fri sikt gjennom rutenettet, inntil 9,5 m (3 m når spilleren sniker).
- **Lyd:**

| Kilde | Rekkevidde |
|---|---|
| Gange | 4,2 m |
| Løping | 11 m |
| Sniking | 0 m |
| Teleport | 2,5 m + 0,7 × avstand |
| Graving | 7 m |
| Tralle | 14 m |
| Kanarifugl-alarm | 13 m |
| Stein | 10 m (13 m i kløft) |
| Fall fra tau | 8 m |

- **Kanarifugler i bur:** varsler når spilleren går forbi innen 4,5 m uten å snike. Lyden lokker troll.

## Kontroller (VR)

- **Velges i startskjermen:** styrespak (myk bevegelse) eller teleport.
- **Styrespak:** venstre spak går dit hodet ser. Sidesteg er av som standard og kan slås på i menyen. Myk sving på høyre spak (hakk valgfritt). Klikk venstre spak for å løpe (kaldere fargetone mens du løper). Klikk høyre spak for å snike av/på. Å huke seg rundt 30 cm ned gir også sniking, og lykta dempes.
- **Teleport:** maks 4 ruter rett frem, ikke forbi eller ved siden av troll. Over en kløft glir spilleren hengende i tauet. Sikt rett ned fra tauet for å slippe deg ned.
- **Stiger:** gå inntil, se mot stigen og skyv spaken frem. Alternativt: grep og trekk hånd over hånd. Haptikk når hånda er nær.
- **Tau:** gå frem eller dra deg hånd over hånd med grep. Høyre spak ned slipper deg ned.
- **Hakke:** i høyre hånd. Pek på gulvruta ved siden av deg og trykk avtrekkeren.
- **Tralle:** avtrekker eller spak-klikk når du står i den.
- **Meny:** Y/X (venstre) eller B (høyre). Valg: Fortsett, Start nivået på nytt, Bevegelse, Sving, Komfort-vignett, Ro-prikker, Sidesteg med spaken, Spiller sittende, Avslutt.
- **Radar:** på venstre håndledd. Troll i farge etter tilstand, gull gult, utgang grønn. Stige ned vises som oransje ▼, stige opp som blå ▲. Kløfter vises som mørke ruter med tau.
- **Komfort:** vignett ved bevegelse, ro-prikker (fast referanseramme festet til kroppen), myk akselerasjon, sittende modus (+45 cm).

**PC-testmodus:** WASD/mus, mellomrom/C klatre, F grav, E tralle, Shift løp, Z snik, Q kast stein, Esc pause.

## Lyd

All lyd er syntetisert med WebAudio (ingen filer). 3D-lyd (HRTF-panner) for troll, kanarifugl, stein og drypp. Bakgrunnsdur fra fjellet. Lyden gjenopptas på `selectstart`/`squeezestart` i VR, fordi Quest pauser lydkonteksten.

## Grafikk

- **Fjell:** vegger og tak er rutenett som deformeres med 3D-støy (`deform`) til et hvelvet, ujevnt fjell. Normaler glattes over skjøter. Prosedyrale teksturer pluss normalkart (512 px).
- **Gulv:** instansiert plate med humper (`FLOOR_GEO`), fordi graving skjuler enkeltceller via instansmatrise.
- **Rekvisitter:** tømmerrammer (instansiert, tre + jern), lykter (instansiert, de tre nærmeste lyktene får ekte punktlys), kanarifuglbur, tønner (lathe), kasser, småstein (instansiert), skinner og tralle.
- **Troll:** prosedyralt, flekket hud, lange armer, hale. Animasjonsnavn som brukes: `arm-1`, `arm1`, `leg-1`, `leg1`.
- **Ytelse:** brukeren rapporterte at Quest 3 kjører jevnt med versjon 0.12.

## Testing

Headless Chrome med SwiftShader og en lokal kopi der importmap peker på `node_modules/three` (CDN blokkert i sandkassa). `window.__dbg` eksponerer blant annet `goto(f,x,z)`, `setLevel(n)`, `dig`, `trolls`, `modes`, `ropes`, `rails`, `tryRide`, `makeNoise`, `yaw` og `pitch`. Selve VR-delen kan bare testes i brillene.

## Ideer og åpne punkter

- Finpusse vanskelighet og balanse for stealth (hørsels- og synsavstander, trollfart).
- Verifisere fall fra tau i VR (bare testet logisk).
- Bevegelsesprikker og komfort: brukeren vurderer effekten.
- Mer realistisk grafikk generelt (brukeren ønsker høyere detaljnivå, så lenge ytelsen holder).
- Lengre spill: flere nivåer, nivåtemaer (vannfylte sjakter, krystallhuler), heis mellom nivåer.
- Lagring av fremgang og topplister i VR.
- Mulig native Quest-app senere (WebXR fungerer nå via nettleseren).

## Om brukeren

Jürgen, skriver norsk. Tester i Quest 3, ofte sittende. Liker konkrete endringer levert raskt, med kort oppsummering av hva som er endret og hva som må testes i brillene.
