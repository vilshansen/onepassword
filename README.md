## Omfattende guide til sikkerhedskopiering af dine 1Password-data

Det er en god idé regelmæssigt at sikkerhedskopiere dine 1Password-data lokalt, uafhængigt af 1Passwords egne indbyggede sikkerhedskopier. Selvom 1Password er en yderst pålidelig tjeneste, der bevarer historikken for dine elementer, giver en lokal backup dig den ultimative beskyttelse mod uforudsete hændelser. En lokal backup sikrer dine data, hvis du mister adgangen til din konto, bliver hacket, eller hvis 1Password-tjenesten af en eller anden grund bliver utilgængelig.

### Metode 1: Sikkerhedskopiering via 1Password CLI (Command Line Interface)

Denne metode er den mest fleksible og anbefales til avancerede brugere, da den gør det muligt at automatisere processen med scripts.

**1. Installation og forberedelse**

  * **Installer 1Password CLI:** Gå til den officielle 1Password CLI hjemmeside og følg instruktionerne for dit operativsystem (macOS, Windows, Linux). Det er en let proces, der typisk involverer en enkelt installationskommando.
  * **Installer `jq` (valgfrit, men anbefales):** For at gøre det nemmere at behandle de eksporterede data og bruge automatiseringsscriptet, er det en god idé at installere `jq`. Kør f.eks. `brew install jq` på macOS eller `sudo apt-get install jq` på Ubuntu/Debian.

**2. Log ind på din konto**
Åbn din terminal eller kommandoprompt og kør kommandoen for at logge ind på din 1Password-konto:

```bash
op signin
```

Følg anvisningerne på skærmen, som typisk inkluderer at angive dit kontonummer, din e-mailadresse, din hemmelige nøgle (Secret Key) og din adgangskode.

**3. Eksporter dine data**
Her er to måder at eksportere dine data på:

  * **Manuel eksport (per Vault):** Find ID'et for den vault, du vil sikkerhedskopiere, med følgende kommando:

    ```bash
    op vault list
    ```

    Brug derefter det specifikke ID til at eksportere data fra den pågældende vault. For at eksportere til en JSON-fil, brug denne kommando, og erstat `VAULT_ID` med ID'et fra din vault:

    ```bash
    op item list --vault VAULT_ID --format json > backup_Privat.json
    ```

    Gentag for hver vault, du ønsker at sikkerhedskopiere.

  * **Automatiseret eksport (med script):** For en mere effektiv tilgang kan du bruge et script, der eksporterer alle dine vaults på én gang. Opret en fil kaldet `backup_script.sh` og indsæt følgende kode i den:

    ```bash
    #!/bin/bash

    echo "Starter 1Password backup..."

    # Log ind på 1Password CLI for at autentificere sessionen
    eval $(op signin --account my.1password.com)

    # Hent en liste over alle vaults og loop igennem dem
    op vault list --format json | jq -r '.[].id' | while read -r vault_id; do
      
      # Hent navnet på vaulten for at navngive filen
      vault_name=$(op vault get "$vault_id" --format json | jq -r '.name')
      
      # Erstat mellemrum og rens navnet for et gyldigt filnavn
      filename=$(echo "$vault_name" | sed 's/ /_/g' | tr -cd '[:alnum:]_')
      
      echo "Sikkerhedskopierer vault: '$vault_name' til filen: '$filename.json'"
      
      # Eksporter alle elementer fra vaulten til en JSON-fil
      op item list --vault "$vault_id" --format json > "$filename.json"
      
    done

    echo "Sikkerhedskopieringen er færdig."
    ```

    **Vigtigt:** Gør scriptet eksekverbart ved at køre `chmod +x backup_script.sh` og kør det derefter med `./backup_script.sh`.

### Metode 2: Sikkerhedskopiering via 1Password GUI (Grafisk Brugerflade)

Denne metode er perfekt til brugere, der foretrækker en simpel, visuel proces uden at skulle bruge terminalen.

1.  **Åbn 1Password-appen:** Start 1Password-programmet på din computer og lås det op med din adgangskode.
2.  **Vælg en vault:** I venstre sidepanel skal du klikke på den vault, som du vil eksportere. Du kan kun eksportere én vault ad gangen.
3.  **Gå til eksport:** I menulinjen øverst skal du vælge **File \> Export \> All Items...** (eller tilsvarende på dansk).
4.  **Vælg filformat:** Vælg **1Password Interchange Format (.1pif)**. Dette format er det mest sikre og detaljerede, da det bevarer alle vedhæftede filer, dokumenter og feltinformation. De andre formater, som `.csv` eller `.json`, kan miste vigtig information.
5.  **Beskyt din eksport:** Indtast din 1Password-adgangskode for at godkende eksporten. Dette er en vigtig sikkerhedsforanstaltning, da det kun er dig, der skal have adgang til dine data.
6.  **Gem filen:** Vælg en midlertidig placering på din computer for at gemme den eksporterede `.1pif`-fil. Husk at gentage processen for hver vault, du vil sikkerhedskopiere.

### Sikker opbevaring på en krypteret USB-stick (f.eks. IronKey)

Uanset om du eksporterer via CLI eller GUI, er det altafgørende at gemme dine sikkerhedskopier sikkert. En krypteret USB-stick som en IronKey er et fremragende valg, da den har hardwarebaseret kryptering.

1.  **Forbered din USB-stick:** Indsæt din IronKey (eller en anden krypteret stick) i computeren. Følg vejledningen for din specifikke enhed for at låse den op. Dette involverer typisk at indtaste et stærkt kodeord, som du har oprettet.
2.  **Kopier dine filer:** Åbn mappen, hvor du har gemt dine eksporterede backup-filer. Kopier dem derefter over til den nu ulåste USB-stick.
3.  **Dobbelttjek og lås:** Efter at have kopieret filerne, skal du sikre dig, at de er overført korrekt. Derefter skal du skubbe enheden sikkert ud af computeren for at låse den igen og beskytte dine data.

**Vigtigt:** Dine eksporterede backup-filer indeholder alle dine følsomme oplysninger i et læsbart format. Opbevar derfor altid den krypterede USB-stick på et sikkert sted, ligesom du ville med din Master Password og Secret Key.

**Anbefalet sikkerhedspraksis**

  * **Kryptering med GPG:** Selvom du bruger en krypteret USB-stick, kan det være en ekstra sikkerhedsforanstaltning at kryptere dine backup-filer med et værktøj som **GnuPG** (GPG), før du gemmer dem.
  * **3-2-1-reglen:** For optimal sikkerhed, følg **3-2-1-reglen** for sikkerhedskopiering:
      * Hav **3** kopier af dine data (den originale, en lokal backup og en offsite backup).
      * Opbevar dine kopier på mindst **2** forskellige medier (f.eks. din computer og en krypteret USB-stick).
      * Hav **1** kopi opbevaret et andet sted end din primære placering (f.eks. i en bankboks eller hos en betroet slægtning).

Ved at kombinere 1Passwords stærke sikkerhed med disse backup-metoder, har du den bedst mulige beskyttelse af dine digitale værdier.

## Sikker opbevaring af dit 1Password Emergency Kit

Dit 1Password Emergency Kit er et kritisk dokument. Det indeholder din personlige **Secret Key** og dit **Master Password** – to essentielle komponenter for at få adgang til din 1Password-konto. Hvis du mister disse, kan du miste adgangen til alle dine data, og 1Password's supportafdeling kan ikke hjælpe dig med at gendanne den, da de ikke har adgang til din konto. Derfor er det altafgørende at opbevare mindst én kopi af dette dokument på et ekstremt sikkert sted, som kun du og eventuelt en betroet person har adgang til.

### Eksempler på sikre opbevaringssteder

Her er eksempler på, hvor du kan opbevare dit Emergency Kit for at maksimere sikkerheden:

* **På papir i en bankboks:** Dette er en af de sikreste løsninger. Print en kopi af dit Emergency Kit, og opbevar den i en bankboks. En bankboks er beskyttet mod tyveri, brand og oversvømmelse og er kun tilgængelig for dig, eller en person du har givet fuldmagt til. Sørg for at informere en betroet person om placeringen, men ikke indholdet.
* **Indelåst i et brandsikkert pengeskab:** Hvis du har et brandsikkert pengeskab derhjemme, er det et ideelt sted at opbevare en papirkopi. Denne løsning beskytter mod både tyveri og brand, hvilket er vigtigt, da papir er letfordærveligt.
* **Krypteret og gemt på en USB-stick (IronKey):** Hvis du foretrækker en digital kopi, skal den være krypteret. Den bedste måde at gøre dette på er at gemme en digital kopi af Emergency Kit (f.eks. som en PDF) på en krypteret USB-stick, som en IronKey. Denne type USB-stick har indbygget hardwarekryptering og kræver et kodeord for at få adgang til dataene. Opbevar USB-sticken et sikkert sted, som f.eks. i et pengeskab eller en aflåst skuffe.
* **Opbevaring hos en betroet advokat:** En advokat har tavshedspligt og professionelle procedurer for sikker opbevaring. I visse tilfælde kan det være en god idé at lade en advokat opbevare en forseglet kuvert med dit Emergency Kit i en pengeskab. Denne metode kan være relevant for folk, der er i forretning, eller som har et stort økonomisk incitament til at opbevare deres digitale værdier sikkert.

### Hvad du IKKE skal gøre

Lige så vigtigt som at vide, hvad du skal gøre, er det at kende de faldgruber, du skal undgå. Disse handlinger kan kompromittere din 1Password-konto og dine data:

* **Gem det på din computer i klartekst:** At gemme dit Emergency Kit (f.eks. som en PDF eller skærmbillede) direkte på din computer er ekstremt risikabelt. Hvis din computer bliver hacket, får hackeren direkte adgang til din Secret Key og dit Master Password.
* **Gem det i skyen (cloud storage) uden kryptering:** Selvom det er praktisk at gemme dokumenter i tjenester som Google Drive, Dropbox eller iCloud, må du **aldrig** gemme dit Emergency Kit i klartekst i skyen. Hvis din cloud-konto bliver kompromitteret, er dine 1Password-data i fare.
* **Send det via e-mail:** E-mail er ikke en sikker kommunikationskanal. Data kan læses, opsnappes eller videresendes uautoriseret. At sende dit Emergency Kit til dig selv eller andre via e-mail er en af de farligste ting, du kan gøre.
* **Opbevar det i din pung eller på din telefon:** En pung og en telefon er lette at miste, blive stjålet eller beskadige. Hvis dit Emergency Kit opbevares her, er det meget udsat for at falde i de forkerte hænder.
* **Gem det samme sted som din computer:** Hvis du opbevarer din Emergency Kit i nærheden af din computer, er den udsat for de samme risici som computeren, f.eks. brand eller tyveri.

### En god opbevaringsstrategi

For at opnå den højeste sikkerhed, anbefales det at kombinere forskellige metoder:
1.  **Print to papirkopier** af dit Emergency Kit.
2.  **Opbevar den ene i en bankboks** eller et brandsikkert pengeskab derhjemme.
3.  **Opbevar den anden kopi et andet fysisk sted**, f.eks. i et pengeskab hos et familiemedlem eller en betroet ven. Dette beskytter dig mod en katastrofe, der rammer dit primære sted (f.eks. en brand).

Ved at følge disse retningslinjer, sikrer du, at du altid har en sikker og pålidelig adgang til dine 1Password-data, selv i nødstilfælde.

