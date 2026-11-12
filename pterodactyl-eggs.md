# Inhoud
- [Inleiding](#inleiding)
- [Wat is een Egg?](#wat-is-een-egg)
- [Eggs zoeken](#eggs-zoeken)
- [Nest aanmaken](#nest-aanmaken)
- [Egg importeren](#egg-importeren)
- [Egg controleren en aanpassen](#egg-controleren-en-aanpassen)
- [Server aanmaken met de egg](#server-aanmaken-met-de-egg)
- [Poorten per game](#poorten-per-game)
- [Eigen egg maken](#eigen-egg-maken)
- [Problemen oplossen](#problemen-oplossen)

# Inleiding
Deze handleiding legt uit hoe je extra **Eggs** toevoegt aan het Pterodactyl panel, zodat je verschillende soorten gameservers kan draaien (Minecraft, Valheim, Rust, Terraria, ...).

**Vereisten:**
- Een werkend panel en minstens één node met een groen hartje (zie de installatiehandleiding)
- Beheerrechten op het panel
- Vrije poorten in de *Allocation* van de node

# Wat is een Egg?
Een egg is een configuratiebestand (JSON) dat Pterodactyl vertelt hoe een bepaalde server moet worden geïnstalleerd en gestart. Een egg bevat onder andere:

- **Docker image(s)**: de omgeving waarin de server draait (de *Yolks*)
- **Startup command**: het commando waarmee de server start
- **Stop command**: hoe de server netjes wordt afgesloten
- **Variables**: instellingen die per server aangepast kunnen worden (bv. versie, map, max spelers)
- **Install script**: script dat de serverbestanden downloadt bij de eerste installatie
- **Config files / startup detection**: hoe het panel ziet dat de server klaar is met opstarten

Eggs zitten in een **Nest** (een groep). Bv. de nest *Minecraft* bevat de eggs Vanilla, Paper, Forge, enz.

# Eggs zoeken
Een grote verzameling community eggs staat op GitHub:

- [parkervcp/eggs](https://github.com/parkervcp/eggs): eggs voor Pterodactyl, gesorteerd per categorie (`game_eggs`, `bots`, `database`, ...)

In de map `game_eggs` staat per spel een map, en daarin een of meerdere bestanden die beginnen met `egg-` en eindigen op `.json`.

**LET OP**
- Download enkel eggs die bedoeld zijn voor **Pterodactyl**. Eggs voor Pelican (de fork van Pterodactyl) hebben een ander formaat.
- Kies de egg die bij je panelversie past (bv. 1.x). Lees de `README.md` in de map van de egg, daar staan vaak extra instructies of vereiste poorten.
- Eggs bevatten scripts die als root in een container draaien. Gebruik enkel eggs van bronnen die je vertrouwt.

# Nest aanmaken
Je kan een egg in een bestaande nest plaatsen, of een nieuwe nest maken om je eggs te groeperen.

1. Open het panel en ga naar de admin-instellingen (tandwieltjes rechtsboven).
2. Ga naar ***Service Management -> Nests***.
3. Klik op **Create New**.
4. Vul een **naam** in (bv. `Survival Games`) en een korte beschrijving.
5. Klik op **Save**.

Standaard bestaan er al nests voor *Minecraft*, *Source Engine*, *Voice Servers* en *Rust*. Voor die spellen hoef je vaak geen nieuwe egg toe te voegen.

# Egg importeren
1. Download het `.json` bestand van de egg op je eigen computer (op GitHub: open het bestand, klik op **Raw** en sla het op, of gebruik de downloadknop).
2. Ga in het panel naar ***Service Management -> Nests***.
3. Klik rechtsboven op **Import Egg**.
4. Klik op **Choose File** en selecteer het `.json` bestand.
5. Kies onder *Associated Nest* de nest waarin de egg moet komen.
6. Klik op **Import**.

De egg staat nu in de gekozen nest. Je ziet hem terug onder ***Nests -> [naam van de nest]***.

> Tip: je kan ook een egg van een bestaande egg bijwerken met de knop **Update Egg** op de pagina van de egg (tab *Configuration*), door een nieuwe versie van het JSON bestand te uploaden.

# Egg controleren en aanpassen
Klik op de egg in de nest. Controleer de volgende tabs voordat je een server aanmaakt.

## Configuration
- **Docker Images**: controleer of het image bestaat en past bij de spelversie (bv. een Java-versie voor Minecraft).
- **Startup Command**: bevat meestal variabelen zoals `{{SERVER_MEMORY}}` en `{{SERVER_PORT}}`. Pas dit enkel aan als je weet wat je doet.
- **Stop Command**: bv. `stop` voor Minecraft. Een verkeerd stop command zorgt ervoor dat de server hard wordt afgebroken.

## Variables
Hier staan de instellingen die bij het aanmaken van een server ingevuld kunnen worden. Per variabele kan je de standaardwaarde en de gebruikersrechten instellen (zichtbaar/aanpasbaar voor gebruikers).

## Install Script
Dit script wordt uitgevoerd bij de eerste installatie van een server. Het gebruikt een aparte container (meestal `ghcr.io/pterodactyl/installers:debian` of `alpine`). Hier moet normaal niets aan aangepast worden.

# Server aanmaken met de egg
1. Ga naar ***Management -> Servers*** en klik op **Create New**.
2. **Core Details**: geef een naam en kies de eigenaar (een gebruiker van het panel).
3. **Allocation Management**: kies de node en een vrij IP/poort uit je gereserveerde range.
4. **Application Feature Limits**: beperk eventueel het aantal databases, extra poorten en backups.
5. **Resource Management**: stel RAM, schijfruimte en CPU in. Houd rekening met wat de node nog vrij heeft.
6. **Nest Configuration**: kies de nest en daarna de egg.
7. **Docker Configuration**: kies het image (staat meestal al goed).
8. **Startup Configuration**: vul de variabelen van de egg in (bv. versie, map, wachtwoord).
9. Zorg dat **Start Server when Installed** aan staat en klik op **Create Server**.

Open nu de server via het gewone dashboard (niet het adminpaneel). De status verandert van *Installing* naar *Offline* of *Running* als alles goed is gegaan. Bekijk de console voor eventuele fouten.

# Poorten per game
Elke game gebruikt andere poorten. Zorg dat deze in de *Allocation* van de node staan en dat ze open staan in de firewall (UFW) en eventueel in je router. De waarden hieronder zijn de standaardpoorten, controleer altijd de README van de egg.

| Game | Standaardpoort(en) | Protocol |
|---|---|---|
| Minecraft (Java) | 25565 | TCP |
| Minecraft (Bedrock) | 19132 | UDP |
| Terraria | 7777 | TCP |
| Valheim | 2456-2458 | UDP |
| Rust | 28015 (spel), 28016 (RCON) | UDP / TCP |
| Counter-Strike 2 | 27015 | TCP en UDP |
| Palworld | 8211 | UDP |
| Project Zomboid | 16261 (+ 16262 en hoger voor spelers) | UDP |
| ARK: Survival Evolved | 7777, 7778 (spel), 27015 (query) | UDP |

Voorbeeld voor UFW:

```sh
sudo ufw allow 25565/tcp
sudo ufw allow 2456:2458/udp
```

**LET OP:** sommige games hebben meerdere poorten nodig. Voeg die dan toe als **extra allocations** aan de server (tab *Build Configuration* in het admin-paneel van de server) en gebruik ze in de variabelen van de egg.

# Eigen egg maken
Bestaat er geen egg voor jouw game, dan kan je zelf een egg maken:

1. Ga naar ***Nests -> [nest] -> New Egg***.
2. Vul naam, beschrijving en het **Docker image** in (een bestaand [yolk](https://github.com/pterodactyl/yolks) of een eigen image).
3. Vul het **Startup Command** in met de juiste variabelen.
4. Stel de **Stop Command** en de **Startup Detection** in (de tekst in de console waaraan het panel ziet dat de server klaar is).
5. Voeg onder **Variables** de instellingen toe die je per server wil kunnen aanpassen.
6. Schrijf onder **Install Script** de stappen om de serverbestanden te downloaden (bv. met SteamCMD voor Steam-games).
7. Test met een nieuwe server en bekijk de console-uitvoer.

Voor het exporteren: klik op de egg en gebruik de knop **Export** zodat je het JSON bestand kan bewaren of delen.

# Problemen oplossen
| Probleem | Mogelijke oorzaak / oplossing |
|---|---|
| Server blijft op *Installing* hangen | Bekijk de installatielog, controleer internettoegang van de node en vrije schijfruimte |
| Foutmelding dat het Docker image niet bestaat | Controleer de naam/tag van het image in de egg en of de node internet heeft |
| Server crasht direct bij het opstarten | Te weinig RAM toegewezen, of een variabele (bv. versie) is fout ingevuld |
| Server start maar niemand kan verbinden | Poort niet open in UFW/router, of verkeerd protocol (TCP vs UDP) |
| Hartje van de node is rood | Wings draait niet of `config.yml` klopt niet (zie installatiehandleiding) |
| Egg importeren mislukt | Het bestand is geen geldig Pterodactyl-egg (bv. een Pelican-egg of een HTML-pagina van GitHub in plaats van de raw JSON) |

Logs van Wings bekijk je met:

```sh
sudo journalctl -u wings -f
```

---

# Sources
- [Pterodactyl docs](https://pterodactyl.io/panel/1.0/getting_started.html)
- [Community eggs (parkervcp/eggs)](https://github.com/parkervcp/eggs)
- [Pterodactyl yolks](https://github.com/pterodactyl/yolks)
