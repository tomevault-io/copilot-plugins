## kalebi

> ეს ფაილი Claude Code-ს (claude.ai/code) აძლევს კონტექსტს ამ repo-ში მუშაობისას.

# CLAUDE.md

ეს ფაილი Claude Code-ს (claude.ai/code) აძლევს კონტექსტს ამ repo-ში მუშაობისას.

## პროექტი

**Cycle Care** (მარკეტინგული სახელი "Qalis Sivrce") — ქართულენოვანი (ka-GE) მენსტრუალური ციკლის თრექინგის Expo/React Native აპი, პარალელური ორსულობის რეჟიმით და OpenAI-ზე დაფუძნებული AI ასისტენტით. Bundle ID `com.mishvela.kalebi`, EAS slug `kalebi`.

Backend: Supabase (Auth, DB, Edge Functions). AI: OpenAI-ს იძახებს მხოლოდ `supabase/functions/ai-assistant/index.ts` — OpenAI key ყოველთვის სერვერზეა, არასდროს frontend-ში (იხ. `AI_SETUP.md`).

---

## ბოლოს რა გავაკეთეთ (ბოლო სესიის snapshot)

> ეს სექცია ყოველთვის უნდა აჩვენებდეს ყველაზე ახალ მდგომარეობას — ახალი სესიის დაწყებისას აქედან დაიწყე, `git log`/`git status`-ის თავიდან აწარმოების ნაცვლად. საჭიროებისამებრ განაახლე.

**🔐 AI ბიუჯეტი გამოწერას მიჰყვება + `premium_override` დაიბლოკა (2026-08-01 — committed `f7d06fc`, pushed, **migration + edge function deployed ✅**)**: აუდიტის ბოლო ორი ხვრელი. ⚠️ **OTA არ სჭირდება** — ორივე სერვერზეა.
- **`ai-assistant/index.ts`** (**deployed ✅**): chat-ის ლიმიტი ადრე მხოლოდ `pregnancy_mode`-ს უყურებდა → ვადაგასულ გამომწერელს (ან ვინც დროშას თვითონ ჩაიწერდა) **10 კითხვა რჩებოდა 1-ის ნაცვლად** = რეალური OpenAI-ს ხარჯი. ახალი `hasPregnancyAccess()` (`resolvePregnancyAccessFromProfile`-ის სარკე) + `pregnancyActive = pregnancy_mode && hasPregnancyAccess(profile)`. select-ს დაემატა `has_pregnancy_subscription, pregnancy_until`. fail-open ქცევა profile-ის წაკითხვის შეცდომაზე **შენარჩუნებულია**.
- **migration `20260801_lock_premium_override.sql`** (**remote ✅**): 🛡️ trigger `guard_premium_override` on `profiles` (BEFORE INSERT OR UPDATE, SECURITY DEFINER). მიზეზი: RLS **სვეტს ვერ ზღუდავს**, `premium_override`-ს კი `ThemeContext` **არასდროს ამოწმებს** RevenueCat-თან (ადრევე ჩერდება) → user-ს საკუთარ row-ში ჩაწერით **სამუდამო Prime** შეეძლო. ახლა: `auth.uid() is null` (service_role/migration/SQL editor) → გაატარებს; admin-ის JWT email → გაატარებს; სხვა შემთხვევაში UPDATE-ზე **`raise exception` (42501)**, INSERT-ზე `false`-ზე აიძულებს.
- ⚠️ **მხოლოდ ეს ერთი სვეტია დაბლოკილი.** `is_premium`/`has_pregnancy_subscription` კლიენტიდან უნდა იწერებოდეს (purchases.js), და ისინი ისედაც **ყოველ გახსნაზე გადაიწერება** store-იდან, ე.ი. თვით-მინიჭება ვერ გადარჩება. სრული დაბლოკვა მოითხოვს ჩაწერის სერვერზე გადატანას — ცალკე სამუშაოა.
- ✅ **ტესტირებული**: 12/12 — რეალური გამომწერელი, legacy `until=null`, გაუქმებული ფასიან პერიოდში → **ბიუჯეტი უცვლელი**; ვადაგასული, თვით-ჩაწერილი, გატეხილი თარიღი → **1/დღეში**. Prime ორსულობას ჯობნის, `premium_override` ისევ მუშაობს.
- ✅ **ხელით გადამოწმებულია live-ზე (user-მა, 2026-08-01)**: admin პანელის „Admin Prime" გადამრთველი მუშაობს (trigger admin-ის JWT-ს ატარებს) და AI ჩატიც პასუხობს.
- ↩️ **rollback**: `drop trigger guard_premium_override on public.profiles;`

**🎯 guard დავიწროვდა — `admin_grant` = ხელით მიცემული წვდომა (2026-08-01 — committed `a38e169`, pushed, ⚠️ OTA ჯერ არ გასულა)**: ⛔ **მთავარი კონტექსტი: Android საერთოდ არ არის გაშვებული — აპი მხოლოდ iOS-ზეა.** ამიტომ ეტაპ 1-ის guard-ი ("დაიცავი ყველაფერი, რასაც RevenueCat ვერ ხედავს") აზრს კარგავდა: Android-ის მყიდველები არ არსებობენ, ხოლო `profiles`-ზე user-ს **საკუთარი row-ის სრული UPDATE უფლება აქვს** (RLS გადამოწმებულია — `Users can update their profile`, სვეტების შეზღუდვის გარეშე), ე.ი. უწყარო `has_pregnancy_subscription=true` თვითონვე შეიძლება ჩაეწერა.
- **`purchases.js`** — ახალი `MANUAL_PREGNANCY_GRANT_SOURCES = ["admin_grant"]`. guard ახლა **მხოლოდ** `pregnancy_source = 'admin_grant'`-ს იცავს; ყველა დანარჩენი (`null`-ის ჩათვლით) ჩვეულებრივ ექვემდებარება RevenueCat-ის პასუხს.
- 💡 **ახალი შესაძლებლობა**: წვდომის ხელით მიცემა **ახლა მუშაობს** (ადრე აპი წაშლიდა — იხ. ძველი გაკვეთილი ქვემოთ): `update profiles set has_pregnancy_subscription=true, pregnancy_source='admin_grant', pregnancy_until='2026-12-31' where email='...'` (`pregnancy_until=null` = უვადო). ვადა მაინც მოქმედებს — გასულ `admin_grant`-საც წვდომა ეკეტება.
- 📊 **გადამოწმებულია რეალურ მონაცემებზე** (RevenueCat CSV export × `profiles`): **16 ორსულობის გამომწერელი** (+9 Pro = 25), **ყველა 16 ემთხვევა profile row-ს `app_user_id`-ით** → არცერთი მათგანი guard-ზე არ არის დამოკიდებული (აქტიური entitlement მიცემის გზაზე მიდის, სადაც guard არ ერევა). ბაზაში დროშა **21**-ს ჰქონდა → **5 არ იხდის**, აქედან **3 დეველოპერის სატესტოა** (`isTestAccountEmail`-ით ისედაც აქვთ). 2 რეალურ user-ს user-ის გადაწყვეტილებით წვდომა ეხსნება.
- ⚠️ **11 გადამხდელს row ჯერ არ ჰქონდა დამარკირებული** (`pregnancy_source=null`) — ეს ეტაპ 1-მდე არსებული სინქრონის დანაკლისია; აპის გახსნისთანავე თავისით სწორდება.
- ✅ **ეტაპი 1-ის დადასტურება ველზე**: export-ში ერთი გამომწერელი 21 ივლისს გაუქმებულია, წვდომა 9 აგვისტოს 18:15-ზე ეწურება — ბაზის `pregnancy_until` **წამში ემთხვევა**. ძველი კოდით სამუდამოდ შეუნარჩუნდებოდა.
- ✅ 9/9 ახალი ტესტი (რეალური სცენარებიდან) + წინა 29 რეგრესიაზე.
- ⚠️ **დარჩენილი ნაპრალი**: ვინც გააუქმებს და აპს **აღარასდროს გახსნის**, row არ განახლდება. მხოლოდ webhook-ით წყდება — 16 გამომწერელზე რისკი მცირეა.

**🧾 გადახდის ისტორია + ანგარიშის გამოცვლა (2026-08-01 — committed `286bb16`, pushed, ⚠️ OTA ჯერ არ გასულა)**: აუდიტის "ეტაპები 3-4", ორივე პატარა.
- **`purchases.js`** — `writePremiumStatusToProfile`/`writePregnancyStatusToProfile` ადრე **ყოველ** "წვდომა არ აქვს" refresh-ზე `*_plan`/`*_source`/`*_last_payment_at`/`*_order_id`-ს **null-ავდა** → ვადის გასვლისთანავე იკარგებოდა, რა და როდის იყიდა. ახლა ეს 4 ველი **მხოლოდ იწერება, არასდროს იშლება** (იგივე pattern, რაც `pregnancy_purchase_context`-ს უკვე ჰქონდა). 💡 **ბონუსი**: შენარჩუნებული `pregnancy_source` ეტაპი 1-ის guard-საც აძლიერებს — მუდმივი კვალია, რომ ამ user-ის წვდომა RevenueCat-იდან მოვიდა.
- **`PregnancyContext` + `FertilityContext`** — ორივეს დაემატა `supabase.auth.onAuthStateChange` (`ThemeContext`-ს ეს ჰქონდა, ამ ორს — არა): გასვლაზე state ნულდება, შესვლაზე იტვირთება. ⚠️ **კრიტიკული დეტალი**: listener ჯერ **user id-ს ადარებს** (`loadedUserIdRef`) — `TOKEN_REFRESHED`/`INITIAL_SESSION` იმავე event-ზე მოდის და ყოველ refresh-ზე გასუფთავება **ორსულობის ეკრანებს remount-ს გაუკეთებდა** (`pregnancyMode` წყვეტს რომელი ეკრანი დაიხატოს).
- ✅ **ტესტირებული**: 18/18 pass — ისტორია lapse-ზე რჩება, renewal-ზე ახლდება, უფასო user-ს ცრუ ისტორია არ უჩნდება; listener token refresh-ზე **არაფერს** აკეთებს, გასვლაზე ასუფთავებს, ანგარიშის გამოცვლაზე თავიდან ტვირთავს. + ეტაპი 1-ის 18 ტესტი რეგრესიაზე — გავიდა.

**🧾 გამოწერის მართვა + გაუქმების ახსნა (2026-08-01 — committed `ac11be0`, pushed, ⚠️ OTA ჯერ არ გასულა)**: გადახდების აუდიტის "ეტაპი 2". აპში **არსად** არ იყო გამოწერამდე მისასვლელი გზა (Apple-ის 3.1.2 ითხოვს), რეჟიმის გამორთვა კი გაუქმებად აღიქმებოდა, თუმცა ფული ჩამოდიოდა.
- **`purchases.js`** — ახალი `openManageSubscriptions()` (`https://apps.apple.com/account/subscriptions`) + `canOpenManageSubscriptions()`. ⚠️ **მხოლოდ iOS**: Android გარე web-checkout-ზეა, ე.ი. Play-ის გამოწერების გვერდი ცარიელი იქნებოდა და მოატყუებდა.
- **`premium.jsx` / `pregnancy-premium.jsx`** — "გამოწერის მართვა / გაუქმება" ღილაკი restore-ის ქვემოთ (`manageButton` style). **`profile.js`** — "აპლიკაცია" ბლოკში 🧾 row (Prime-ის შემდეგ).
- **გამორთვის ორივე Alert** (`handlePregnancyDisable`, `handleFertilityDisable`) — ახლა წერია: (ა) რეჟიმის გამორთვა **გამოწერას არ აუქმებს**, (ბ) სად გააუქმოს (`CANCEL_HINT`, platform-ის მიხედვით), (გ) **ერთი გამოწერა ორივე რეჟიმს ხსნის** (`SHARED_SUBSCRIPTION_HINT`) — გაუქმება ორივეს დახურავს.
- **`PregnancyContext`** — ახალი `accessLapsed` (`pregnancy_mode && !access`). **`index.js`** — home-ზე ბარათი "ორსულობის გამოწერა დასრულდა" (ჩანაწერები დაცულია, ტაპი → paywall), light/dark ორივეზე. ⚠️ ჩვეულებრივი ორსულობის ბანერს დაემატა `&& !accessLapsed` — ორი ბარათი ერთად არ ჩანს.

**🔒 გაუქმებული ორსულობის გამოწერა აღარ ინარჩუნებს წვდომას (2026-08-01 — committed `77aeedf`, pushed, ⚠️ OTA ჯერ არ გასულა)**: გადახდების აუდიტის შემდეგ, "ეტაპი 1". მიზეზი: გამოწერა ფონზე **არასდროს** მოწმდებოდა (Prime-ს ეს აქვს, ორსულობას — არა) და `pregnancy_mode` თავად ითვლებოდა გადახდის დადასტურებად.
- **`PregnancyContext.js`** — `loadPregnancyData` ახლა **iOS-ზე ჯერ `checkPregnancySubscriptionStatus()`-ს იძახებს** (try/catch, offline-ზე ძველ მონაცემებზე ვარდება), მერე კითხულობს profile-ს. `access = admin || testAccount || resolvePregnancyAccessFromProfile`; `pregnancyMode = pregnancy_mode && access`. ⚠️ **`|| nextPregnancyMode` წაშლილია** — ეს იყო უფასო წვდომის მთავარი ხვრელი. ორი ურთიერთსაწინააღმდეგო write-back (რაც ყოველ foreground-ზე flag-ს არყევდა true↔false) წაშლილია — ჩაწერას მხოლოდ `purchases.js` აკეთებს.
- **`enablePregnancyMode`/`disablePregnancyMode`** აღარ წერენ `has_pregnancy_subscription`-ს. ⚠️ ბაზის **`pregnancy_mode` სვეტს არ ვეხებით** — გამოწერის განახლებაზე ყველაფერი მონაცემებთან ერთად ბრუნდება.
- **`profile.js`** — `handlePregnancyEnable`-ის `!configured` ტოტი აღარ რთავს რეჟიმს უფასოდ ("დროებით მიუწვდომელია", fertility-ის pattern). ტესტირება admin/test ანგარიშზეა (`freeModeAccess`).
- **`purchases.js`** — (ა) `getPrime/PregnancyExpirationDate`-ს წაერთვა `|| customerInfo.latestExpirationDate` fallback (ეს **ყველა პროდუქტის** უახლესი ვადაა → Prime-ის ვადა `pregnancy_until`-ში ჟონავდა). (ბ) 🛡️ **`writePregnancyStatusToProfile` არ ართმევს წვდომას, რომელსაც ვერ ამოწმებს**: `hasSub=false`-ზე ჯერ კითხულობს `pregnancy_source`-ს და თუ ის **არ იწყება `revenuecat`-ით**, ჩაწერას **გამოტოვებს** (Android web-checkout და ძველი ჩანაწერები RevenueCat-ს არ უჩანს → "no entitlement" ≠ "გააუქმა"). მიცემა ყოველთვის ნებადართულია — ე.ი. გადამხდელი, ვისაც ჩაწერა ჩავარდნია, **ავტომატურად სწორდება**.
- **`assistantOrchestrator.js`** — ერთი საერთო `pregnancyAccess`/`pregnancyModeActive` (ეკრანების იდენტური); `pregnancy_until` დაემატა select-ს. + **ძველი ბაგი გასწორდა**: `fertilityUnlocked` `has_pregnancy_subscription`-ს **ვადის გარეშე** ამოწმებდა.
- **`notifications.js`** — `syncCycleRemindersForUser`-ის pregnancy ტოტსაც იგივე access წესი დაედო.
- ✅ **ტესტირებული**: 18/18 pass (11 access truth-table + 7 guard) — ცოცხალი/გაუქმებული/ვადაგასული/legacy `until=null`/lifetime/admin/test/გატეხილი თარიღი. ქცევა იცვლება **მხოლოდ 4 სცენარზე** და ოთხივე ხვრელია; მოქმედი გამომწერელი არ ზარალდება.
- 📊 **ბაზის მდგომარეობა commit-ის დროს**: `has_pregnancy_subscription=true` → **21**, აქედან **16-ს ვადა არ უწერია** და **15-ს წყარო არ უწერია** (= ძველი `enablePregnancyMode`-ის ჩანაწერები; `pregnancy_mode=true` სწორედ 15-ია). RevenueCat: 25 paid subscriber, მაგრამ **Prime+pregnancy ერთად** — დაშლილი არ არის. სწორედ ამ გაურკვევლობის გამო დაემატა guard-ი.
- **დარჩა**: `ai-assistant` edge function ჯერ კიდევ მხოლოდ `pregnancy_mode` სვეტს უყურებს ლიმიტისთვის (გაუქმებულს 10 კითხვა რჩება 1-ის ნაცვლად) — სერვერის deploy-ია, ცალკე. "გამოწერა ამოიწურა" ეკრანი — ეტაპი 2.
- ⚠️ **OTA ვერ გავუშვი**: `app.json` owner = **`qalebi`**, CLI კი `mishvelidze.u@gmail.com`-ზეა (accounts: `mishvela`, `horoskopi`) → `Entity not authorized`. საჭიროა `eas login` სწორი ანგარიშით ან `qalebi`-ში მოწვევა.

**🧠 ორსულობის ასისტენტს დაემატა გრძელვადიანი მეხსიერება (2026-07-11 — committed `6175dab`, pushed, OTA production ✅ update group `62737a09`)**: user-ის კითხვიდან — "9 თვეზეა საუბარი, რა ჭირდა და რა პრობლემა ჰქონდა უნდა ახსოვდეს".
- **ადრე როგორ იყო**: ჩატს **მეხსიერება საერთოდ არ ჰქონდა** — `messages` მხოლოდ `useState`-ში (აპის დახურვა = სუფთა ფურცელი), AI-ს ბოლო **6 შეტყობინება** მიდიოდა (`history.slice(-6)`), `assistant_chat_history` ცხრილში მხოლოდ **ჩაწერა** ხდებოდა და **არსად იკითხებოდა** (grep-ით დადასტურებული — 0 `select`).
- 💡 **მთავარი ხედვა**: ახალი ინფრასტრუქტურა არ დასჭირდა — **მონაცემები უკვე გვქონდა**. `symptoms` ცხრილი მთელ ორსულობას ინახავს, უბრალოდ ასისტენტი მხოლოდ ბოლო 10 დღეს კითხულობდა. საჭირო იყო **წაკითხვა + შეჯამება**, არა შენახვა.
- **`utils/pregnancyMemory.js`** (ახალი, სუფთა ლოგიკა): `summarizePregnancySymptomHistory` (სიმპტომი → რამდენჯერ + **ორსულობის კვირების დიაპაზონი**, მაგ. "გულისრევა · 7-ჯერ · კვირა 6-12"; top-8 სიხშირით; ქართული label-ები), `collectPregnancyNotes` (მისივე სიტყვებით დაწერილი კომენტარები კვირით, ბოლო 5), `collectRecentQuestionTopics` (ბოლო 12 კითხვის თემა, transcript-ები არა), `buildPregnancyMemory` (ცარიელზე **null** → context-იდან საერთოდ ვარდება).
- **`assistantOrchestrator.js`**: `getAssistantContext`-ში — **მხოლოდ `pregnancy_mode`-ზე** — მთელი ორსულობის `symptoms` (`gte pregnancy_start_date`, limit 300) + ბოლო 15 კითხვა `assistant_chat_history`-დან; try/catch-ში (base context ვერასდროს გატყდება). `context.pregnancyMemory` ჩატში `...context`-ით სრულად მიდის (**გადამოწმებული** — გუშინდელი `sendContext`-ის ბაგის გამო).
- 🛡️ **Medical safety prompt-ში** (`# LONG-TERM MEMORY`): ძველი სიმპტომი **წარსულად** ახსენოს, არა დღევანდელად; **აკრძალულია** ტრენდების ანალიზი "თითქოს ჯანმრთელობას აკვირდება" ("your blood-pressure history concerns me" პირდაპირ აკრძალულია); ეს **სითბოსა და უწყვეტობის მეხსიერებაა, არა სამედიცინო ბარათი**; საეჭვო პატერნზე — ექიმთან მიმართვის რჩევა.
- ✅ **ტესტირებული**: 21/21 pass რეალურ 9-თვიან სცენარზე (გულისრევა I ტრიმესტრში → "კვირა 6-12", ზურგის ტკივილი 20-30, კომენტარები კვირებით, სიხშირით დალაგება, LMP-მდე თარიღები კვირა 1-ზე clamp-დება, null/ცარიელი safety).
- **დარჩა (არ გამიკეთებია)**: ჩატის სესიის **აღდგენა** აპის გახსნაზე (`messages` კვლავ იშლება) — ცალკე ფუნქციაა; ეს commit მხოლოდ **ფაქტების** მეხსიერებაა, როგორც user-მა სთხოვა.

**🐛 რეჟიმის გადართვა რესტარტს ითხოვდა — გასწორდა (2026-07-11 — committed `3a2c4ef`, pushed, OTA production ✅ update group `12e8d6a8`)**: user-ი იტყობინებოდა, რომ "მინდა დაორსულება"-ს ჩართვა/გამორთვა **მხოლოდ აპის ხელახლა გაშვების შემდეგ** მოქმედებდა.
- **მიზეზი**: `FertilityContext` profile-ს კითხულობს **მხოლოდ mount-ზე და AppState foreground-ზე**. `profile.js` კი `useFertility()`-ს **საერთოდ არ იყენებდა** — goal-ს ბაზაში წერდა, მაგრამ context-ს ვერავინ ატყობინებდა. (`PregnancyContext`-ს ეს ჰქონდა — `reloadPregnancy` ყველგან იძახებოდა; fertility-ს დამატებისას გამომრჩა.)
- **fix**: `profile.js`-ს დაემატა `const { reload: reloadFertility } = useFertility()` და `await reloadFertility()` **4 წერტილში**: `updateGoalMode` (ერთი ცვლილება ფარავს fertility-ის ჩართვასაც და გამორთვასაც — ორივე ამ ფუნქციაზე გადის), ორსულობის ჩართვა, ორსულობის გამორთვა, Android web-checkout-იდან დაბრუნება.
- ⚠️ **ორსულობის გზებიც გასწორდა, არა მარტო fertility-ის**: `fertilityMode = goal==="დაორსულება" && access && !pregnancy_mode` — ანუ ორსულობის ჩართვა/გამორთვაც ცვლის fertility-ის მდგომარეობას. `statistics.js`-ის "ორსულად ვარ" ღილაკს ეს **უკვე სწორად ჰქონდა** (ფაზა 6-იდან).

**🔑 `TEST_ACCOUNT_EMAILS` — სატესტო ანგარიში ≠ admin (2026-07-11 — committed `6e316b1` + Prime switch `4a199d5`, pushed, OTA production ✅ update group `a4d1a749`)**: `u@gmail.com`-ს **მხოლოდ ფასიანი რეჟიმები ეხსნება უფასოდ**, სხვა ყველაფერში ჩვეულებრივი user-ია.
- ⛔ **ჯერ ADMIN_EMAILS-ში დავამატე (commit `7fde48f`) — ეს შეცდომა იყო და მაშინვე გასწორდა**: `_layout.tsx:231-264` admin-ს **მალავს** home/calendar/symptoms/statistics tab-ებს (`href: isAdmin ? null : undefined`) და აპს admin dashboard-ზე ხსნის (`initialRouteName`) → fertility-ს ტესტირება **შეუძლებელი** გახდა. **ნუ გამოიყენებ admin-ს სატესტო წვდომისთვის.**
- **სწორი მექანიზმი**: `services/adminAccess.js` → ახალი `TEST_ACCOUNT_EMAILS` + `isTestAccountEmail()`, ცალკე ADMIN_EMAILS-ისგან (ტესტით გადამოწმებული — სიები არ იკვეთება). `ThemeContext`-ს დაემატა `isTestAccount`. ყველა ფასიანი gate განახლდა: `profile.js` (`freeModeAccess` — fertility row, `handleFertilityEnable`, `handlePregnancyEnable`, `handlePregnancyEntryPress`), `index.js` (home strip), `FertilityContext`, `assistantOrchestrator` (AI persona), `notifications` (fertility შეხსენებები). **`isAdmin` დარჩა მხოლოდ admin UI-ზე** (პანელის ლინკი, tab-ები).
- 🧪 **Prime გადამრთველი (2026-07-11, user-ის მოთხოვნით)**: პროფილში ახალი "ტესტირება" სექცია (**მხოლოდ `isTestAccount`-ს უჩანს**) — Switch-ით Free ↔ Prime ნებისმიერ დროს. `ThemeContext`: `testPrimeEnabled` + `setTestPrimeEnabled`, ინახება **AsyncStorage**-ში (`cycle_app_test_prime_enabled`) — ⚠️ განზრახ ლოკალურად, რადგან (ა) ბაზის დროშას RevenueCat-ის sync გადააწერდა, (ბ) რეალურ user-ებზე ვერანაირად ვერ იმოქმედებს. `checkPremiumStatus`-ში test-branch **RevenueCat sync-ამდე `return`-ს აკეთებს**, ამიტომ foreground/AppState refresh switch-ს არ ცვლის. `setTestPrimeEnabled` არა-სატესტო ანგარიშზე **no-op-ია** (დაცვა — შემთხვევითი გამოძახებით ვერავინ მიიღებს უფასო Prime-ს).
- **პრობლემა, რამაც ეს გამოიწვია**: user-მა ბაზაში ხელით დააყენა `has_pregnancy_subscription=true` fertility-ს გასატესტად; მეორე დღეს გამოირთო და Prime ჩანდა.
- ⚠️ **მიზეზი (მნიშვნელოვანი გაკვეთილი)**: **iOS-ზე RevenueCat არის სიმართლის წყარო, არა ბაზა.** `checkPregnancySubscriptionStatus()` ([purchases.js:545](../women-calendari/cycle-app/services/purchases.js#L545)) RevenueCat-ს ეკითხება და პასუხს **ბაზაში გადააწერს** — რეალური `pregnancy` entitlement-ის გარეშე `has_pregnancy_subscription=false` ჩაიწერა და ხელით დაყენებული დროშა წაიშალა. იგივე ხდება Prime-ზე: `syncPremiumStatusFromPurchases()` **ყოველ გახსნაზე** ეშვება (`ThemeContext.js:83`) — sandbox-ში ნაყიდი $1 Prime RevenueCat-ს ახსოვს, ამიტომ `is_premium=true` დაბრუნდა. **დასკვნა: iOS-ზე subscription-დროშის ხელით ჩაწერა ბაზაში უაზროა — აპი წაშლის.** ⚠️ **განახლდა 2026-08-01 (`a38e169`)**: ორსულობაზე ეს **აღარ არის სიმართლე** — `pregnancy_source='admin_grant'`-ით ჩაწერილ წვდომას აპი პატივს სცემს (იხ. ზემოთ). **Prime-ზე კვლავ ძალაშია** — იქ ხელით მისაცემად `premium_override=true` გამოიყენე.
- 💡 რატომ კოდში და არა ბაზაში: `isTestAccountEmail` კოდშია, ამიტომ RevenueCat-ის sync ვერ გადააწერს. DB RLS policy-ებს **არ შევხებივარ** — სატესტო ანგარიშს სხვისი მონაცემები არ სჭირდება; ai-assistant edge function-ის ცალკე ADMIN_EMAILS-საც არა (AI ლიმიტები ჩვეულებრივია, ეს შეგნებულია).
- 💡 რისკი დაბალია: `u@gmail.com` ანგარიში **უკვე არსებობს** (user-ისაა) → Supabase დუბლიკატს არ დაუშვებს. Gmail-ზეც ვერ დარეგისტრირდებოდა (6 სიმბოლოზე ნაკლები).

**🐛 3 ბაგი გასწორდა fertility რჩევაში (2026-07-11, code review — committed `6b7fcbe`, pushed, OTA production ✅ update group `c36a8c5f`)**: user-მა სთხოვა გადამემოწმებინა — სამივე რეალური აღმოჩნდა და სამივე ჩემი წინა commit-იდან იყო.
1. **ცარიელი ბარათი**: `getDiaryAssistantSupport` `text:""`-ს აბრუნებდა, თუ entry-ში symptoms/mood/note არ იყო, ჩემი ავტო-trigger კი **log-ებზე** ეშვებოდა → მარტო LH ტესტის მონიშვნაზე loader აციმციმდებოდა და ბარათი ქრებოდა. **fix**: `hasFertilitySignals` — fertility რეჟიმში ჩაწერილი სიგნალი თავისთავად კონტენტია.
2. **⚠️ რჩევა სიგნალებს ვერ ხედავდა** — CLAUDE.md-ში ჩემი წინა ჩანაწერი (*"fertilityTracking-ის წყალობით LH/BBT-საც ითვალისწინებს, გატარება არ დასჭირდა"*) **მცდარი იყო**: `sendContext` შეგნებულად მინიმალურია (`user_name/phase/cycle_day/symptoms/mood/note`) — `fertilityTracking` იქ **არ მიდიოდა**, prompt-იც ზოგადი ფსიქოლოგიური მხარდაჭერა იყო. **fix**: ცალკე `fertilityPrompt` (LH surge → ოვულაცია 24-36სთ, არ დაპირდეს ჩასახვას, hedge როცა ბუნდოვანია) + `sendContext`-ს დაემატა `fertility_today/days_to_ovulation/best_ovulation_estimate/confirmed_ovulation/cycle_regularity`.
3. **AI ბიუჯეტს წვავდა გახსნაზე**: `dayLogs` mount-ზე `{}`→DB-ის მონაცემებზე გადადიოდა, `loggedSignalsKey` იცვლებოდა და trigger ეშვებოდა — ანუ კალენდრის **უბრალო გახსნა** (თუ დღეს რამე ჩაწერილია) 1 AI გამოძახება, ყოველ ვიზიტზე (cap 30/დღე). **fix**: `hasUserEditedRef` — რჩევა მხოლოდ **რეალურ ცვლილებაზე** იძახება; დღის გადართვა ref-ს ანულებს.
- ✅ **ტესტირებული**: 16/16 pass — client-ის trigger და server-ის content-წესი **ზუსტად ემთხვევა** ყველა 6 log ტიპზე (სწორედ ამ შეუსაბამობამ დაბადა #1); ცარიელი დღე არცერთს არ ააქტიურებს; `bbt 36.0` ითვლება (`!= null`, არა falsy); ცარიელი supplement მასივი — არა.
- ✅ **რეგრესია არ არის**: არა-fertility user-ს `context.fertilityTracking` არ აქვს → `fertility = null` → gate/prompt/sendContext ზუსტად ძველია.

**fertility კალენდარს დაემატა კომენტარი + ავტომატური AI რჩევა (2026-07-11 — committed `630b736`, pushed, OTA production ✅ update group `670483c5`)**: user-ის მოთხოვნით — ისევე როგორც სხვა რეჟიმების კალენდარს აქვს.
- 📝 **კომენტარი**: `note` state + TextInput + "შენახვა" ღილაკი. ინახება **საერთო `symptoms` ცხრილში** (`user_id,date`), არა `fertility_logs`-ში — ასე ასისტენტი მას თავის ჩვეულებრივ context-ში ხედავს (`todayEntry.note`). ⚠️ upsert **partial-ია** (მხოლოდ `note`) — PostgREST `ON CONFLICT DO UPDATE SET`-ში მხოლოდ გადაცემულ სვეტებს წერს, ე.ი. არსებული `symptoms`/`mood` არ იშლება. ნებისმიერ დღეზე მუშაობს (არა მხოლოდ დღეს).
- 🤖 **AI რჩევა**: `getDiaryAssistantSupport(entry)` — **Prime-ით არ იბლოკება** (ორსულობის კალენდრის pattern; fertility იმავე entitlement-ს იყენებს, Prime არ აქვთ აუცილებლად). რჩევა **მხოლოდ დღევანდელ დღეზე** ჩნდება (წარსული დღეები მხოლოდ ჩანაწერია).
- ⚠️ **ორი ხაფანგი, რაც გზადაგზა დავიჭირე**: (1) `buildDiarySignature` მხოლოდ symptoms/mood/note-ს ითვლის — მარტო LH-ის შეცვლაზე signature იგივე რჩებოდა და რჩევა **არ განახლდებოდა**; ამიტომ `loadFertilityAdvice(entry, signature, options)` ახლა signature-ს გარედან იღებს (`JSON.stringify({logs, note})`). (2) `getAssistantContext`-ს **45წმ cache** აქვს — `saveLog`-ს დაემატა `invalidateAssistantContextCache()`, თორემ ახლად ჩაწერილ ნიშანს ვერ დაინახავდა.
- 💡 **ავტო-trigger მხოლოდ ნიშნებზეა** (`loggedSignalsKey = JSON.stringify(dayLogs)`, 1.5წმ debounce) — **კლავიატურა შეგნებულად არ ურევია**, თორემ წერისას ყოველ პაუზაზე AI-ს გამოიძახებდა და დღიურ ლიმიტს დაწვავდა. კომენტარი ასისტენტამდე შენახვის ღილაკით მიდის (`force: true`).

**ასისტენტის header დროებით დამალულია (2026-07-11 — committed `8d023bb`, pushed, OTA production ✅ update group `fd4fa879`)**: `constants/tempFlags.js` → **`TEMP_HIDE_ASSISTANT_HEADER = true`**. `symptoms.js`-ის ზედა ბლოკი ("PERSONAL HEALTH ASSISTANT" / "ასისტენტი" / subtitle + მარჯვნივ portrait სურათი) აღარ ჩანს — მის ნაცვლად მხოლოდ safe-area spacer (`insets.top + 12`), თორემ ჩატი status bar-ის ქვეშ ჩავარდებოდა (header-ს `paddingTop: 62` ჰქონდა). **დასაბრუნებლად: flag → `false`** (JSX ხელუხლებელია ternary-ის მეორე ტოტში, არაფერი წაშლილა).

**Tab bar redesign + fertility calendar bug/guide (2026-07-11 — committed `cead2a1`, tab bar reverted in `7e907b2`, OTA production ✅ update group `8daf58fb`)**:
- 🐛 **ბაგი გასწორდა**: `FertilityCalendarScreen`-ს **`useFocusEffect(loadData)` არ ჰქონდა** — `useCycles` თავისით არ იტვირთება, ამიტომ კალენდარი ცარიელი მარკერებით ირენდერებოდა, სანამ pull-to-refresh `loadData()`-ს არ გამოიძახებდა. (`RegularCalendarScreen`-ს ეს ჰქონდა, ჩემს ახალ ეკრანს — არა.)
- **ინსტრუქცია**: კალენდრის header-ში `?` ღილაკი → bottom-sheet მოდალი `FERTILITY_GUIDE`-ით (7 პუნქტი: ფერები, დღის არჩევა, LH, BBT, ლორწო, ურთიერთობა, სად ნახოს შედეგები) + disclaimer.
- ⛔ **Tab bar-ის Instagram-რედიზაინი გაკეთდა და მაშინვე დაბრუნდა უკან** (user-ს არ მოეწონა). `app/(tabs)/_layout.tsx` **ბაიტ-ბაიტ თავდაპირველია**: ლეიბლებით, ამობურცული ასისტენტის ბუშტით (`marginTop: -32`, ჩრდილი), `colors.primary` ფერებით. **ნუ გადააკეთებ თავიდან** — მოცურებული ცენტრალური ასისტენტი და ლეიბლები შეგნებული არჩევანია.

**fertility მწვანე პალიტრა მთელ აპზე (2026-07-11 — committed `b555f61`, pushed, OTA production ✅ update group `5cfc7119`)**: user-ის მოთხოვნით — კალენდარი/სტატისტიკა უკვე მწვანე იყო, დანარჩენიც გაუტოლდა.
- **`symptoms.js` (ასისტენტი)**: `useFertility()` + მთელი `theme` ობიექტი fertility-ვარიანტით (ternary მთლიან theme-ზე, არა ველ-ველად — უფრო წაკითხვადია), ფონის გრადიენტი, notice glow-ები, eyebrow-ები, header eyebrow, "შენი ნაყოფიერება 🌿" + leaf icon (`FERTILITY OVERVIEW`).
- **`index.js` (home)**: `theme`-ის ყველა ველი fertility-სენსიტიური (bg/card/text/subText/primary/circleBg/softCard/border/peach/lavender/fertile/glassIcon/pageGradient/heroGradient/cardGradient) + hardcoded ვარდისფრები დაპატჩდა: tracker ring-ის `#FF7EA8`, `trackerCta` ღილაკი, heart/analytics icon-ები, hero glow-ები, eyebrow-ები.
- პალიტრა ერთიანია სამივე ეკრანზე: primary `#0E9F6E`, accent `#35C99B`, ლურჯი `#60A5FA`, light bg `#F4FFFB`/`#EBF9F2`/`#EEF6FF`, dark bg `#12241D`/`#141E20`/`#14161D`.

**🔓 fertility რეჟიმი გაიხსნა user-ებისთვის (2026-07-11 — committed `edf07a9`, pushed, OTA production ✅ update group `6d2b6b54`)**: `TEMP_FERTILITY_COMING_SOON` → **`false`** (flag დარჩა როგორც **kill switch** — `true`-ზე დაბრუნება მყისვე ბლოკავს შესვლასაც და გადახდასაც; იხ. `constants/tempFlags.js`).
- ⚠️ **გახსნამდე გასწორდა ხაფანგი** `profile.js`-ის `handleFertilityEnable`-ში: `!configured` (RevenueCat offerings ვერ ჩაიტვირთა) branch **goal-ს აყენებდა და "ჩაირთო ✨" წარმატებას აჩვენებდა, მაგრამ `has_pregnancy_subscription`-ს არ წერდა** → `fertilityUnlocked` false რჩებოდა და user-ს ფუნქციები დაბლოკილი (🔒) დახვდებოდა წარმატების შეტყობინების მიუხედავად. ახლა პატიოსნად ვარდება: "დროებით მიუწვდომელია". (ორსულობის რეჟიმის ანალოგიური `!configured` branch **არ შემიხებია** — იქ `enablePregnancyMode` რეალურად რთავს entitlement-საც; ეს არსებული live ქცევაა.)
- **დარჩენილი გარე დამოკიდებულებები (კოდით ვერ დავადასტურე — შენ გადაამოწმე)**: (1) RevenueCat/App Store Connect-ში **`pregnancy` პროდუქტი/offering რეალურად არსებობს?** თუ არა — iOS-ზე ყიდვა "დროებით მიუწვდომელია"-ზე ჩავარდება; (2) Android web checkout სერვერი **წერს თუ არა `profiles.has_pregnancy_subscription=true`** გადახდის შემდეგ (`.env`-ში URL set-ია, სერვერის მხარე ამ repo-ს გარეთაა).

**"მინდა დაორსულება" — ფაზა 6: Export + "ორსულად ვარ" (2026-07-11 — committed `67c8f8f`, pushed, OTA production ✅ update group `3f4aa7f2`) — 🏁 ფაზები 1-6 დასრულებულია**: client-only.
- **`utils/fertilityExport.js`** (ახალი): `buildFertilityReport(...)` — ექიმისთვის წასაკითხი txt (შეჯამება, დადასტურებული ოვულაციები მეთოდებით, ციკლების ისტორია, LH/BBT/ლორწო/ნიშნები/ვიტამინები — ყველა ქართული label-ით, + disclaimer). `calculateDueDate` (Naegele: LMP+280). `buildPregnancyTransition` → `{lmp,dueDate,currentWeek(≤40),daysRemaining,isPlausible}`; **`isPlausible` = 14..280 დღე** — უცნაურ LMP-ზე UI ცალკე აფრთხილებს, მაგრამ არ ბლოკავს.
- **`statistics.js`**: "ორსულად ვარ 🤰" ბარათი — Alert აჩვენებს LMP/EDD/კვირას დადასტურებამდე → `enablePregnancyMode(lastStart)` + `reloadFertility()`. FertilityContext-ის `fertilityMode` ავტომატურად false-დება (`!isPregnant`), აპი ორსულობის ეკრანებზე გადადის, notification-ებიც (enablePregnancyMode → schedulePregnancyNotifications ჯერ cancelAll-ს აკეთებს). + "ექიმისთვის ანგარიშის გაზიარება" ღილაკი (`FileSystem` + `Sharing`, იგივე pattern რაც profile.js-ის ძველ export-ს).
- ✅ **ტესტირებული**: 26/26 pass — EDD რეალურ Naegele-ზე (2026-01-01+280=2026-10-08), isPlausible boundary-ები (3 დღე/400 დღე/ზუსტად 14), კვირის 40-ზე cap, ყველა label თარგმანი, ცარიელი ანგარიში არ ვარდება.

**"მინდა დაორსულება" — ფაზა 5: Prediction Accuracy / symptothermal (2026-07-11 — committed `ed42a8a`, pushed, OTA production ✅ update group `0c45321f`)**: client-only, DB ცვლილება არ სჭირდებოდა.
- ⚠️ **მთავარი დიზაინის გადაწყვეტილება**: **`cycleEngine`-ის გლობალური პროგნოზი ხელუხლებელია** (მასზე მთელი აპი დგას — ყველა user, კალენდარი, home). Phase 5 მხოლოდ **fertility-scoped refined estimate**-ს აწარმოებს. ნუ გადაიტან გლობალურად რეგრესიის რისკის გარეშე.
- **`utils/ovulationDetection.js`** (ახალი): `detectBbtShift` (**კლასიკური "3 over 6"** — coverline = წინა 6-ის max + 0.1°C, 3 ზედიზედ მაღალი; ოვულაცია = პირველი მაღლის **წინა დღე**; gap guard — 3 მაღალი ≤5 დღეში, თორემ ხარვეზებზე გადაბმული "shift" ცრუდ დაიჭერს), `detectLhSurgeOvulation` (პირველი დადებითი +1 დღე), `detectMucusPeakOvulation` (ბოლო eggwhite/watery +1), `confirmOvulation` (**BBT იმარჯვებს თარიღზე**; BBT+თანხმობა → high, BBT მარტო → medium, LH მარტო → low; `agreement` ≤2 დღე), `getConfirmedOvulationsByCycle`, `getPersonalizedLutealLength` (8–18 დღის ფილტრით — **სტანდარტული 14-ის ნაცვლად user-ის რეალური**), `refineOvulationEstimate` (პრიორიტეტი: მიმდინარე ციკლის სიგნალები → personalized luteal (sample≥2) → calendar), `buildPredictionQuality` (level + რჩევები როგორ გააუმჯობესოს).
- **`statistics.js`**: "ოვულაციის დადასტურება" ბარათი (თარიღი/მეთოდები/სანდოობა + ⚠️ თუ ნიშნები არ ემთხვევა) + "პროგნოზის ხარისხი" (pill + suggestions) + ლუტეალური ფაზა. Test timing ახლა **refined ovulation-ს** იყენებს.
- **AI**: `buildFertilityAiContext`-ს დაემატა `confirmed_ovulation` / `best_ovulation_estimate` / `personal_luteal_phase_days` (ISO სტრიქონებად). Prompt-ს ავუხსენი: source `signals` > `personalized` > `calendar`; დადასტურება **რეტროსპექტული მტკიცებულებაა, არა სამედიცინო ფაქტი** ("ნიშნების მიხედვით სავარაუდოდ", არა "დადასტურდა").
- ✅ **ტესტირებული**: 48/48 pass — ბრტყელი ტემპერატურა და სუსტი აწევა **ვერ** ატყუებს, gap guard, ნაგავი temp-ები, LH/mucus, confidence-ის ყველა კომბინაცია (თანხმობა/უთანხმოება), luteal outlier ფილტრი, refine-ის პრიორიტეტი, quality დონეები. + Phase 4 regression (ძველი context ხელუხლებელი, ახალი ველები null როცა არაა). ტესტები წაშლილი.

**"მინდა დაორსულება" — ფაზა 4: AI + Recommendations (2026-07-11 — committed `a60f242`, pushed, OTA production ✅ update group `e3acf192`)**: DB ცვლილება არ სჭირდებოდა (client-only).
- **`utils/fertilityInsights.js`** (ახალი): `buildFertilityRecommendations` (კონტექსტური tips — LH შედეგი > ლორწო > ფაზა fallback; BBT nudge თუ არ აქვს), `evaluateDoctorVisitSignals` (**ასაკზე დამოკიდებული ზღვარი**: 12 თვე, ან 6 თუ 35+ — `birth_date`-იდან; + არარეგულარული ციკლი, ციკლი <21/>35 დღე, 8+ ტესტი 0 დადებითით), `getAgeFromBirthDate`/`getTryingThresholdMonths`, `buildFertilityAiContext`, სტატიკური `PARTNER_TIPS`/`LIFESTYLE_TIPS`/`MUCUS_HINTS`.
- **`services/assistantOrchestrator.js`**: `getAssistantContext`-ს fertility რეჟიმში დაემატა `fertilityTracking` ბლოკი (დღევანდელი LH/BBT/ლორწო/ურთ./ვიტამინები + regularity/trying/totals) — try/catch-ში, რომ ვერასდროს გატეხოს base context. profile select-ს დაემატა `birth_date`. **System prompt**-ს ორი ახალი სექცია: `# FERTILITY TRACKING DATA` (როგორ გამოიყენოს სიგნალები; low confidence → hedge, დიაპაზონი და არა ზუსტი თარიღი) და `# FERTILITY SAFETY` (არ დაპირდეს ჩასახვას, ტესტი ~11 DPO-დან, არ დაასვას უნაყოფობის დიაგნოზი). ⚠️ context JSON-ად მიდის Edge Function-ში (`buildInput`), ამიტომ ველის დამატება საკმარისია.
- **`calendar.js`**: 💡 "დღევანდელი რეკომენდაციები" ბარათი (მხოლოდ დღევანდელ დღეზე). **`statistics.js`**: "ღირს ექიმთან ახსენო" (ნარინჯისფერი, მკაფიო "ეს არ არის დიაგნოზი"), "პარტნიორის მხარდაჭერა", "ცხოვრების წესი".
- ✅ **ტესტირებული**: 36/36 pass (ასაკის ზღვრები 11/12 თვე @30 და 6 @36, LH პრიორიტეტი, ფანჯრის გარეთ დუმილი, null/no-args safety) + ცალკე integration ტესტი — რეალური `analyzeCycleRegularity` → `prediction_confidence:"high"` (prompt contract დაცულია). ტესტები მერე წაშლილი.

**"მინდა დაორსულება" — ფაზა 3: Daily Plan + Reminders + Vitamins (2026-07-11 — committed `743d3c8`, pushed, migration + OTA production ✅ update group `078f86a3`)**:
- **migration `20260714_add_supplement_log_type.sql`** (**remote ✅**): `fertility_logs` check constraint-ს დაემატა `'supplement'` (value `{taken:text[]}`). ⚠️ constraint drop+add-ია, იდემპოტენტური.
- **`utils/fertilityPlan.js`** (ახალი): `SUPPLEMENT_OPTIONS` (6), `getLhTestWindow` (ოვ.−5..+1), `buildDailyPlan({forecast,todayLogs})` — პირობითი ელემენტები: LH ტესტი მხოლოდ ტესტ-ფანჯარაში, ურთიერთობა მხოლოდ ნაყოფიერ ფანჯარაში (peak = ოვ. დღე/წინა), BBT/ვიტამინები/ნიშნები ყოველდღე; `done` flag დღევანდელი log-ებიდან, priority-ით დალაგება, doneCount/totalCount.
- **`services/notifications.js`**: `scheduleFertilityReminders(lastPeriod, cycleLength)` — ⚠️ **iOS-ს 64 pending local notification-ის ლიმიტი აქვს**, ამიტომ 3 ყოველდღიური nudge (BBT 07:00, ვიტამინები 10:00, წყალი 15:00) **repeating trigger**-ითაა (`SchedulableTriggerInputTypes.DAILY`, 1 slot თითო, არა 30), დათარიღებულები კი მხოლოდ **2 ციკლზე** (LH ოვ.−4..−2, peak ოვ.−1..0, fertile start, period) → ჯამში ~17 slot. `syncCycleRemindersForUser`-ში fertility branch (pregnancy-ის შემდეგ): `goal==="დაორსულება" && (isAdminEmail || resolvePregnancyAccessFromProfile)`. notifications.js ახლა იმპორტავს purchases.js-ს — circular import არაა (გადამოწმებული).
- **`calendar.js`**: მე-6 ლოგერი 💊 ვიტამინები (multi-select). **`index.js`**: "დღევანდელი გეგმა" ბარათი (fertilityMode) — peak/fertile სათაური, done/total pill, ticked items, ტაპი → calendar.
- ✅ **ლოგიკა ტესტირებულია**: 27/27 pass (peak/ramp/early ციკლის დღეები, done flags, ცარიელი supplement, null forecast, no-args safety), მერე წაშლილი.

**"მინდა დაორსულება" — ფაზა 2: Dashboard + Stats (2026-07-11 — committed `7f5fc4c`, pushed, OTA production ✅ update group `9557a490`)**: fertility ანალიტიკა `statistics.js` tab-ში (ცალკე ეკრანი არა — pregnancy-ის pattern: `pregnancyMode → fertilityMode → regular`).
- **`utils/fertilityStats.js`** (ახალი, სუფთა ლოგიკა — ეკრანი მხოლოდ ხატავს): `getObservedCycleGaps` (რეალური gap-ები consecutive start_date-ებს შორის, არა stored cycle_length; ფილტრავს <15/>60 დღე ნაგავს), `analyzeCycleRegularity` (avg/shortest/longest/spread, `isRegular = spread<=7`, accuracy high/medium/low sample+spread-ით), `buildFertileWindows` + `isDateInFertileWindows`, `computePregnancyTestTiming` (earliest = ოვ.+11დღე, reliable = ოვ.+14), `summarizeFertilityLogs` (ურთ. ნაყოფიერში, LH+, BBT, top სიმპტ.), `computeTryingHistory` (თვეები/ციკლები/ნაყოფიერი დღეები).
- **`statistics.js`**: `FertilityStatisticsScreen` (მწვანე პალიტრა) — hero: სანდო ტესტამდე დარჩენილი დღეები; მცდელობის ისტორია (4 cell); ციკლის რეგულარულობა (+არარეგულარულზე გაფრთხილება, medical safety); ნაყოფიერების მაჩვენებლები; BBT bar-trend (35–38°C zoom, `AnimatedBar` reuse); ოვ. ნიშნების სიხშირე; disclaimer.
- ✅ **ლოგიკა ტესტირებულია**: 31/31 pass ad-hoc ESM ჰარნესით (რეგულარული/არარეგულარული/ცარიელი/ნაგავი-gap/არასწორი BBT სცენარები), მერე წაშლილი.

**"მინდა დაორსულება" — ფაზა 1: Core Tracking (2026-07-11 — committed `053f6cf`, pushed, migration + OTA production ✅ update group `8aae7cbf`)**: fertility რეჟიმის საფუძველი. დიზაინი: ორსულობის რეჟიმის ანალოგიურად, `fertilityMode` flag ცვლის tab-ებს (ჯერ calendar + home strip; სხვა tab-ები ფაზა 2+).
- **DB migration `20260713_create_fertility_logs.sql`** (**remote "women"-ზე ✅**): ერთი მოქნილი ცხრილი `fertility_logs (user_id, date, type, value jsonb)`, `type` ∈ intercourse/lh_test/bbt/cervical_mucus/ovulation_symptom, `unique(user_id,date,type)`, RLS own-only. ⚠️ ვერსია `20260713` (არა `20260712`) — 20260712 უკვე დაკავებულია purchase_context-ით (duplicate version-ის თავიდან ასაცილებლად).
- **`context/FertilityContext.js`** (ახალი): `fertilityMode = goal==="დაორსულება" && (admin||has_pregnancy_subscription) && !pregnancy_mode`. `_layout.tsx`-ში `FertilityProvider` (PregnancyProvider-ის შიგნით).
- **`services/fertilityLogs.js`** (ახალი): `upsertFertilityLog(date,type,value)` (value=null → delete), `getFertilityLogsForDay`, `getFertilityLogsRange`. fail-silent.
- **`utils/cycleEngine.js`**: `getPregnancyChanceKey` 3→**4 დონე** (დაემატა "medium": ოვ. ±7/±3 კიდეები).
- **`app/(tabs)/calendar.js`**: ახალი `FertilityCalendarScreen` (მწვანე პალიტრა) — ციკლის კალენდარი (`useCycles` marks) + შერჩეულ დღეზე 5 ლოგერი: ❤️ ურთიერთობა (დაცული/დაუცველი + "დაემთხვა ნაყოფიერ დღეს" ბეჯი), 🧪 LH ტესტი (4), 🌡️ BBT (34–43°C input), 💧 ლორწო (5), 🌸 ოვ. ნიშნები (multi). ლოგიან დღეებზე dot. default export ახლა: pregnancy → fertility → regular.
- **`app/(tabs)/index.js`**: `fertilityMode`-ზე hero-ს ქვემოთ strip — "ოვულაციამდე N დღე" + შანსი, ტაპი → calendar tab.
- ⚠️ `TEMP_FERTILITY_COMING_SOON` კვლავ `true` — რეალურ user-ს fertilityMode არ ერთვება (paywall ბლოკია); ხილვადია მხოლოდ admin-ს, ვინც `goal="დაორსულება"` ხელით დააყენებს DB-ში. ფაზები 2-6 დარჩა (dashboard/stats, plan/reminders/vitamins, AI injection, prediction accuracy, export + "ორსულად ვარ").

**Shared subscription + analytics context (2026-07-11 — committed `1890627`, pushed, migration + OTA production ✅ update group `7e070d26`)**: "მინდა დაორსულება" (fertility) და ორსულობის რეჟიმი ერთ `pregnancy` RevenueCat entitlement-ს იზიარებენ — **ერთი $2.99 გამოწერა ორივეს ხსნის** (დიზაინი A, user-ის გადაწყვეტილება). ცალკე პროდუქტი RevenueCat-ში **არ** დაემატა. დაემატა მხოლოდ analytics მარკერი:
- **migration `20260712_add_pregnancy_purchase_context.sql`** (**remote-ზე ✅**): `profiles.pregnancy_purchase_context text` — `"fertility"` | `"pregnancy"`, access-ზე გავლენა არ აქვს.
- **`services/purchases.js`**: `writePregnancyStatusToProfile`-ს context key **მხოლოდ მაშინ** უწერს, როცა გადმოეცემა (refresh calls null-ით არ გადააწერენ — კრიტიკული). `purchasePregnancyPackage(pkg, {context})` მე-2 არგუმენტი. `openAndroidPregnancyCheckout({context})` → URL-ს `&context=` ამატებს. ახალი `recordPregnancyPurchaseContext(context)` (fail-silent, Android return flow-სთვის).
- **`profile.js`**: iOS fertility ყიდვა → `{context:"fertility"}`, iOS pregnancy → `{context:"pregnancy"}`, `startAndroidPregnancyCheckout` context-ს payload.type-იდან იღებს, Android return handler-ში `recordPregnancyPurchaseContext(pendingCheckout.type)`. Fertility მოდალის ტექსტი: "ერთი გამოწერა ორივე რეჟიმს ხსნის".
- ანალიტიკა: `select pregnancy_purchase_context, count(*) from profiles where has_pregnancy_subscription group by 1;`
- ⚠️ ძველ გამომწერებს context=null (უცნობია). ⚠️ `TEMP_FERTILITY_COMING_SOON` კვლავ `true` — გადახდა ისევ დაბლოკილია, ეს მხოლოდ plumbing.

**TEMP: fertility mode სრულად დაბლოკილია — "მალე დაემატება" (2026-07-11 — committed `f81ea4d`, pushed, OTA production ✅ update group `7cebbf79`)**: user-ის მოთხოვნით, სანამ რეჟიმი მზადდება, **ვერავინ** (admin-ის ჩათვლით) ვერ შედის და ვერ იხდის. `constants/tempFlags.js` → `TEMP_FERTILITY_COMING_SOON = true` (წინა `TEMP_UNLOCK_FERTILITY_FOR_ALL` წაშლილია — free-unlock აღარ მოქმედებს). ყველა შესვლის წერტილი აჩვენებს Alert-ს "მალე დაემატება 🌿": home ბანერი (`index.js` onPress), პროფილის fertility row-ები (`openFertilityFlow` helper), deep-link param, და `handleFertilityEnable`-ის თავში hard guard (belt-and-suspenders). `fertilityUnlocked` სამივე gate-ში დაბრუნდა `isAdmin/has_pregnancy_subscription`-ზე — free-window-ში შესულებს AI/რჩევები ისევ დაეკეტათ. **Revert**: flag → `false` აბრუნებს paywall flow-ს (გადახდა ისევ ამუშავდება!).

**ბანერების ვიზუალის განახლება (2026-07-11 — committed `ffadbc7`, pushed, OTA production ✅ update group `6ef61a3c`)**: (ა) ორივე ბანერში სურათის კონტეინერი გაიზარდა — `pregnancyBannerImageWrap` 78×98 → **104×124**, `pregnancyBannerGradient` minHeight 132 → **152**. (ბ) "მინდა დაორსულება" ბანერს საკუთარი პალიტრა მიეცა (ორსულობის pink/peach-ისგან განსხვავებით **მწვანე-ლურჯი**): `theme.fertilityBannerGradient` (light: mint→aqua→ცისფერი; dark: მუქი მწვანე-ლურჯი), styles `fertilityBannerShell` (მწვანე border/shadow), `fertilityBannerGlowGreen/Blue`, `fertilityBannerEyebrowText` (#0E9F6E), `fertilityBannerDot` (#60A5FA), leaf icon #0E9F6E. Layout styles კვლავ გადაზიარებულია pregnancyBanner*-დან.

**Home "მინდა დაორსულება" ბანერი (2026-07-11 — committed `ece1d82`, pushed, OTA production ✅ update group `34925f5d`)**: `index.js`-ში ორსულობის ბანერის ქვემოთ დაემატა იდენტური სტრუქტურის მეორე ბანერი (`styles.pregnancyBanner*` styles გადაზიარებულია, ახალი style არ შექმნილა): 🌿 leaf icon, eyebrow FERTILITY·PREMIUM, title "მინდა დაორსულება", სურათი `assets/images/minda-daorsuleba.png` (`FERTILITY_BANNER_IMAGE`). ჩვენების პირობა იგივეა რაც ორსულობის ბანერს (`!isPremium && !pregnancyMode`). ტაპი → `router.push({ pathname: "/(tabs)/profile", params: { openFertility: Date.now() } })`; `profile.js`-ში `useLocalSearchParams().openFertility`-ის useEffect ხსნის fertility paywall მოდალს (timestamp param-ის გამო ყოველი ტაპი თავიდან ხსნის).

**TEMP: fertility mode unlocked for every account (2026-07-11 — committed `d6a3517`, pushed, OTA production ✅ update group `a3a4155f`)**: user-ის მოთხოვნით დროებით — `constants/tempFlags.js` → `TEMP_UNLOCK_FERTILITY_FOR_ALL = true` (**ერთი ცვლადი, revert-ისთვის `false` დააყენე ან წაშალე flag+usages**). თავდაპირველად Prime-ზე გავაბი (`isPremium`), user-მა გაასწორა — free account-იც (test account) თავისუფლად უნდა შედიოდეს, არა მხოლოდ Prime, ამიტომ flag თავისთავად (ანგარიშის tier-ის მიუხედავად) ხსნის. `fertilityUnlocked` 3-ივე გადამოწმების წერტილში (`profile.js`, `assistantOrchestrator.js`, `index.js`) გავრცელდა. `profile.js`-ის `handleFertilityEnable`-ს დაემატა TEMP branch (isAdmin branch-ის შემდეგ, real purchase-მდე) — ერთი კლიკით უფასოდ ააქტიურებს ნებისმიერი ანგარიშისთვის. **ვიზუალი (modal-ის $2.99 copy, ფასის ლეიბლი) შეგნებულად არ შეხებია** — user თვითონ დაამუშავებს მოგვიანებით.

**Fertility mode display rename (2026-07-11 — committed `22508df`, pushed, OTA production ✅ update group `ac54c524`)**: profile.js-ში fertility goal-ის **ჩვენება** გადაერქვა "დაორსულების რეჟიმი"-დან → **"მინდა დაორსულება"**-ზე (goal picker option, "ჩემი მიზანი" row value, "ორსულობა" სექციის row title, paywall მოდალის სათაური, ყველა Alert). **DB value უცვლელია** — კვლავ `"დაორსულება"` ინახება/შედარდება ყველგან (`goal === "დაორსულება"`), მხოლოდ ახალი `getGoalLabel()`/`FERTILITY_MODE_LABEL` helper-ი ცვლის ჩვენებას. `assistantOrchestrator.js`/`dailyAdvice.js`-ის შიდა GOAL_MAP-ებზე არ მოქმედებს (ისინი raw value-ს იყენებენ). Description წინადადებები (მოდალის body ტექსტი, ბენეფიტების სია) შეგნებულად არ შეცვლილა — ბუნებრივი წინადადებებია, არა ბრენდის სახელი.

**Fertility paywall fix (2026-07-11 — committed `acd1b5a`, pushed, OTA production ✅ update group `1c35bf7d`)**:
- პრობლემა: 🎯 "ჩემი მიზანი" picker-იდან "დაორსულება" **უფასოდ** ირთვებოდა — 🌿 fertility paywall-ს ($2.99/თვე, pregnancy entitlement) მთლიანად უვლიდა გვერდს.
- გადაწყვეტა (user-ის არჩევანი): goal არჩევა დარჩა უფასო, მაგრამ **ფუნქციები დაიკეტა** გადახდამდე. `fertilityUnlocked = isAdmin || has_pregnancy_subscription`:
  - `profile.js` — "ორსულობა" სექციის fertility row ახლა 3-მდგომარეობიანია: 🌿 აქტიური (paid) / 🔒 "მიზნად არჩეულია — გახსენი $2.99-ად" (goal არჩეული, unpaid → ტაპი paywall მოდალზე) / ჩვეულებრივი intro row. `handleFertilityEnable`-ის წარმატებულ paths-ზე დაემატა `await reloadPregnancy()` (თორემ iOS ყიდვის მერე local `hasSubscription` ძველი რჩებოდა და row 🔒-ზე იჭედებოდა).
  - `services/assistantOrchestrator.js` — `getAssistantContext` (ყველა AI ფუნქციის საერთო წყარო) ახლა `effectiveGoal`-ს იყენებს: unpaid "დაორსულება" → AI-სთვის "ციკლის კონტროლი" (Get Pregnant პერსონა არ ირთვება). profile select-ს დაემატა `has_pregnancy_subscription`.
  - `index.js` (home) — static fallback `getDailyAdvice` + advice cache key იგივე `effectiveGoal` ლოგიკით.
- Scope-გარეთ დარჩა (user-მა გადადო): AI ჩატის ლიმიტი fertility-paid user-ს კვლავ Free 1/დღეა (edge function მხოლოდ `pregnancy_mode`-ს ცნობს, goal-ს არა).

**წინა commit** — `1614bc3` "Add server-side AI usage limits + admin/profile polish" (2026-07-11): სერვერ-საიდ AI usage limiting + admin/profile/purchases polish. **Committed + pushed + deployed ✅** (migration + edge function + OTA `82514617`).

**AI usage limiting (2026-07-11 — done, ცოცხალია)**:
- `supabase/functions/ai-assistant/index.ts` — ახლა auth-ს ამოწმებს (anon key მარტო აღარ გადის) და დღიურ ლიმიტებს აწესებს: chat — Free **1**, Pregnancy **10**, Prime **20**; feature-calls (home-daily-advice, calendar-diary-support, pregnancy-weekly-advice და pregnancy ვარიანტები) — **30/day** თითო; admin unlimited. უცნობი feature name → strictest (chat) ლიმიტზე ვარდება (forged feature ვერ გახსნის დიდ budget-ს). **Deployed remote-ზე ✅**.
- `supabase/migrations/20260706_create_assistant_ai_usage.sql` — `assistant_ai_usage` table + `consume_assistant_ai_usage`/`refund_assistant_ai_usage` RPC (SECURITY DEFINER, service_role only, Asia/Tbilisi timezone). **Remote-ზე აპლიცირებულია ✅** (migration list: 20260706 Local+Remote).
- client-side: `app/(tabs)/*`, `admin.jsx`, `PregnancyContext.js`, `purchases.js` ცვლილებები. ⚠️ ესენი users-თან **EAS OTA/build-ით** მიდის, არა supabase deploy-ით — გადაამოწმე OTA გავიდა თუ არა.

⚠️ **Supabase account/project — ყურადღება!** ამ repo-ს backend = project **"women"** `ewogfcyhkoevnxmstdin` (`.env` EXPO_PUBLIC_SUPABASE_URL). ეს **სხვა Supabase account-ია**, ვიდრე football repo-ს (football = `exbakxkfbglnsdescimj`). CLI-ს default login ხშირად football-ზეა — cycle-app-ის deploy-მდე გადაამოწმე `npx supabase projects list`-ით რომ "women" ჩანს და LINKED-ია (●), თორემ migration არასწორ ბაზაში წავა.

**`supabase/.temp/*`** — CLI cache, git-ში ტრექინგშია (უჩვეულო); feature commit-ებში ნუ ჩააგდებ, ხელით ნუ წაშლი user-თან შეთანხმების გარეშე.

---

## Commands

```powershell
cd cycle-app   # ან იმ დირექტორიის root, სადაც ეს package.json-ია

npx expo start              # dev server
npx expo start --dev-client # dev client-ით
npm run android
npm run ios
npm run web
npm run lint                # expo lint
```

ტესტი (`test` script) არ არსებობს ამ პროექტში.

### Supabase
```powershell
npx supabase functions deploy ai-assistant
npx supabase functions deploy send-push-notification
npx supabase db push --linked
```

---

## არქიტექტურა

### Routing (`app/`, expo-router)
- `_layout.tsx` — root Stack (`headerShown: false`, fade animation). Provider ჯაჭვი: `ThemeProvider` → `PregnancyProvider` → `OnboardingProvider` → `LayoutContent`. Mount-ზე: Meta app events init, profile email sync, push token register, `supabase.auth.onAuthStateChange` subscribe.
- `index.jsx` — `<Redirect href="/splash" />`.
- `splash.jsx` — სესიის მდგომარეობის მიხედვით რაუთინგი (auth/onboarding/tabs).
- `auth/` — `login.jsx`, `register.jsx`.
- `onboarding/` — 7 ნაბიჯიანი wizard (name, birth, protection, health, cycle-length, last-period, notifications) + `_layout.js`.
- `(tabs)/` — მთავარი tab navigator: `_layout.tsx` (tab bar + floating "AdminAssistant" chat widget), `index.js` (home), `calendar.js`, `statistics.js`, `symptoms.js` (AI chat assistant UI-იც აქ ცხოვრობს), `profile.js`.
- `admin.jsx` — admin dashboard (root-level, `(tabs)`-ის გარეთ).
- `premium.jsx` / `pregnancy-premium.jsx` — paywall-ები.

### State / Context (`context/`)
- **`ThemeContext.js`** — თემა + `isPremium`/`isAdmin`/`usePremiumTheme` (persist AsyncStorage-ში, key `cycle_app_use_premium_theme`). `checkPremiumStatus()` თანმიმდევრობა: admin email (`services/adminAccess.js`) → `profiles.premium_override` → `resolvePremiumAccessFromProfile` + RevenueCat sync (`services/purchases.js`). AppState foreground-ზეც re-check-ავს.
- **`PregnancyContext.js`** — `profiles.pregnancy_mode`/`pregnancy_start_date`/`has_pregnancy_subscription`/`pregnancy_until`-დან ითვლის `currentWeek`(≤40)/`currentTrimester`(week 12/27 threshold)/`daysRemaining`(280-day gestation). `enablePregnancyMode`/`updatePregnancyStartDate`/`disablePregnancyMode`.
- **`components/OnboardingContext.js`** — onboarding wizard ფორმის state (მიუხედავად სახელისა, `context/`-ში კი არა, `components/`-შია).

⚠️ **`წაიკითხე.txt`** root-ში დეველოპერის ძველი scratch note-ია, შეიცავს `ThemeContext.js`-ის მოძველებულ ასლს — **არ ეყრდნობი**, მიმდინარე `context/ThemeContext.js`-ს გადაამოწმებ ყოველთვის.

### Services (`services/`)
| ფაილი | პასუხისმგებლობა |
|---|---|
| `supabase.js` | Supabase client, AsyncStorage session persistence |
| `ai.js` | თხელი client wrapper → Edge Function `ai-assistant` (`EXPO_PUBLIC_AI_FUNCTION_NAME`) |
| `assistantOrchestrator.js` | AI system prompt-ების აწყობა (cycle-assistant + pregnancy-assistant, ორივე ქართულად, დინამიური context injection), 45s in-memory cache |
| `purchases.js` | RevenueCat (`prime` + `pregnancy` entitlements). Android-ზე web-checkout fallback (`openAndroidPrimeCheckout`/`...Pregnancy...`) — `profiles` table წყაროდ ითვლება, არა RevenueCat SDK |
| `adminAccess.js` | Hardcoded `ADMIN_EMAILS` — ამჟამად `mishvelidze.u@gmail.com` |
| `adminQuery.js` | Local keyword-based ქართული NLP admin chatbot-ისთვის (LLM-ის გარეშე) |
| `notifications.js` | Push notification schedule (cycle reminders, pregnancy weekly) |
| `metaAppEvents.js` | Facebook SDK event log (purchases, app events) |
| `profileSync.js` | auth email → `profiles.email` sync |
| `cycleDataMigration.js` | ძველი cycle data ფორმატის მიგრაცია |

### Hooks / Utils
- `hooks/useCycles.js` — cycle CRUD (`cycles` table), fallback `profiles.last_period`-ზე, calendar `markedDates` აწყობა (`utils/cycleEngine.js`-ით).
- `utils/cycleEngine.js` — სუფთა ციკლის მათემატიკა: `calculateCycleState`, `getCycleWindowDates`, `getCyclePhaseKey` (period/follicular/fertile/luteal), `getPregnancyChanceKey`. Ovulation = შემდეგი პერიოდის −14 დღე; fertile window = ovulation −5..+1.

### Supabase DB
ძირითადი tables: `profiles` (name, phone_number, email, cycle/period length, last_period, premium/pregnancy ველები), `cycles`, `assistant_chat_history`, `billing_entitlements`, `assistant_ai_usage` (ახალი, დღიური AI ლიმიტი).

Migrations (`supabase/migrations/`, ქრონოლოგიურად): phone_number → premium reset → premium_override → assistant_chat_history → profile email/created_at + admin RLS → premium billing fields → `billing_entitlements` (გენერალიზებული entitlement ledger prime/pregnancy) → `assistant_ai_usage` + rate-limit RPC-ები.

Edge Functions (`supabase/functions/`):
- **`ai-assistant/index.ts`** — auth ვალიდაცია, premium/pregnancy სტატუსის სერვერ-საიდ resolve, დღიური ლიმიტები (chat: free 1/day, pregnancy 10/day, prime 20/day; feature-calls — daily-advice/diary-support/pregnancy-weekly — 30/day თითოეული; admin unlimited). OpenAI Responses API (`gpt-5.4-mini` default, `OPENAI_MODEL` secret). წარუმატებლობაზე `refund_assistant_ai_usage` RPC-ით აბრუნებს quota-ს.
- **`send-push-notification/index.ts`** — server-side push sender.
- **`_shared/cors.ts`** — CORS headers helper.

---

## Product rules (Free vs Prime vs Pregnancy)

- **ორი premium tier**: **Prime** (მუქი თემა + AI chat quota ზრდა) და **Pregnancy** (ცალკე subscription, ორსულობის რეჟიმი). RevenueCat iOS/Android-ზე, Android-ზე დამატებით web-checkout fallback (`EXPO_PUBLIC_ANDROID_PRIME_PAYMENT_URL`/`..._PREGNANCY_PAYMENT_URL`).
- Premium access resolution თანმიმდევრობა: `premium_override` (admin manual) > `is_premium && premium_until` ვადა > legacy fallback (თუ `premium_until` არ არის — აქტიურად ითვლება, pre-expiry-tracking row-ებისთვის).
- Admin (`mishvelidze.u@gmail.com`) ავტომატურად იღებს Prime-ს + unlimited AI + admin dashboard/chatbot წვდომას.
- AI chat დღიური ლიმიტები: Free 1, Pregnancy 10, Prime 20; feature-calls (daily-advice, diary-support, pregnancy-weekly) 30/day ყველასთვის, Tbilisi timezone-ის მიხედვით.
- ინტერფეისი/AI პრომფტები/admin ტექსტი — ყველგან ქართული (ka-GE).

---

## მნიშვნელოვანი დეტალები

- **AI Edge Function**: `supabase/functions/ai-assistant/index.ts` — `OPENAI_API_KEY` მხოლოდ Supabase secret-ში. მოდელი `gpt-5.4-mini` default.
- **Android premium**: RevenueCat/Play Billing-ის ნაცვლად web checkout-ზეა გადამისამართებული (`services/purchases.js`), `profiles` table-ია წყარო, არა RevenueCat SDK პასუხი.
- **admin email hardcoded** `services/adminAccess.js`-ში — ახალი admin-ის დამატება ამ ფაილში ხდება.
- **`supabase/.temp/`** git-ში ტრექინგშია (უჩვეულოა) — ნუ წაშლი, user-თან შეთანხმების გარეშე.
- **`constants/theme.ts`** — Expo scaffold-ის default ფერებია, სავარაუდოდ აღარ გამოიყენება (`ThemeContext.js`-მა ჩაანაცვლა).
- **Georgian encoding**: PowerShell-ი ზოგჯერ ქართულს mojibake-ად აჩვენებს, ფაილები UTF-8-ია — ტექსტი არ შეცვალო სანამ browser/app-ში გატეხილი არ ჩანს.

---

## შესრულებული სამუშაოების ისტორია (ბოლო commit-ები, დეტალებისთვის `git log`)

- `9bfd071` Fix input row layout — collapsible WhatsApp-style ორივე ასისტენტში
- `220b2f2` Auto-dismiss keyboard on send, expandable input
- `8484a52` Move admin assistant to floating button, hide symptoms tab for admin
- `13345e2` Fix avatar loading — fallback getPublicUrl
- `a474d3c` Full-screen avatar preview modal admin panel-ში
- `606594c` Revert version to 1.0.6 (OTA compatibility)
- `dfae5e5`/`2ba9a02` Metro parse error ფიქსები (curly quotes)
- `26e039b` Admin photo viewer, users-with-photos stat, admin assistant დამატება
- `d52f2b2` Android web premium checkout support
- `64e813e` Meta app events ინტეგრაცია

---
> Source: [mishvelidzeu-code/kalebi](https://github.com/mishvelidzeu-code/kalebi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
