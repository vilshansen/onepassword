## Den komplette guide til sikkerhedskopiering af 1Password

Dit 1Password-vault er et sikkert sted for alle dine digitale værdier. Selvom 1Password er ekstremt pålidelig, er den bedste beskyttelse mod uforudsete hændelser en regelmæssig, lokal sikkerhedskopi. Denne guide dækker, hvordan du opretter og sikkert opbevarer sikkerhedskopier af dine 1Password-data, med metoder til både almindelige og avancerede brugere.

-----

### Del 1: For almindelige hjemmebrugere (GUI og fysisk backup)

Denne del af guiden er til dig, der foretrækker en simpel, visuel proces uden brug af kommandoer. Den fokuserer på at eksportere dine data via 1Password-appen og gemme dem sikkert offline.

#### Trin 1: Eksport af data via 1Password-appen

Eksport via den grafiske brugerflade (GUI) er den mest ligetil metode.

1.  **Åbn 1Password-appen** på din computer og lås den op.
2.  **Vælg den vault**, du vil sikkerhedskopiere, i venstre sidepanel. Du skal eksportere én vault ad gangen.
3.  Gå til menulinjen øverst og vælg **File \> Export \> All Items...**
4.  Vælg **1Password Interchange Format (.1pif)** som filformat. Dette er afgørende, da dette format bevarer alle vedhæftede filer, dokumenter og feltinformation, hvilket gør det muligt at importere dataene igen uden tab af information.
5.  Indtast dit **Master Password** for at godkende eksporten, og vælg derefter et midlertidigt sted på din computer til at gemme den eksporterede fil.
6.  Gentag processen for hver vault, du vil sikkerhedskopiere.

#### Trin 2: Sikker opbevaring af din Emergency Kit og backup

Dit Emergency Kit indeholder din **Secret Key** og dit **Master Password** – to nøglekomponenter til at få adgang til din konto. At opbevare dit Emergency Kit og din data-backup sikkert er lige så vigtigt som at lave dem.

**Sikre opbevaringssteder:**

  * **Papirkopi i en bankboks:** Print en kopi af dit Emergency Kit og opbevar den i en bankboks. Dette beskytter mod tyveri, brand og vandskade. Det er en af de sikreste løsninger.
  * **Brandsikkert pengeskab:** Opbevar en papirkopi og en digital kopi (på en krypteret USB-stick) i et brandsikkert pengeskab i dit hjem.
  * **Krypteret USB-stick:** Gem din digitale backup (den eksporterede .1pif-fil) på en hardware-krypteret USB-stick som en **IronKey**. Du låser den op med en adgangskode, før du gemmer filen. Opbevar stikken et sikkert sted, adskilt fra din computer.

**Hvad du IKKE skal gøre:**

  * **Opbevare på din computer i klartekst:** Undgå at gemme din backup-fil direkte på din computer, hvor den er sårbar over for hackere og malware.
  * **Opbevare i en almindelig cloud-tjeneste:** Gem aldrig din Emergency Kit eller backup-filer ukrypteret i tjenester som Google Drive eller Dropbox. Hvis din cloud-konto kompromitteres, er dine 1Password-data i fare.
  * **Sende via e-mail:** E-mail er ikke en sikker kommunikationskanal.

-----

### Del 2: For avancerede brugere (CLI og automatisering)

Denne del er for dig, der er fortrolig med kommandolinjen og ønsker at automatisere din backup-proces for maksimal effektivitet.

#### Trin 1: Installation og forberedelse af 1Password CLI

1.  **Installer 1Password CLI** ved at følge den officielle vejledning for dit operativsystem.
2.  **Installer `jq`:** Dette letvægtsværktøj er uundværligt for at behandle JSON-filer fra kommandolinjen og er nødvendigt for det automatiserede script.

#### Trin 2: Opret en automatiseret backup-proces

Det mest effektive er at oprette et script, der eksporterer alle dine vaults på én gang.

1.  **Log ind:** Åbn din terminal og log ind på din 1Password-konto. Dette vil autentificere din session.
    ```bash
    eval $(op signin --account my.1password.com)
    ```
2.  **Opret backup-scriptet:** Opret en ny fil (f.eks. `backup_1p.sh`) og indsæt følgende kode i den. Dette script henter automatisk alle dine vaults og eksporterer dem til separate, navngivne JSON-filer.
    ```bash
    #!/bin/bash

    echo "Starter 1Password backup..."

    # Hent en liste over alle vaults og loop igennem dem
    op vault list --format json | jq -r '.[].id' | while read -r vault_id; do
      
      # Hent navnet på vaulten
      vault_name=$(op vault get "$vault_id" --format json | jq -r '.name')
      
      # Rengør navnet for at skabe et gyldigt filnavn
      filename=$(echo "$vault_name" | sed 's/ /_/g' | tr -cd '[:alnum:]_')
      
      echo "Sikkerhedskopierer vault: '$vault_name' til filen: '$filename.json'"
      
      # Eksporter alle elementer fra vaulten til en JSON-fil
      op item list --vault "$vault_id" --format json > "$filename.json"
      
    done

    echo "Sikkerhedskopieringen er færdig."
    ```
3.  **Kør scriptet:** Gør scriptet eksekverbart med `chmod +x backup_1p.sh` og kør det derefter med `./backup_1p.sh`.

#### Trin 3: Sikker opbevaring af JSON-filerne

Da de eksporterede JSON-filer er i klartekst, skal du opbevare dem meget sikkert.

  * **Krypter filerne:** Brug et stærkt krypteringsværktøj som **GPG (GnuPG)** til at beskytte dine JSON-filer med et stærkt kodeord, før du gemmer dem.
  * **Følg 3-2-1-reglen:** For at opnå den højeste sikkerhed, anbefales det at følge **3-2-1-reglen**: Hav **3** kopier af dine data, på mindst **2** forskellige medier, med **1** kopi opbevaret offsite. En krypteret IronKey USB-stick og et krypteret cloud-drev (med stærk end-to-end kryptering) kan være ideelle til dette.
