## applicatieantigrav

> **Doelstelling:** Ontwikkel een uitgebreide webapplicatie (Mobile-first design) voor de leiding van een jeugdbeweging (KSA). De app dient voor het beheren van de agenda, bar-consumpties (strepen), facturatie, frituur-bestellingen, notificaties, en leuke extra's zoals bierpong en quotes.

# Project: KSA LeidingsApp (AIStudio-KSA-APP)
**Doelstelling:** Ontwikkel een uitgebreide webapplicatie (Mobile-first design) voor de leiding van een jeugdbeweging (KSA). De app dient voor het beheren van de agenda, bar-consumpties (strepen), facturatie, frituur-bestellingen, notificaties, en leuke extra's zoals bierpong en quotes.

## 1. Tech Stack & Configuratie
- **Framework:** React met Vite.
- **Taal:** TypeScript (strikte typering).
- **Styling:** Tailwind CSS (gebruik native aanvoelende componenten, mobile-first, iOS/Android look).
- **Routing:** `react-router-dom` (met een main layout die een `BottomNav` bevat en detailpagina's met een `ChevronBack` knop).
- **Backend/Database:** Supabase (gebruik `lib/supabase.ts` voor de connectie).
- **Icoon-bibliotheek:** `lucide-react`.
- **Extra libraries:** XLSX of een vergelijkbare library voor het genereren/previewen van Excel bestanden (voor facturatie).

## 2. Applicatie Structuur & Mappen
Zet de volgende mappenstructuur op:
- `/src/components/` (Herbruikbare UI elementen)
- `/src/screens/` (Alle pagina's/views)
- `/src/lib/` (Supabase client, helpers, en dummy data via `data.ts`)
- `/src/types.ts` (Alle TypeScript interfaces en types)

## 3. Database Datamodel (Types / Supabase Tabellen)
Definieer de volgende interfaces in `types.ts` en zet de bijbehorende tabellen op in Supabase:
- **User/Profile:** ID, naam, email, rol (admin, team_drank, standaard), actieve status.
- **Agenda/Event:** ID, titel, datum, tijd, beschrijving, locatie.
- **Consumptie (Strepen):** ID, user_id, drank_id, aantal, datum.
- **Drank/Stock:** ID, naam, prijs, huidige_voorraad, categorie.
- **Factuur (Invoice):** ID, user_id, totaal_bedrag, periode, status (betaald/onbetaald).
- **FrituurBestelling:** ID, user_id, snack_naam, saus, opmerking, status.
- **Quote:** ID, tekst, auteur, datum, upvotes.
- **BierpongMatch:** ID, team1 (users), team2 (users), score1, score2, datum.
- **Notificatie/Bericht:** ID, zender_id, ontvanger_id, titel, bericht, gelezen_status.

## 4. Componenten (Herbruikbaar)
Maak minimaal de volgende componenten:
- **`BottomNav.tsx`:** Een vaste navigatiebalk onderaan het scherm (zoals in iOS/Android apps) met iconen voor: Home, Agenda, Strepen (Bar), en Instellingen.
- **`ChevronBack.tsx`:** Een 'Terug'-knop (pijltje naar links) voor bovenaan geneste schermen om terug te keren naar de vorige route.

## 5. Schermen (Screens) en Functionaliteiten
Bouw de volgende schermen (componenten in de `/src/screens/` map). Zorg voor state management per pagina (of via een global context indien nodig) en koppel deze aan Supabase:

### Algemeen & Authenticatie
- **`CredentialsScreen.tsx`:** Login en registratie pagina. Check credentials via Supabase Auth.
- **`RolesManageScreen.tsx`:** (Alleen voor Admins) Beheer welke gebruikers de rol 'admin', 'team_drank', of 'user' hebben.
- **`SettingsScreen.tsx`:** Persoonlijke instellingen, uitloggen, en links naar persoonlijke facturen.

### Home & Communicatie
- **`HomeScreen.tsx`:** Dashboard met een overzicht (widgets) van aankomende activiteiten, openstaande rekeningen, en snelkoppelingen.
- **`NotificationsScreen.tsx`:** Overzicht van ontvangen pushmeldingen/in-app berichten.
- **`NewMessageScreen.tsx`:** Scherm om een nieuw bericht naar de leidingsploeg of individuen te sturen.
- **`NudgeSelectorScreen.tsx`:** Een scherm om snelle pre-gedefinieerde notificaties ("nudges") te sturen (bijv. "Kom naar het lokaal!", "Vergadering start over 15 min").

### Activiteiten & Eten
- **`AgendaScreen.tsx`:** Lijst van aankomende KSA evenementen en vergaderingen.
- **`AgendaManageScreen.tsx`:** (Admin) Scherm om nieuwe activiteiten toe te voegen, te wijzigen of te verwijderen.
- **`FriesScreen.tsx`:** Scherm waar een gebruiker zijn/haar bestelling voor de frituur kan doorgeven (vleesje, saus, frieten).
- **`FriesOverviewScreen.tsx`:** (Admin/Verantwoordelijke) Een overzichtslijst van alle bestellingen, samengevoegd zodat de besteller makkelijk aan de frituur kan voorlezen wat er nodig is.

### Team Drank (Bar, Strepen & Facturatie)
*Dit is het meest uitgebreide deel van de app. Zorg voor strikte scheiding tussen standaard gebruikers en 'team_drank' beheerders.*
- **`StrepenScreen.tsx`:** De digitale drankkaart. Een interface waar leiding snel "streepjes" (consumpties) kan toevoegen aan hun eigen naam of die van anderen (indien permissie). Moet zeer snel werken (bijv. plus/min knoppen per drankje).
- **`TeamDrankDashboardScreen.tsx`:** (Team Drank) Overzicht van omzet, meest gedronken bieren, en openstaande saldo's.
- **`TeamDrankStockScreen.tsx`:** Beheer van de actuele voorraad en prijzen van de dranken in de frigo.
- **`TeamDrankStreaksScreen.tsx`:** Leuke statistieken/leaderboard over wie de langste "drink streak" heeft of het meeste gedronken heeft.
- **`TeamDrankBillingScreen.tsx`:** Scherm om een nieuwe facturatie-periode af te sluiten. Bereken alle onbetaalde strepen per persoon.
- **`TeamDrankInvoicesScreen.tsx`:** Overzicht van alle gegenereerde facturen per lid. Markeer als betaald/onbetaald.
- **`TeamDrankArchiveScreen.tsx`:** Historiek van oude facturen en afgesloten bar-periodes.
- **`TeamDrankBillingExcelPreviewScreen.tsx` & `TeamDrankExcelPreviewScreen.tsx`:** Schermen die een tabel genereren met alle verbruiken/kosten die rechtstreeks naar `.xlsx` (Excel) geëxporteerd kan worden (voor de penningmeester).

### Persoonlijk Financieel
- **`ConsumptionOverviewScreen.tsx`:** Een gedetailleerde lijst voor de ingelogde gebruiker van al zijn/haar genoteerde consumpties in de huidige periode.
- **`MyInvoiceScreen.tsx`:** Scherm waar de gebruiker zijn eigen openstaande schuld voor de bar kan zien, inclusief een overschrijvingsbericht of QR-code om te betalen.

### Fun & Ontspanning
- **`BierpongScreen.tsx`:** Een tool om de score bij te houden tijdens een potje bierpong (met ELO of simpele ranking).
- **`QuotesScreen.tsx`:** Een "Wall of Fame" waar leiding grappige uitspraken van medeleiding kan toevoegen, met een upvote-systeem.

## 6. Ontwerp & UI/UX Instructies
- Zorg ervoor dat de app eruitziet als een native smartphone app. Verberg de scrollbars waar mogelijk, gebruik voldoende padding, grote 'touch-friendly' knoppen en duidelijke typografie.
- Gebruik voorwaardelijke weergave (conditional rendering): Zorg dat navigatie-items of knoppen naar 'Manage' of 'Team Drank' schermen verborgen zijn voor gebruikers zonder de juiste rol.
- Geef laad-statussen weer (spinners) wanneer data via Supabase wordt opgehaald of weggeschreven.
- Voorzie robuuste foutafhandeling (error toasts) als bijvoorbeeld het toevoegen van een "streepje" mislukt door geen internetverbinding.

Start direct met het schrijven van de `package.json`, de `vite.config.ts`, de Supabase configuratie en werk daarna systematisch scherm per scherm af, beginnend bij Types en Herbruikbare componenten.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SausMeesterXIV)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SausMeesterXIV)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
