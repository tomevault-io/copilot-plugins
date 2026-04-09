## leitnerai-api

> Projekt-Zweck (TL;DR)

Projekt-Zweck (TL;DR)

Diese Repo enthält eine NestJS-API, die als leichtgewichtiger Ingestion-/Control-Layer auf EC2 (hinter einem ALB) läuft.
Die API nimmt PDF-Jobs an und delegiert die Verarbeitung asynchron an einen Worker (ECS Fargate).
Dateien werden direkt vom Client in S3 hochgeladen (presigned URLs), die API verwaltet Jobs/Status über SQS + DynamoDB.

Wichtig: Hier wird nur die API gebaut. Der Worker-Code läuft in einem separaten Service/Repo.

⸻

Architektur (High Level)

Client → POST /upload-url → presigned PUT → S3 (inbox/uploads/…)
Client → POST /extract { jobId, s3Key } → API → DynamoDB(status=queued) + SQS(Message)
Worker (ECS) liest SQS, holt PDF aus S3 inbox, extrahiert Text, schreibt Ergebnis nach S3 results, setzt DDB status=done
Client → GET /status/:jobId (optional GET /download/:jobId → presigned GET)

⸻

Anforderungen an die API (diese Repo)

Endpunkte (müssen existieren)
• POST /upload-url → Body { filename: string }
→ Response { jobId, key, url } (presigned PUT für S3 inbox/uploads/)
• POST /extract → Body { jobId: string, s3Key: string, ocr?: boolean }
→ schreibt DDB {status:'queued'} und sendet SQS-Message {jobId,s3Key,options}
→ Response { jobId } (202 Accepted semantisch, faktisch 200 ist ok)
• GET /status/:jobId → Response { jobId, status, resultKey?, error? }
• GET /download/:jobId → Response { jobId, url } (presigned GET für results/)
• GET /health → { ok: true } (ALB Health Check)

Technische Leitplanken
• Node 18, NestJS, AWS SDK v3 (@aws-sdk/\*)
• Global ValidationPipe (whitelist: true, transform: true)
• Server bindet an 0.0.0.0 (ALB)
• Idempotenz über jobId (Client liefert/erhält die gleiche ID)
• Keine Fremd-HTTP-Downloads in der API (keine SSRF). Es werden nur S3-Keys akzeptiert.
• S3 Prefix-Policy: Nur Pfade unter uploads/ (inbox) und results/ (results) verwenden.

Environment Variablen (müssen gelesen werden)
• AWS_REGION (z. B. eu-north-1)
• INBOX_BUCKET (z. B. leitnerai-inbox-7634-8705-3303)
• RESULTS_BUCKET (z. B. leitnerai-results-7634-8705-3303)
• QUEUE_URL (z. B. https://sqs.eu-north-1.amazonaws.com/763487053303/leitnerai-jobs)
• TABLE_NAME (z. B. leitnerai-jobs)

Diese werden auf EC2 via user_data.sh in /etc/environment gesetzt.

⸻

Implementierungsdetails (erwartete Struktur)

DTOs
• src/dto/upload-url.dto.ts

export class UploadUrlDto { filename: string }

    •	src/dto/extract.dto.ts

export class ExtractDto { jobId: string; s3Key: string; ocr?: boolean; language?: string }

Service-Funktionen (AppService)
• createUploadUrl(filename) → presigned PUT (uploads/<jobId>\_<safeName>.pdf)
• enqueueJob(jobId, s3Key, ocr?, language?)
• HeadObject auf INBOX_BUCKET/s3Key (existiert?)
• DDB PutItem: { jobId, status:'queued', s3Key, createdAt }
• SQS SendMessage: { jobId, s3Key, bucket, resultsBucket, language, options:{ocr, language} }
• getStatus(jobId) → DDB GetItem → { status, resultKey?, error? }
• getResultDownloadUrl(jobId) → presigned GET für RESULTS_BUCKET/resultKey
• getAssetSignedUrl(key) → presigned GET für RESULTS_BUCKET/key (nur results/*)
• health() → { ok: true }
• Bestehende Demo-Route /host bleibt zu Debugzwecken erhalten.

Controller (AppController)
• Obige Routes exakt abbilden; 400 werfen bei fehlenden/invaliden Params.
• GET /asset?key=results/... → signierten Download zurückgeben

⸻

IAM (Erwartungen an Instanzrolle – Info für Dev)

Die EC2-Instanzrolle der API braucht:
• sqs:SendMessage auf leitnerai-jobs
• dynamodb:PutItem/GetItem/UpdateItem auf leitnerai-jobs (Table)
• s3:HeadObject (und optional GetObject) auf **INBOX_BUCKET/\*`
• s3:GetObject auf **RESULTS_BUCKET/\*(für/download`)

Die Worker-Rolle ist nicht Teil dieser Repo.

⸻

Deployment (EC2/ALB via Launch Template)
• user_data.sh erledigt Node 18, npm ci && npm run build && npm prune --production, PM2 Setup.
• Nach Änderungen am LT: ASG → Launch Template Version = Latest setzen, dann Instance Refresh (Warmup ~300s).
• ALB Health Check zeigt auf HTTP /health (200–399).
• Security Groups: EC2 3000/tcp nur von ALB-SG; ALB 80/tcp von 0.0.0.0/0.

⸻

Entwicklungs-Tasks für die KI (konkret) 1. API-Endpunkte implementieren
• DTOs + Validation
• Service-Methoden mit AWS SDK v3
• Controller-Routen gemäß oben
• main.ts: ValidationPipe + listen(3000,'0.0.0.0') 2. Fehlerbehandlung
• Bei HeadObject 404 → BadRequestException("Object not found")
• Unerwartete AWS-Fehler → 500 loggen (keine sensiblen Daten im Body)
• /download/:jobId nur bei status='done' + vorhandenem resultKey 3. Sicherheit
• s3Key muss mit uploads/ beginnen
• Keine Weiterleitung an externe URLs
• Response keine internen Pfade/Stacktraces 4. Tests (leicht)
• Unit-Test für Controller-Service-Verkabelung (Mock-Clients)
• E2E-Test für /health und / (Hello World) 5. Docs (README)
• Kurzer Abschnitt „API Routes & Flow“ (Copy der Ziele + cURL-Beispiele)

⸻

Akzeptanzkriterien (Definition of Done)
• POST /upload-url liefert {jobId, key, url} (URL 15 Min gültig, Content-Type application/pdf)
• POST /extract schreibt DDB-Item (queued) und sendet SQS-Message
• GET /status/:jobId gibt queued|processing|done|failed zurück
• GET /download/:jobId gibt presigned GET-URL nur bei done
• GET /health → { ok: true }
• API lauscht auf 0.0.0.0:3000
• Keine devDependencies zur Laufzeit erforderlich (build + npm prune --production)
• Code lintet & baut: npm run build ok

⸻

Lokale Entwicklung (Schnellstart)

npm ci
npm run start:dev

# ENV lokal per .env oder Shell setzen (für AWS-Aufrufe lokale creds nötig)

Für lokale Tests ohne AWS: Service-Clients mocken (InMemory-Stubs).

⸻

Später (außerhalb dieses Repos)
• Worker-Service (ECS): SQS → S3(inbox) lesen → extrahieren → S3(results) schreiben → DDB status='done' + resultKey
• Auto Scaling: ECS-Service nach Queue-Länge
• S3 Lifecycle: inbox/ nach 7–30 Tagen löschen; results/ nach Bedarf
• Monitoring: Alarme auf DLQ > 0, 5xx-Rate, ECS TaskFailures

⸻

Kontaktpunkte im Code (heute vorhanden)
• src/app.service.ts → wird erweitert um AWS-Methoden
• src/app.controller.ts → wird um neue Routen ergänzt
• src/main.ts → ValidationPipe + 0.0.0.0
• user_data.sh → belassen (Build-Flow ok)

⸻

Bitte beim Implementieren Platzhalter (<ACCOUNT_ID>, Bucket-Namen) nicht in den Code hardcoden – alles über ENV beziehen.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/danielba777)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/danielba777)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
