# Ilionx Young Professional: adfworkshop
In deze README staan de opdrachten die je kan oppakken tijdens deze hands-on workshop. Het maakt niet uit hoeveel of hoe snel je deze opdrachten maakt. Het gaat er vooral om of je begrijpt wat er gebeurd. Per opdracht word er ook een voorbeeld gegeven van wat je uiteindelijk voor elkaar wilt krijgen. Succes!
## Opdrachten

### Opdracht 1
In deze opdracht gaan we de data van de bron ophalen en wegschrijven naar de staging layer. Er zijn twee bronnen waarvan je data kan ontsluiten: Er is een API waar we data van kunnen ophalen én er is data die op de external storage account staat. Het data formaat die we hanteren in de staging layer is parquet.

### Opdracht 1a: Excel data ontsluiten
Er is een container genaamd ```ypworkshopexternal``` . Op deze blob storage staat een csv bestand die we willen ontsluiten. Het bestand heet: ```fake_data.csv``` 

Het doel is om dit bestand naar een .parquet bestand om te zetten en deze te plaatsen op de ```ypworkshopinternal``` container. 

Benodigheden:
- pipeline met copy data activity
- dataset voor de source side
- dataset voor de sink side
- linked service voor de source side
- linked service voor de sink side

Eisen:
- Geef het bestand de naam: ```fake_date.parquet``` 
- Geef ook jou eigen naam mee als directory waarin het geplaatst moet worden

Je kan gebruik maken van de bestaande linked services, maar mocht je dit ook zelf willen maken dan ben je daar ook vrij in!


### Opdracht 1b: API data ontsluiten

We gaan data ontsluiten uit de Pokédex.
In het project bestaat er een linked service die een connectie heeft met de API van deze Pokédex. Via deze API kunnen we data over pokemon verkrijgen. 

Het doel is om deze API aan te roepen en deze data om te zetten naar een .parquet bestand. Dit bestand moet geplaatst worden op de ```ypworkshopinternal``` container. 

Benodigheden:
- pipeline met copy data activity
- dataset voor de source side
- dataset voor de sink side
- linked service voor de source side
- linked service voor de sink side

Eisen:
- Geef het bestand de naam: ```{jou pokemon}.parquet``` 
- Geef ook jou eigen naam mee als directory waarin het geplaatst moet worden

Je kan gebruik maken van de bestaande linked services, maar mocht je dit ook zelf willen maken dan ben je daar ook vrij in!

### Opdracht 2 Aansturing via snowflake
Nu we data van de bron naar parquet kunnen schrijven willen we dit proces automatiseren. We gebruiken hiervoor een stuurtabel die in snowflake is aangemaakt. De tabel heet: MA.RTETL_EXTRACT_CONTROL_TABLE. 

Tip: Je kan de snowflake omgeving in je browser openen om te kijken hoe deze tabel eruit ziet voordat je hiermee gaat werken in ADF.

In ADF maken we een constructie van de volgende blokken:
- LOOKUP Activity
- FOREACH Activity
- COPY DATA Activity

Met de ```Lookup Activity``` kunnen we queryen naar de stuurtabel in snowflake, de output hiervan  gebruiken we om verdere activiteiten in de pipeline aan te sturen.

De ```For Each Activity``` staat gekoppeld aan de ```Look Up Activity```. Zoals de naam aangeeft dient dit blokje om pér regel in de output van de lookup een bepaalde activiteit uit te voeren.

De ```Copy Data Activity``` staat ín de ```For Each Activity```. Doordat het in de for each activity staat zal dit herhaadelijk uitgevoerd worden.

#### Opdracht 2a CSV data via stuurtabel
Doel: We willen hetzelfde resultaat als bij opdracht 1a. Alleen nu willen we dat het aangestuurd word door onze stuurtabel.
Bouw de constructie die hierboven beschreven word, maak alle benodigde koppelingen en schrijf opnieuw een parquet bestand naar ```ypworkshopinternal``` container. Plaats dit bestand opnieuw in jou directory.

#### Opdracht 2b Pokemon data via stuurtabel
Doel: In de stuurtabel staan twee pokemon, deze willen we alle twee in een eigen parquet bestand hebben op de ```ypworkshopinternal``` container. Bouw de constructie die hierboven beschreven word, maak alle benodigde koppelingen en zorg ervoor dat de bestanden in jou directory komen te staan.

### Opdracht 3 Datasets en parameters
In deze opdracht gebruiken we een nieuwe linked service: ```LS_REST_ANON```. Via deze linked service gaan we dezelfde API aanroepen als die in opdracht 1b en 2b gebruikt word. Het issue is hier echter dat baseurl in deze linked service geparameteriseerd is. Aan jou de taak om dit werkend te krijgen.

Benodigheden:
- een nieuwe dataset met parameters: baseurl, endpoint.

Doel: maak een nieuwe dataset genaamd: ```DS_REST_ANON``` die gebruik maakt van de linked service: ```LS_REST_ANON```. Maak dezelfde constructie als in opdracht 2b, maar maak nu gebruik van deze nieuwe dataset voor de source side.

Als eindresultaat moet de pipeline opnieuw voor beide pokemons een bestand wegschrijven.

### Opdracht 4 Azure Key Vault integreren
Het is conventie om voor credentials of api links gebruik te maken van de keyvault. In deze opdracht voegen we een key vault request toe aan onze huidige pipeline.

Benodigheden:
- WEB activity

De keyvault roepen we aan via de ```Web Activity```. Je zou online kunnen uitzoeken hoe je dit zou kunnen doen, maar ik kan het je ook zelf hier zeggen. Maak een Web Activity aan en vul de volgende gegevens in:
```
URL: @concat(pipeline().globalParameters.GP_key_vault_url,'/secrets/', ‘vul hier je keyvault_entrynaam in’,'/?api-version=7.0') 

Method: GET 

Authentication: System Assigned 

Resource: https://vault.azure.net 
```

Tip: Je kan via de browser de key vault voor dit project bekijken en zien wat hier in staat.

We willen de baseurl die we voor de pokemon api nu hardcoded meegeven voortaan vanuit de keyvault meegeven.

Doel: Verbind de WEB activity aan de lookup activity en geef plaats de output van deze activiteit in het veld waar we de baseurl van de API nu meegeven.

### Opdracht 5 Pagination
```Pagination``` word gebruikt wanneer er veel data tegelijk opgehaald moet worden van een API. Vaak hebben API's een limiet hoeveel data je binnen een request terug kan krijgen. Door middel van pagination kan je in een actie requests achter elkaar versturen.

Stel dat de pokemon api slechts 10 resultaten per keer terug geeft, maar je wilt de eerste 100 opvragen. Dan kan je dit via pagination voor elkaar krijgen.

#### 5a Pagination
In plaats van dat we naar een specifieke pokemon gaan queryen, gaan we nu een range aan id's aanvragen.

Maak hiervoor een nieuwe pipeline aan, met een nieuwe copy data activity. Vul voor de de Sink Activity hetzelfde in als hoe we bij de vorige opdrachten een bestand hebben weggeschreven. Voor de source side kiezen we voor de dataset ```DS_REST_POKEMON```. Het enige verschil is dat we nu i.p.v. een pokemon naam op de plaats van par_baseurl zetten een placeholder zetten genaamd: ```{id}```.

Deze placeholder gaan we gebruiken voor de pagination. Pagination vind je onder de opties onderin. Ik laat de uitdaging verder aan jezelf om uit te vogelen hoe je dit zou moeten gebruiken.

Veel succes!



