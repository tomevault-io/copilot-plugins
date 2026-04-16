## project-templates

> Läs och uppdatera case-journal / troubleshooting-log för projektkontext


# Projektkontext: case-journal och troubleshooting-log

## Vid start i ett nytt projekt (viktigt)

- **Först:** Kolla om **case-journal.md** eller **troubleshooting-log.md** finns i projektet (root eller `docs/`).
- **Om ingen av filerna finns:** Fråga användaren innan du sätter igång med annat: *"Jag ser att det inte finns någon case-journal.md eller troubleshooting-log.md i det här projektet. Vill du köra en fork av project-templates-repot (https://github.com/Ridvan-bot/project-templates) och lägga in rätt mall innan vi sätter igång?"* Vänta på svar – om användaren vill forka, hjälp till med det; om inte, gå vidare med användarens övriga fråga.
- **Om minst en fil finns:** Fortsätt enligt "I början av arbete" nedan.

## Vilka filer

- **case-journal.md** – används när projektet handlar om att **sätta upp nya saker, uppgraderingar eller ändringar i en fungerande miljö**. Innehåll: miljö, mål, planerade ändringar, genomfört, status, nästa steg.
- **troubleshooting-log.md** – används när du **felsöker något**. Innehåll: problem, felmeddelanden, hypoteser, experiment, resultat, öppna frågor. All info ska löpande läggas in här under felsökningens gång.

Sök filerna i reporoten eller under `docs/`.

## I början av arbete

- Om projektet innehåller **case-journal.md** eller **troubleshooting-log.md**: läs relevant fil(er) först för att få kontext (snabbstatus, miljö, vad som är gjort, nästa steg).
- Avgör utifrån innehåll och användarens fråga om det är ett setup/uppgraderingsprojekt (case-journal) eller felsökning (troubleshooting-log).

## Under och efter arbete

- **Setup/uppgradering/ändring:** Efter genomfört arbete föreslå eller gör uppdateringar i **case-journal.md**, särskilt:
  - "0) Snabbstatus" (status, senast ändrat, nästa steg)
  - "3) Genomfört" (nytt block för vad vi gjort)
  - "5) Öppna punkter" om något nytt väntar eller är oklart.
- **Felsökning:** Under och efter felsökning uppdatera **troubleshooting-log.md**, särskilt:
  - "0) Quick status" (nuvarande problem, nästa steg)
  - "4) Experiments and results" (nytt block per test/försök)
  - "5) Known facts" och "6) Open questions" när något blivit klart eller nytt frågetecken.
- Skriv kort och konkret. Inga hemligheter. Vid osäkerhet: föreslå text så användaren kan granska.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ridvan-bot) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
