# CRP → Volumetric Explosions: libreria delle risorse

Lavoro preparatorio per una PR a Volumetric Explosions (VE). Obiettivo: le risorse del
Community Resource Pack (CRP 1.4.2, `CommonResources.cfg`, 170 definizioni) vengono
ricondotte alle categorie che VE già conosce. Nessuna esplosione nuova.

Bozza del file libreria: [resources-crp.cfg](resources-crp.cfg). Implementato (2026-10-11) in
`src/VolumetricExplosions/resources.cfg`, `Resources.cs` (lettura e applicazione), una riga in
`Records.Sort`, la chiamata in `Addon.Start`, la copia in `build.sh`, un punto nel README.

## Come VE classifica oggi (`Records.Sort`, `src/VolumetricExplosions/Records.cs:147`)

Ogni `PartResource` del pezzo che muore passa per `Sort(d, name, kg, units)`:

1. nomi stock esatti: `ElectricCharge` → `charge` (in unità), `LiquidFuel` → `fuel`,
   `Oxidizer`, `SolidFuel` → `solid`, `MonoPropellant` → `mono`, `XenonGas` → `gas`,
   `Ore` → `ore`; `Ablator` e `IntakeAir` ignorati;
2. altrimenti euristiche per sottostringa sul nome minuscolo, in quest'ordine:
   `hydrogen` → fuel + flag `hydrogen`; `oxygen|oxidi|nto|n2o4|htp|nitrous` → oxidizer;
   `solid`; `hydrazine|mono`; `methane|kerosene|methalox|fuel|mmh|udmh|aerozine|ethanol|karbonite`;
   `argon|krypton|xenon|nitrogen|helium|carbondioxide` → gas; `water|ammonia` → vapour;
   `ore|rock|dirt|metal|regolith` → ore;
3. nessuna corrispondenza → la risorsa non conta nulla.

Le categorie (campi di `Death`) e il loro effetto in `Plan.What` (`Plan.cs:113`):

| campo | unità | effetto |
| --- | --- | --- |
| `fuel` | kg | brucia con l'ossidante (1 : 1.222) in palla di fuoco arancione e fumo scuro; da solo brucia solo in aria con ossigeno, il resto diventa vapore |
| `oxidizer` | kg | come sopra; l'eccesso diventa mezzo vapore |
| `solid` | kg | lampo bianco, fumo chiaro (tint 1, "solid propellant and burning metal") |
| `mono` | kg | fuoco giallo pallido (tint 2) |
| `gas` | kg | niente fuoco, vapore × 3 (sbuffo freddo, come lo xenon) |
| `vapour` | kg | nuvola senza fuoco |
| `ore` | kg | più polvere a terra (`1 + ore/300`); sopprime il "piccolo botto" del pezzo vuoto |
| `charge` | unità | se non brucia altro: lampo elettrico blu-bianco, `fire = min(charge/60, 10)` |
| `hydrogen` | flag | tint 4: fiamma limpida, soot × 0.05 |

## Cosa succede oggi alle risorse CRP (simulazione delle euristiche)

Errori veri e propri:

| risorsa | oggi | perché | dovrebbe |
| --- | --- | --- | --- |
| `StoredCharge` (condensatori NFE, 133 parti in questo GameData) | `ore` con 0 kg → nulla | contiene "ore" (st**ore**d), densità 0 | ElectricCharge |
| `HTPB` (legante dei solidi) | oxidizer | contiene "htp" | SolidFuel |
| `CarbonMonoxide` | mono | contiene "mono" | gas |
| `DepletedFuel` (scorie nucleari) | fuel | contiene "fuel" | Ore |
| `RocketParts` | ore | contiene "rock" | None |

Risorse rilevanti ignorate del tutto: `LqdCO2`, `LqdCO`, `LqdHe3`, `NeonGas`, `LqdNeon`;
gli ossidanti RealFuels `MON1..MON25`, `IRFNA`, `IWFNA`, `AK20/27`, `CaveaB`, `FLOX`, `ClF3/5`,
`OF2`, `N2F4`, `LqdFluorine`; i combustibili RealFuels `UH25`, `Hydyne`, `Syntin`, `Tonka`,
`Furfuryl`, `Aniline`, `AvGas`, `Ethane`, `Ethylene`, borani; i solidi `PBAN`, `PSPC`,
`NGNC`, `HNIW`; `LqdDeuterium`/`Tritium`; tutti i minerali e metalli non chiamati
"ore/metal/rock/dirt/regolith" (`Silicates`, `Minerals`, `Uraninite`, `Lithium`...).

Già giuste per caso o per nome: `LqdHydrogen`, `LqdMethane`, `LqdOxygen`, `Kerosene`,
`NTO`, `HTP`, `Hydrazine`, `MMH`, `UDMH`, `Aerozine50`, gas nobili, `Water`, `Ammonia`,
`Metals`, `MetallicOre`, `Rock`, `Dirt`, `Regolith`.

## Assegnazioni proposte

Nomi di categoria nel file = nome della risorsa stock a cui "equivale", più tre parole
proprie: `Hydrogen` (LiquidFuel con fiamma limpida), `Vapour`, `None`.

| categoria | risorse CRP |
| --- | --- |
| **LiquidFuel** | LqdMethane, Methane, Kerosene, Karbonite, Aerozine50, MMH, UDMH, UH25, Hydyne, Syntin, Tonka250, Tonka500, Furfuryl, Aniline, AvGas, Ethanol, Ethanol75, Ethanol90, Methanol, Ethane, Ethylene, Diborane, Pentaborane, Hexaborane, Decaborane, TEATEB |
| **Hydrogen** | LqdHydrogen, Hydrogen, SolidHydrogen, LqdDeuterium, Deuterium, LqdTritium, Tritium |
| **Oxidizer** | LqdOxygen, LqdOxygen18, Oxygen, NTO, MON1/3/10/15/20/25, HTP, NitrousOxide, IRFNA-III, IRFNA-IV, IWFNA, AK20, AK27, CaveaB, LqdFluorine, Fluorine, FLOX30/70/88, ClF3, ClF5, OF2, N2F4, Chlorine |
| **SolidFuel** | HTPB, PBAN, PSPC, NGNC, HNIW |
| **MonoPropellant** | Hydrazine |
| **XenonGas** | ArgonGas, LqdArgon, KryptonGas, LqdKrypton, LqdXenon, NeonGas, LqdNeon, Helium, Helium3, Helium4, LqdHelium, LqdHe3, Nitrogen, LqdNitrogen, LqdNitrogen15, CarbonDioxide, LqdCO2, CarbonMonoxide, LqdCO |
| **Vapour** | Water, WasteWater, HeavyWater, Ammonia, LqdAmmonia |
| **Ore** | Dirt, Rock, Regolith, Substrate, Hydrates, Gypsum, Minerals, ExoticMinerals, MetallicOre, Metals, RareMetals, RefinedExotics, Karborundum, Silicates, Silicon, Monazite, Spodumene, Fluorite, Borate, Nitratine, Salt, Carbon, Sulphur, Alumina, Aluminium, Boron, Lithium, Lithium6, LithiumHydride, LithiumDeuteride, Sodium, Potassium, Caesium, Calcium, Magnesium, Lead, LeadBallast, Uraninite, EnrichedUranium, DepletedUranium, DepletedFuel, Actinides, UraniumNitride, UF4, Thorium, ThF4, Plutonium-238 |
| **ElectricCharge** | StoredCharge |
| **None** | IntakeAtm, IntakeLqd, CompressedAir, Food, Supplies, ColonySupplies, Fertilizer, Mulch, Organics, Waste, Recyclables, Chemicals, Polymers, PolyvinylChloride, Machinery, MaterialKits, SpecializedParts, SpareParts, RocketParts, Glykerol, FusionPellets, ResourceLode, SolarWind, VacuumPlasma, Antimatter, AntiHydrogen, ExoticMatter, ChargedParticles, Megajoules, ThermalPower, WasteHeat |

Criteri seguiti:

* le scelte già fatte dall'autore restano (MMH/UDMH/Aerozine come fuel e non mono,
  HTP come ossidante, Karbonite come fuel);
* solidi estratti o raffinati → Ore (polvere, niente botto proprio); materiali di consumo
  e "parti" leggere → None;
* isotopi dell'idrogeno → Hydrogen (bruciano come H2);
* gas non combustibili, compresi CO e CO2 → XenonGas;
* pseudo-risorse di flusso a densità 0 (Megajoules, ThermalPower, WasteHeat,
  ChargedParticles) → None; l'unica energia *immagazzinata* è StoredCharge.

Da discutere con l'autore (scelte di giudizio, non di fatto):

* **Lithium / Lithium6** (propellente NF Propulsion, 47 parti qui): proposto Ore. L'alternativa
  è SolidFuel, che l'autore stesso descrive come "solid propellant and burning metal", ma una
  tonnellata di litio darebbe un lampo da 800 kg di propellente solido: troppo.
* **Megajoules** (KSPI): proposto None. Alternativa `ElectricCharge` con fattore ×1000; il
  lampo elettrico è comunque limitato a `fire = 10`.
* **AntiHydrogen**: oggi dà una fiamma a idrogeno (per nome); proposto None perché
  l'annichilazione non è una delle esplosioni esistenti. Caso di nessuno in pratica.
* **StoredCharge → ElectricCharge 1 : 1**: NFE scarica i condensatori in EC circa alla pari;
  se servisse un fattore, il formato può prevederlo (`= ElectricCharge *N`).

## Strategia di implementazione proposta

Principio: una tabella esatta nome → categoria consultata **prima** delle euristiche, che
restano com'è per tutto ciò che la tabella non nomina. Così nessun comportamento attuale
cambia se non per le risorse elencate.

1. **Il file.** `src/VolumetricExplosions/resources.cfg`, nodo `VOLUMETRIC_EXPLOSIONS_RESOURCES`,
   una riga per risorsa (`LqdMethane = LiquidFuel`). Installato in `PluginData/` accanto a
   `settings.cfg` da `build.sh` (una `cp` in `build()`; nel ramo `install` va *sovrascritto*,
   a differenza di `settings.cfg`, perché è dato della mod e non del giocatore). Stare in
   PluginData rispetta la scelta dell'autore: VE oggi non ha nessun `.cfg` letto dal gioco, e
   installarla o toglierla non costa mai un avvio lento con ModuleManager.
2. **La lettura.** In `Records` (o in `Settings.Load`, che già fa la stessa cosa per
   `settings.cfg`): `ConfigNode.Load(Folder + "PluginData/resources.cfg")`, iterazione su
   `node.values`, riempimento di un `Dictionary<string, Kind>` con chiave minuscola.
   `Kind` = categoria (enum) e flag idrogeno; un log con quante righe sono state lette.
   Categorie accettate: i sette nomi stock, `Hydrogen`, `Vapour`, `None` (case-insensitive).
3. **`Sort`.** Dopo i confronti stock esatti e prima delle euristiche:
   `if (table.TryGetValue(name, out k)) { Apply(d, k, kg, units); return; }`.
   `Apply` fa esattamente ciò che fanno i rami esistenti (`charge += units`, gli altri `+= kg`,
   `Hydrogen` → `fuel += kg; hydrogen = true`, `None` → nulla). Circa 30 righe in tutto.
4. **Porta aperta per gli altri mod (facoltativo, 3 righe).** Dopo il file in PluginData,
   leggere anche `GameDatabase.Instance.GetConfigNodes("VOLUMETRIC_EXPLOSIONS_RESOURCES")`:
   un altro mod, o il giocatore con una patch ModuleManager, può aggiungere o sovrascrivere
   righe senza toccare i file di VE. Costa zero quando non ce ne sono.
5. **Documentazione.** Una riga nel README ("Other mods' resources", con il nome del file)
   e nella pagina wiki "What decides how an explosion looks".

Alternative scartate:

* *estendere le euristiche*: non risolve `StoredCharge` (contiene "ore"), `HTPB` (contiene
  "htp"), `CarbonMonoxide` (contiene "mono"); ogni nuova sottostringa rischia nuove collisioni;
* *tabella solo nel codice C#*: funziona, ma ogni aggiunta è una PR; il file la rende
  modificabile dal giocatore e dagli altri mod;
* *`.cfg` dentro `GameData/VolumetricExplosions/` letto da GameDatabase*: è la via "normale"
  di KSP, ma farebbe perdere a VE la proprietà di non avere config letti dal gioco.

## Fuori ambito (annotato per completezza)

Risorse non CRP presenti in questo GameData che la stessa libreria potrebbe coprire in un
secondo file: `MtlHydrogen`, `FissionPellets`, `FissilePebbles`, `NuclearSaltWater` (Far Future
Technologies), `StaticCharge`, `Graviolium` (Wild Blue), `Snacks`, `FreshAir`, `Soil`, ecc.
