## mnemo-c-subset

> Riferimento completo — subset C di Mnemo, cosa manca rispetto al C standard e limiti (da c_lower.py)


# Mnemo — subset C vs C “normale”

Mnemo compila un **sottoinsieme** di C verso **Kairos** (`mnemo/c_lower.py`, `mnemo/layout_collect.py`, `mnemo/compile.py`). Non è un compilatore C conforme allo standard. **Backlog e priorità**: vedi [`TODO.md`](../../TODO.md) nella root del repo. Questa nota elenca **limiti e differenze** rispetto al C d’uso comune.

## Tipi fondamentali

- **`char` / `signed char` / `unsigned char`** come tipo scalare di variabile: gli scalari ammessi sono sostanzialmente `int`, `unsigned` / `unsigned int`, `bool` / `_Bool` (`_SCALAR_NAMES`).
- **`short`, `long`, `long long`** (e varianti unsigned) come tipi distinti per dichiarazioni normali.
- **`float`, `double`, `long double`** e tipi complessi.
- **`wchar_t`, `size_t`, `ptrdiff_t`** come tipi “veri” del C standard (non c’è ecosistema ABI completo).

## Letterali e stringhe

- Letterali interi: `_const_int` accetta solo tipi AST `int`, `long`, `unsigned int`, `long long`.
- **Stringhe `"..."`** e uso “da manuale C” di array di caratteri / puntatori a stringa read-only.
- **Costanti carattere** `'x'` come primarie intere ovunque (percorso ristretto).

## Dichiarazioni e puntatori

- Puntatori dichiarati nel subset: **`int *`**, **`unsigned *` / `unsigned int *`**, **`void *`** (un livello, dove previsto), e **`T *`** con **`T`** typedef di **`struct`** / **`union`** (un livello; stesso modello “handle” dei parametri). Più locali puntatore (es. più `malloc`) sono supportati tramite pool `__mn_pool_*`.
- **`char *`** con stringa letterale `"..."` e **`const char *`** con la stessa convenzione sono supportati per `printf`/`putchar` nel subset; non è il modello puntatori/stringhe del C completo.
- **`const`, `volatile`, `restrict`, `_Atomic`**: spesso **accettati sintatticamente** (qualificatori su `TypeDecl`); **nessuna** semantica C completa (nessun enforcement const-correctness).
- **`static` / `extern`**: Mnemo usa **una TU** verso Kairos; linkage e durata non coincidono con il C standard su più unità di traduzione.

## `main`, `argc`, `argv`

- **`main`**: solo nessun parametro (`void`), solo **`int argc`**, oppure **`int argc`** poi **`char **argv`**; altre firme → errore.
- **`argv`** è **stub** (non array di stringhe POSIX, non dereferenziabile come in C reale).
- Nessun terzo argomento (`envp`), `wmain`, ecc.

## Array

- **VLA**: dimensione deve essere **costante intera** a compile-time.
- Prodotto dimensioni ≤ **`ARR_MAX` (1024)** elementi totali.
- Elementi: solo **scalari Mnemo** o **puntatore** nel senso sopra (`int*`, `void*`, ecc.).

## Struct e union

- **Passaggio struct per valore** (flatten sui campi) è supportato dove previsto dal layout; **union** per valore come singola parola.
- **Inizializzatori `{ ... }`** alla dichiarazione: **struct** (lista in ordine di campi, senza designatori); **union** con **un** solo valore in `{ … }`.
- **`struct.campo`**: `=` e assegnamenti composti (`+=`, `*=`, `^=`, `<<=`, … come per gli scalari).
- **`p->campo`**: idem (`=` e composti).
- **Union**: assegnamenti composti solo per un sottoinsieme di operatori; altri → errore.
- **Bit-field** nei struct: non nel modello (campi appiattiti a interi).

## Enum

- Enum come **costanti intere**; `switch`/`case` con costante intera o enumeratore — non sistema tipi enum completo del C.

## Operatori unari

- Implementati in forme limitate: `-`, `*`, `&` (solo **`&id`** e **`&struct.campo`** con vincoli), **`~`** (come `(-1) ^ x`), `sizeof`.
- **`++` / `--`** (prefisso/postfisso) su **`ID`**, **`*p`**, **`a[i]`**, **`s.campo`**, **`p->campo`** (istruzione ed espressione), con ordine di side-effect allineato agli assegnamenti esistenti.
- Altri unari → `operatore unario non supportato`.

## Operatori binari e logici

- Espressioni: insieme chiuso di operatori (`+ - * / %`, `& | ^`, `<< >>`, virgola, ecc.); il resto → `operatore binario non supportato`.
- **`&&` / `||` / `==` / …** non sono valutati come nel C su ogni sotto-espressione: il controllo usa predicati dedicati (`if`, confronti atomici, `!` sul predicato).

## Assegnamenti

- Su **`ID`** scalare: `=`, `+=`, `-=`, `*=`, `/=`, `%=`, `^=`, `&=`, `|=`, `<<=`, `>>=`.
- **Non supportati** su scalare: altri composti → `assegnamento con … non supportato`.
- **`array[i]`** e **`*p`**: **`=`** e assegnamenti composti (stesso insieme di operatori degli scalari).
- Molti lvalue → `lvalue non-ID non supportato`.

## Cast

- Solo se `_cast_accepts_pointer_or_scalar`: **`void`**, puntatori **`void*` / `int*` / `T*`** (`T` struct o union typedef), scalari del subset; altro → `cast non supportato`.

## Indirizzo `&`

- Solo **`&variabile`** e **`&struct.campo`** (con disponibilità slot); non `&a[i]`, `&(expr)` generico, ecc.

## Puntatori e memoria

- Semantica **indici pool** / celle **`__mn_mem*`**, non aritmetica puntatori e confronti del C pieno.
- **`malloc` / `free`**: modello **pool** dimensionato dalla toolchain, non libc.

## Chiamate

- Callee: nome diretto **`f(...)`**, oppure **puntatore a funzione** risolto a **compile-time**: variabile dichiarata `int (*p)(…)` con init **`g`** o **`&g`** (`g` dichiarata nello stesso file), più **`p(...)`** e **`(*p)(...)`**. Nessun puntatore a funzione calcolato a runtime generico.
- **Variadiche** (`...`): nelle **definizioni** utente → errore; **dichiarazioni** con `...` (es. `printf`) restano per I/O built-in. Nessun `va_list`.
- **`void`**: non come sotto-espressione (solo istruzione), con errori dedicati.

## `switch` / `case`

- Corpo **`switch`** deve essere **`{ ... }`**.
- **Fall-through** lineare tra `case`: supportato tramite **espansione a compile-time** dei corpi (catena `if disc==v` sulla VM reversibile).
- **`default`**: ammesso; non è obbligatorio che sia ultimo.
- **`break` annidato** dentro `if`/altri costrutti verso lo stesso `switch` → **errore** (servirebbe gating IF/FI non reversibile con lo schema attuale); solo `break` in **coda** al case / catena fall-through.
- Corpo: solo **`case`** e **`default`**.
- **`case`**: costante intera o enumeratore noto; etichette duplicate → errore.

## `for`

- Init solo nei casi gestiti (`DeclList`, `Decl`, `Assignment`, `FuncCall`, `ExprStmt`); altro → `for-init non supportato`.

## Altre istruzioni

- **`goto`**, etichette: **non supportate** (`istruzione non supportata` per `Goto`, ecc.).
- **`setjmp` / `longjmp`**: no.

## Preprocessore

- **`gcc -E`**: macro e `#include` sul testo, ma **nessuna libreria/ABI C reale** aggiunta dal solo preprocessore.

## Estensioni e C moderno

- **`__asm`**, **`_Generic`**, **`typeof`**, attributi compiler-specific: fuori dal lowering Mnemo (parse o errore a valle).

## Kairos / reversibilità

- ** `/` e `%`**: vincoli delle procedure (`__mn_divmod_nonneg`, `__mn_mod_nonneg`, ecc.), non UB C generico.
- Parallelismo: solo ABI **`mnemo_pthread_*`** e mutex Mnemo, non pthread POSIX completo.

## Emitter IR (nota tecnica)

- In `mnemo/emit_kairos.py`, istruzioni IR **`ICopy`**, **`IStoreRev`**, **`IBranch`**, **`IJump`** sono emesse come **commenti** (non Kairos eseguibile). Il lowering attuale non le genera in genere; **`IReturn`** è commento (fine procedura reale via sequenza + `delocal`).

## Riferimento rapido “cosa usare”

- Tipi: `int`, `unsigned`, `bool`, puntatori `int*` / `void*` come da codice; struct/union con limiti sopra.
- `main`: `int main(void)` o `int main(int argc, char **argv)`; `argc` da direttiva o CLI; `argv` fittizio.
- Controllo: `if`, `while`, `do/while`, `for`, `switch` (con limiti su `break` annidato); no `goto`.
- Heap: `malloc`/`free` sul pool; dimensione con `--ptr-pool-size` / layout.

---
> Source: [nicologiuliani6/mnemo](https://github.com/nicologiuliani6/mnemo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
