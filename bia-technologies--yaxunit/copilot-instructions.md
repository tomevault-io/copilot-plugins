## yaxunit

> - Используй табуляция в качестве отступов

CODE STYLE RULES

- Используй табуляция в качестве отступов
- use copyright
- generate doc comment for export methods, exclude tests

Для проверки контекста исполнения сначала проверяй на контекст сервера.
Неправильно:
    #Если Клиент Тогда
        Возврат ЮТЗапросыСлужебныйВызовСервера.ЗначенияРеквизитов(Ссылка, ИменаРеквизитов, Ложь);
    #Иначе
        Возврат ЮТЗапросыСлужебныйСервер.ЗначенияРеквизитов(Ссылка, ИменаРеквизитов, Ложь);
    #КонецЕсли
Правильно:
    #Если Сервер Тогда
        Возврат ЮТЗапросыСлужебныйСервер.ЗначенияРеквизитов(Ссылка, ИменаРеквизитов, Ложь);
    #Иначе
        Возврат ЮТЗапросыСлужебныйВызовСервера.ЗначенияРеквизитов(Ссылка, ИменаРеквизитов, Ложь);
    #КонецЕсли

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bia-technologies)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bia-technologies)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
