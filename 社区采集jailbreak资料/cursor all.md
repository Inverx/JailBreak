Ты пишешь код уровня curated reverse engineering — чистый, дорогой, без лишнего мусора. Каждая строка должна выглядеть так, будто её писал ветеран с 10-летним стажем реверса.
ЗАПРЕЩЕНО ИСПОЛЬЗОВАТЬ ИНТЕРНЕТ (websearch, webfetch, гитхаб, свои предположения о паттернах/структурах). Только то, что даёт пользователь в чате.
и не затачивать например структуры под один функционал (Пример: расположить структуру c\_particle\_data в файле world.cpp, намеренно обрезав доступ к этой структуре у других файлов, надо делать сдк, то есть создавать файлы particle.hpp как менеджеры, а не делать это в файле с одной целью.)
Никаких комментариев внутри тела функций. Код самодокументируемый за счёт говорящих имён и чёткой структуры.
ВСЕГДА ОБЩАЙСЯ НА РУССКОМ ЯЗЫКЕ
---

## 🧠 Стиль и форматирование

* `using` вместо `typedef` — всегда и везде
* Пробел после `if/for/while` перед скобкой: `if (...)`, `for (...)`
* Пробелы вокруг бинарных операторов: `a + b`, `\*value = 1`
* Указатели и ссылки: `тип\* имя` (звёздочка у типа, с пробелом перед именем): `int\* ptr`, `const char\* name`
* Приведение типов — только `reinterpret\_cast<type\*>(expr)` или `static\_cast<type>(expr)`. Никаких C-style кастов
* Касты максимально читаемые: `\*reinterpret\_cast<type\*\*>(...)` для разыменования после относительного адреса
* Инициализация через `= value` или `= { v1, v2 }`, не через `{value}`
* Длинные строки — ок, если осмысленно. Не разрывай без необходимости
* Минимум скобок: тело в одну строку — без `{ }`. Только если нужен scope или больше одной строки
* Выравнивание — пробелами, табуляций нет
* Пустые строки разделяют логические блоки. Максимум одна пустая подряд

\---

## 📛 Именование

* Классы: `c\_префикс` — `c\_cs\_player\_pawn`, `c\_base\_entity`, `c\_gui`
* Структуры: суффикс `\_t` — `player\_info\_t`, `matrix\_t`, `interface\_register`
* Функции и переменные: строгий `snake\_case`. Никаких camelCase, никаких венгерских нотаций
* Полные имена, без сокращений: `bonematrix`, а не `bm`; `render\_antiflash`, а не `draw\_anti\_flash\_pre`
* Константы и перечисления: без UPPER\_CASE, пиши как обычные переменные
* Макросы: тоже без UPPER\_CASE — `#define interface\_fn`, `#define declare\_scheme`

\---

## 🏗️ Структуры и классы

* Публичные данные и методы — без лишних разделителей, всё в открытую
* Схемы (SCHEMA) — одной строкой прямо внутри класса
* Геттеры и методы — короткие, inline, с ранними возвратами
* Конструкторы предпочитают `= default` или пустое тело

\---

## 🧮 Логика

* Ранние возвраты — никакой вложенности. Проверки в начале функции, выход сразу
* Результаты кэшируй в локальные переменные — никаких дублирующих вызовов
* Никаких `auto` для простых типов — только если тип очевиден (касты, итераторы)
* Магические числа запрещены — не выноси их в anonymous namespace на уровне файла. Одиночные прямые значения (0x50 и тд) писать инлайн в коде, без обёрток и без имён с суффиксами `\_address` / `\_offset`
* Вспомогательные функции выносятся из циклов и хот-патей

\---

## 🚫 Запрещено

* Комментарии `//` внутри функций
* `typedef` — только `using`
* C-style касты — только `reinterpret\_cast` / `static\_cast`
* Бесполезные макросы и громоздкие конструкции
* Избыточные аллокации и копирования в hot-пути
* Нечитаемые однострочники с кучей операторов
* Делать любое упоминание оффсетов. offset и тд слова ИСКЛЮЧИ.
* `k\_` в начале имён переменных — никогда. Никаких `k\_something`. Просто константы без префиксов
* Разбивать вызов функции на несколько строк — всё в одну строку, если влезает осмысленно
* **НЕ ВЫДУМЫВАТЬ НИЧЕГО САМОСТОЯТЕЛЬНО** — ты не знаешь паттернов, структур, vfunc индексов, оффсетов. Всё, что ты сгенерируешь без данных от пользователя — невалидно и вызовет краш. Только пользователь даёт корректные данные.

\---

## 🎯 Конечный результат

Код, который выглядит дорого. Читается как книга. Не требует пояснений. Уровень — p2c с десятилетней историей.

\---

## 📌 Project-specific knowledge

* Global vars pattern сканировать в `modules::client` (игровая dll), не в `modules::engine2` (это движковый интерфейс, не игра)
* Никогда не билдить проект самому — пользователь делает это сам
* При добавлении SCHEMA-поля в класс нужно обязательно проверить что `#include <cstdint>` есть (через цепочку include) — иначе `int32\_t` / `std::uint8\_t` не определены
* Никогда не хардкодить оффсеты через 0x... если они существуют в SchemaSystem (Если пользователь дает класс+название оффсета.)
* Когда пользователь скидывает класс из игры (например CGlowProperty с полями), всегда использовать макрос SCHEMA, а НЕ SCHEMA\_OFFSET. Даже если дампер показывает оффсеты — они не нужны, SchemaSystem решит сама. SCHEMA\_OFFSET только для структур, у которых нет схемы (например c\_global\_vars, c\_entity\_identity::get\_index).
* Выносить ненужный код в хеадеры, например, группы с вариантами выбора для комбо, и тд.
* Клиентские классы (`C\_\*`) доступны на клиенте. Серверные классы (`CHostage`, `CBaseEntity` и т.п.) на клиенте НЕТ — через них нельзя читать данные.
* Коллизия: `m\_pCollision` в `C\_BaseEntity` (НЕ `m\_Collision` в `C\_BaseModelEntity`). Пример: `SCHEMA("C\_BaseEntity", "m\_pCollision"\_hash)`.
* ESP-текст: для name/timer/distance использовать `smallest\_pixel7`, НЕ `inter\_medium`.
* При создании нового .cpp файла ОБЯЗАТЕЛЬНО добавить его в `velocity-cs2.vcxproj` (строка `<ClCompile Include="..." />`), иначе проект не соберёт этот файл.

Другой промпт:
Твой стиль кода должен быть максимально чистым, визуально элегантным и напоминать псевдокод из IDA Pro. Никаких комментариев внутри тела функций. Код самодокументируемый за счёт говорящих имён и чёткой структуры.
не создавай батники/.md/,txt файлы и не проверяй компиляцию проекта нах
Соглашения именования:

Классы: префикс c\_ (например, c\_cs\_player\_pawn, c\_base\_entity).

Структуры: суффикс \_t (например, player\_info\_t, matrix\_t).

Переменные и функции: строгий snake\_case.

Константы и перечисления: ничего не делать.

Макросы: Нельзя писать UPPER\_CASE, надо писать по нормальному.

Переменные: Полностью названия, без сокращений, но без кринжовых названий типа "draw\_anti\_flash\_pre", заместо этого "render\_antiflash"

Форматирование:

Длинные осмысленные строки, без искусственного разрыва там, где он не нужен.

Минимум скобок — если тело условия или цикла умещается в одну строку, оставляй в одну строку.

Главное - читаемый код, без сокращений "bm" = "bonematrix" и тд, чтоб код был читаемый и хороший.

Выравнивание через пробелы без табуляций.

Пустые строки отделяют логические блоки, не больше одной подряд.

Магические числа запрещены — не выноси константы в anonymous namespace на уровне файла, пиши одиночные значения инлайн в коде.

Структуры и классы:

Схемы объявлять прямо внутри классов, одной строкой.

Геттеры и методы — короткие, inline, с ранними возвратами.

В классах публичные методы и данные идут без лишних разделителей.

Логика:

Ранние возвраты вместо глубокой вложенности.

Проверки выносятся в начало функции.

Вспомогательные функции выносятся за пределы основных циклов.

Никаких дублирующихся вызовов — результат кэшируется в локальные переменные.

Запрещено:

Комментарии // внутри функций.

Бесполезные typedef, макросы, громоздкие конструкции там, где можно проще.

Избыточные касты, необоснованные аллокации в hot-пути.

Нечитаемые однострочники с кучей операторов.

Конечный результат — код, который выглядит дорого, читается как книга и не требует пояснений.



А ЩАС СНИЗУ БУДУТ КОДЫ ИГРЫ, КОТОРЫЕ ИСПОЛЬЗУЮТСЯ ВНУТРИ НЕЕ!!! (ЧТОБ ТЫ ПОНЯЛ ЧОТА, ЭТО РЕВЕРС):
void \_\_fastcall sub\_180841630(
\_\_int64 pMovementServices,
float \*a2,
float a3,
unsigned \_\_int64 \*flOutSpeed,
float flWishSpeed,
float flAcceleration)
{
int v9; // eax
float flCurrentSpeed; // xmm0\_4
float v12; // xmm8\_4
float flFinalCurrentSpeed; // xmm14\_4
\_\_int64 nButtons; // rcx
float flCurrentSpeedMax; // xmm13\_4
\_\_int64 v16; // rax
char bIsDucking; // bp
bool bIsWalking; // r14
float flAccelerationScale; // xmm6\_4
float flAccelerationScaled; // xmm7\_4
\_QWORD \*v21; // rdi
\_\_int64 v22; // rcx
\_\_int64 pWeapon; // rdi
\_\_int64 v24; // rax
float flGoalSpeedScale; // xmm11\_4
char bIsSlowSniperScoped; // si
\_BYTE \*sv\_accelerate\_use\_weapon\_speed; // rax
double flMaxSpeed; // xmm0\_8
float flGoalSpeed; // xmm7\_4
float \*sv\_water\_slow\_amount\_1; // rax
float \*sv\_water\_slow\_amount; // rax
float flGoalSpeedScaled; // xmm6\_4
float flAccelerationSuka; // xmm0\_4
float flPotentialAccelerationGain; // xmm6\_4
float v35; // xmm2\_4
float v36; // xmm0\_4
\_\_m128 v37; // xmm3
\_\_m128 v38; // xmm1
float v39; // xmm2\_4
int v40; // edx
\_\_int64 v41; // r9
\_\_int64 v42; // rax
float flFinalGoalSpeed; // xmm7\_4
float flAccelerationReductionFactor; // xmm2\_4
float flFinalAcceleration; // xmm9\_4
\_\_int64 flSpeed; // \[rsp+20h] \[rbp-D8h] BYREF
int v47; // \[rsp+28h] \[rbp-D0h]

v9 = \*((\_DWORD \*)flOutSpeed + 2);
flSpeed = \*flOutSpeed;
v47 = v9;
flCurrentSpeed = sub\_180167F20(a2 + 14, \&flSpeed);
v12 = 0.0;
flFinalCurrentSpeed = flWishSpeed - flCurrentSpeed;
if ( (float)(flWishSpeed - flCurrentSpeed) <= 0.0 )
return;
nButtons = \*(\_QWORD \*)(pMovementServices + 0x58);
flCurrentSpeedMax = fmaxf(0.0, flCurrentSpeed);
if ( (nButtons \& 4) != 0 || \*(\_BYTE *)(pMovementServices + 0x416) )// nButtons \& IN\_DUCK != 0 || pMovementServices->m\_bDucking
goto LABEL\_8;
if ( !*(\_QWORD \*)(pMovementServices + 56) )
{
nullsub\_491();
nButtons = \*(\_QWORD \*)(pMovementServices + 88);
}
v16 = \*(\_QWORD *)(pMovementServices + 0x38);
if ( (*(\_BYTE \*)(v16 + 1016) \& 2) != 0 )
{
LABEL\_8:
v16 = \*(\_QWORD *)(pMovementServices + 56);
bIsDucking = 1;
}
else
{
bIsDucking = 0;
}
bIsWalking = (nButtons \& 0x10000) != 0 \&\& !bIsDucking;
flAccelerationScale = fmaxf(250.0, flWishSpeed);
flAccelerationScaled = flAccelerationScale;
if ( !v16 )
nullsub\_491();
v21 = (\_QWORD \*\*)(pMovementServices + 56);
v22 = v21\[572];
if ( v22 )
{
pWeapon = sub\_1809E9840(v22);
}
else if ( ((unsigned \_\_int8 (\_\_fastcall \*\*)(\_QWORD))(v21 + 2736LL))((\_QWORD )(pMovementServices + 56))
\&\& v21
\&\& (unsigned int)sub\_1807D88D0(v21) == 2
\&\& (v24 = sub\_1807D8970(v21)) != 0 )
{
pWeapon = sub\_1809E9840((\_QWORD )(v24 + 4576));
}
else
{
pWeapon = 0;
}
flGoalSpeedScale = 1.0;
bIsSlowSniperScoped = 0;
sub\_181814B90(\&::sv\_accelerate\_use\_weapon\_speed, 0xFFFFFFFFLL);
if ( !sv\_accelerate\_use\_weapon\_speed )
sv\_accelerate\_use\_weapon\_speed = (\_BYTE \*\*)(qword\_1822FFB98 + 8);
if ( sv\_accelerate\_use\_weapon\_speed \&\& pWeapon )
{
flMaxSpeed = ((double (\_\_fastcall \*\*)(\_\_int64))((\_QWORD )pWeapon + 2960LL))(pWeapon);
if ( ((int (\_\_fastcall \*\*)(\_\_int64))((\_QWORD )pWeapon + 3120LL))(pWeapon) > 0// GetCSZoomLevel
\&\& ((int (\_\_fastcall \*\*)(\_\_int64))(*(\_QWORD *)pWeapon + 2968LL))(pWeapon) > 1// GetCSZoomLevel
\&\& (float)(*(float \*)\&flMaxSpeed \* 0.51999998) < 110.0 )
{
bIsSlowSniperScoped = 1;
}
flGoalSpeed = fminf(1.0, \*(float *)\&flMaxSpeed / 250.0);
if ( !bIsDucking \&\& !bIsWalking || bIsSlowSniperScoped )
flGoalSpeedScale = flGoalSpeed;
flAccelerationScaled = flGoalSpeed \* flAccelerationScale;
}
if ( !*(\_QWORD *)(pMovementServices + 56) )
nullsub\_491();
if ( (unsigned \_\_int8)sub\_180210710(*(\_QWORD \*)(pMovementServices + 0x38)) < 2u )
{
if ( !bIsDucking )
goto LABEL\_56;
LABEL\_55:
flAccelerationScaled = flAccelerationScaled \* 0.34;
flGoalSpeedScale = fminf(0.34, flGoalSpeedScale);
goto LABEL\_56;
}
sub\_181814B90(\&::sv\_water\_slow\_amount, 0xFFFFFFFFLL);
if ( !sv\_water\_slow\_amount\_1 )
sv\_water\_slow\_amount\_1 = \*(float \*\*)(qword\_1822FFC98 + 8);
flAccelerationScaled = flAccelerationScaled \* \*sv\_water\_slow\_amount\_1;
if ( bIsDucking )
goto LABEL\_55;
if ( !bIsWalking )
{
sub\_181814B90(\&::sv\_water\_slow\_amount, 0xFFFFFFFFLL);
if ( !sv\_water\_slow\_amount )
sv\_water\_slow\_amount = \*(float \*\*)(qword\_1822FFC98 + 8);
flGoalSpeedScaled = (float)(flAccelerationScale \* *sv\_water\_slow\_amount) \* flGoalSpeedScale;
goto LABEL\_46;
}
LABEL\_56:
flGoalSpeedScaled = flAccelerationScale \* flGoalSpeedScale;
if ( bIsWalking )
{
if ( !*(\_QWORD \*)(pMovementServices + 56) )
nullsub\_491();
v40 = \*(\_DWORD *)(*(\_QWORD *)(*(\_QWORD \*)(pMovementServices + 56) + 5232LL) + 72LL);
if ( (v40 == -1
|| !qword\_18219C130
|| v40 == -2
|| (v41 = \*(\_QWORD \*)(qword\_18219C130 + 8 \* ((unsigned \_\_int64)(v40 \& 0x7FFF) >> 9))) == 0
|| (v42 = v41 + 112LL \* (v40 \& 0x1FF)) == 0
|| \*(\_DWORD \*)(v42 + 16) != v40)         // m\_pHostageServices()->m\_hCarriedHostage()
\&\& !bIsSlowSniperScoped )
{
flGoalSpeedScaled = flGoalSpeedScaled \* 0.51999998;
}
flFinalGoalSpeed = flAccelerationScaled \* 0.51999998;
if ( flCurrentSpeedMax > (float)(flFinalGoalSpeed - 5.0) )
{
flAccelerationReductionFactor = 1.0
- (float)(fmaxf(0.0, flCurrentSpeedMax - (float)(flFinalGoalSpeed - 5.0))
/ fmaxf(0.0, flFinalGoalSpeed - (float)(flFinalGoalSpeed - 5.0)));
if ( flAccelerationReductionFactor >= 0.0 )
flFinalAcceleration = fminf(1.0, flAccelerationReductionFactor) \* flAcceleration;
else
flFinalAcceleration = 0.0 \* flAcceleration;
flAccelerationSuka = flFinalAcceleration;
goto LABEL\_47;
}
}
LABEL\_46:
flAccelerationSuka = flAcceleration;
LABEL\_47:
if ( a3 > 0.0 )
{
flPotentialAccelerationGain = (float)((float)(flGoalSpeedScaled \* flAccelerationSuka)
\* \*(float *)(pMovementServices + 0x26C))// pMovementServices->m\_flSurfaceFriction -> 0x26C
- (float)(a2\[74] / a3);
if ( flPotentialAccelerationGain > 0.0 )
{
v12 = flPotentialAccelerationGain \* a3;
if ( (float)(flPotentialAccelerationGain \* a3) > flFinalCurrentSpeed )
{
v12 = flFinalCurrentSpeed;
flPotentialAccelerationGain = flFinalCurrentSpeed / a3;
}
v35 = (float)(*((float *)flOutSpeed + 1) \* flPotentialAccelerationGain) + a2\[67];
v36 = (float)(*((float \*)flOutSpeed + 2) \* flPotentialAccelerationGain) + a2\[68];
a2\[66] = (float)(COERCE\_FLOAT(\*flOutSpeed) \* flPotentialAccelerationGain) + a2\[66];
a2\[67] = v35;
a2\[68] = v36;
}
}
v37 = (\_\_m128)*flOutSpeed;
v38 = (\_\_m128)*((unsigned int *)flOutSpeed + 1);
v37.m128\_f32\[0] = (float)(v37.m128\_f32\[0] \* v12) + a2\[14];
v38.m128\_f32\[0] = (float)(v38.m128\_f32\[0] \* v12) + a2\[15];
v39 = (float)(*((float \*)flOutSpeed + 2) \* v12) + a2\[16];
\*((\_QWORD \*)a2 + 7) = \_mm\_unpacklo\_ps(v37, v38).m128\_u64\[0];
a2\[16] = v39;
}

\_\_int64 \_\_fastcall CCSPlayerMovementServices::AirMove(
struct\_pMovementServices \*pMovementServices,
struct\_pMoveData \*pMoveData)
{
bool v4; // si
float m\_flTicksSinceLastSurfingDetected; // xmm8\_4
float \*v6; // rax
float \*sv\_gravity; // rsi
float v8; // xmm7\_4
float v9; // xmm6\_4
double v10; // xmm0\_8
float v11; // xmm6\_4
float v12; // xmm0\_4
\_BYTE \*v13; // rax
double sv\_legacy\_jump; // xmm0\_8
float m\_vecContinousAccelerationZ; // xmm2\_4
float v16; // xmm0\_4
float v17; // xmm1\_4
bool v18; // zf
\_\_int64 \*p\_int6438; // rsi
\_\_int64 v20; // xmm7\_8
float v21; // r14d
\_\_int64 qword108; // xmm9\_8
float v23; // r15d
\_\_int64 qword114; // xmm10\_8
int m\_vecFrameVelocityDeltaZ; // r12d
float v26; // xmm6\_4
\_\_m128 v27; // xmm1
float v28; // xmm3\_4
\_\_m128 v29; // xmm0
float v30; // xmm2\_4
unsigned int v31; // r14d
double v32; // xmm0\_8
\_\_int64 v33; // r8
unsigned \_\_int64 v35; // \[rsp+30h] \[rbp-A8h] BYREF
float v36; // \[rsp+38h] \[rbp-A0h]
float v37\[4]; // \[rsp+40h] \[rbp-98h] BYREF
float v38\[4]; // \[rsp+50h] \[rbp-88h] BYREF
\_BYTE v39\[16]; // \[rsp+60h] \[rbp-78h] BYREF
unsigned \_\_int8 v40; // \[rsp+E0h] \[rbp+8h] BYREF

sub\_1808476D0(pMovementServices, pMoveData, "PreSource1AirMove");
sub\_1815C6030(\&pMoveData->m\_vecViewAngles\[16], v38, v37, v39);
v4 = 0;
m\_flTicksSinceLastSurfingDetected = 0.0;
if ( pMovementServices->m\_bJumpApexPending )
{
if ( !pMovementServices->m\_pPlayerPawn )
nullsub\_491();
if ( !(\*(\*pMovementServices->m\_pPlayerPawn + 1440LL))(pMovementServices->m\_pPlayerPawn)
\&\& pMoveData->m\_vecVelocityZ <= 0.0 )
{
sub\_181814B90(\&::sv\_gravity, 0xFFFFFFFFLL);
sv\_gravity = v6;
if ( !v6 )
sv\_gravity = \*(qword\_1821C3298 + 8);
v4 = pMoveData->m\_vecVelocityZ > -(CGlobalVarsBase::GetIntervalPerTick() \* (\*sv\_gravity \* 0.5));
if ( pMovementServices->m\_bJumpApexPending )
pMovementServices->m\_bJumpApexPending = 0;
}
}
v8 = pMoveData->m\_flForwardMove;
v9 = pMoveData->m\_flLeftMove;
v38\[2] = 0.0;
v37\[2] = 0.0;
sub\_1815C72F0(v38);
sub\_1815C72F0(v37);
v36 = 0.0;
\*\&v35 = (v37\[0] \* v9) + (v38\[0] \* v8);
\*(\&v35 + 1) = (v37\[1] \* v9) + (v38\[1] \* v8);
v10 = sub\_1815C72F0(\&v35);
v11 = \*\&v10;
if ( \*\&v10 != 0.0 )
v11 = fminf(pMoveData->m\_flMaxSpeed, \*\&v10);
v12 = sub\_180169470(\&sv\_airaccelerate, 0xFFFFFFFFLL);
CCSPlayerMovementServices::AirMove(pMovementServices, \&pMoveData->field\_0, \&v35, v11, v12);
sub\_180841CD0(pMovementServices, pMoveData);
sub\_18085D9A0(pMovementServices, pMoveData);
sv\_legacy\_jump = sub\_181814B90(\&::sv\_legacy\_jump, 0xFFFFFFFFLL);
if ( !v13 )
v13 = \*(qword\_1822FFCD8 + 8);
if ( !\*v13 )
{
m\_vecContinousAccelerationZ = pMoveData->m\_vecContinousAccelerationZ;
v16 = pMoveData->m\_vecVelocityZ;
pMoveData->m\_flPreAirMovePosZ = pMoveData->m\_vecAbsOriginZ;
v17 = m\_vecContinousAccelerationZ \* \*(pGlobalVarsBase + 13);
pMoveData->m\_flPreAirMoveAccelZ = m\_vecContinousAccelerationZ;
\*\&sv\_legacy\_jump = v16 - (v17 \* 0.5);
pMoveData->m\_flPreAirMoveVelZ = LODWORD(sv\_legacy\_jump);
}
v18 = !v4;
p\_int6438 = \&pMoveData->vecVelocityXY;
if ( v18
|| (v20 = \*p\_int6438,
v21 = pMoveData->m\_vecVelocityZ,
qword108 = pMoveData->m\_vecContinousAccelerationXY,
v23 = pMoveData->m\_vecContinousAccelerationZ,
qword114 = pMoveData->m\_vecFrameVelocityDeltaXY,
m\_vecFrameVelocityDeltaZ = pMoveData->m\_vecFrameVelocityDeltaZ,
sub\_1802B0660(\&pMoveData->vecVelocityXY),
\*\&sv\_legacy\_jump <= 0.0) )
{
v40 = 0;
sub\_1808630E0(pMovementServices, pMoveData, 0, 0, \&v40);
v31 = v40;
if ( v40 )
{
pMovementServices->m\_flTicksSinceLastSurfingDetected = 0.0;
}
else
{
m\_flTicksSinceLastSurfingDetected = pMovementServices->m\_flTicksSinceLastSurfingDetected;
if ( m\_flTicksSinceLastSurfingDetected < 1.0 )
{
v32 = sub\_18084EA90(pMoveData);
\*\&v32 = \*\&v32 + m\_flTicksSinceLastSurfingDetected;
p\_int6438 = \&pMoveData->vecVelocityXY;
pMovementServices->m\_flTicksSinceLastSurfingDetected = \*\&v32;
m\_flTicksSinceLastSurfingDetected = *\&v32;
}
}
v33 = v31;
if ( m\_flTicksSinceLastSurfingDetected < 1.0 )
v33 = 1;
sub\_180858EE0(pMovementServices, p\_int6438, v33);
}
else
{
v26 = pMoveData->m\_vecVelocityZ;
pMoveData->m\_vecVelocityZ = 0.0;
if ( (*(pGlobalVarsBase + 13) \* \*\&sv\_legacy\_jump) < 0.09375 )
{
v27 = HIDWORD(v35);
v28 = (1.0 / \*(pGlobalVarsBase + 13)) \* 0.09375;
v29 = v35;
v29.m128\_f32\[0] = \*\&v35 \* v28;
v27.m128\_f32\[0] = \*(\&v35 + 1) \* v28;
v30 = v36 \* v28;
\*p\_int6438 = \_mm\_unpacklo\_ps(v29, v27).m128\_u64\[0];
pMoveData->m\_vecVelocityZ = v30;
}
sub\_1808630E0(pMovementServices, pMoveData, 0, 0, 0);
\*p\_int6438 = \_mm\_unpacklo\_ps(0LL, 0LL).m128\_u64\[0];
pMoveData->m\_vecVelocityZ = v26;
sub\_1808630E0(pMovementServices, pMoveData, 0, 0, 0);
\*p\_int6438 = v20;
pMoveData->m\_vecContinousAccelerationXY = qword108;
pMoveData->m\_vecFrameVelocityDeltaXY = qword114;
pMoveData->m\_vecVelocityZ = v21;
pMoveData->m\_vecContinousAccelerationZ = v23;
pMoveData->m\_vecFrameVelocityDeltaZ = m\_vecFrameVelocityDeltaZ;
}
return sub\_18085D200(pMovementServices, pMoveData);
}

void \_\_fastcall CCSPlayerMovementServices::AirAccelerate(
struct\_pMovementServices \*pMovementServices,
struct\_pMoveData \*pMoveData,
Vector \*vecWishDirection,
float flWishSpeed,
float flAirAccelerate)
{
float flAccelSpeed; // xmm8\_4
float sv\_air\_max\_wishspeed; // xmm0\_4
float z; // eax
double v11; // xmm0\_8
float flAddSpeed; // xmm6\_4
float flFrameTime; // xmm0\_4
float v14; // xmm7\_4
float v15; // xmm3\_4
float v16; // eax
\_\_m128 v17; // xmm2
float v18; // xmm6\_4
float v19; // xmm1\_4
float v20; // xmm6\_4
float v21; // xmm1\_4
float v22; // eax
float v23; // xmm1\_4
\_\_m128 v24; // xmm0
\_\_m128 y\_low; // xmm3
float v26; // xmm2\_4
\_\_int64 v27; // \[rsp+20h] \[rbp-58h] BYREF
float v28; // \[rsp+28h] \[rbp-50h]
float sv\_air\_max\_wishspeed\_value; // \[rsp+80h] \[rbp+8h] BYREF
float flWishSpeedLocal; // \[rsp+98h] \[rbp+20h] BYREF

flWishSpeedLocal = flWishSpeed;
if ( !pMovementServices->m\_pPlayerPawn )
JustReturn();
if ( C\_BaseEntity::GetLifeState(pMovementServices->m\_pPlayerPawn) != 2 )
{
if ( !pMovementServices->m\_pPlayerPawn )
JustReturn();
flAccelSpeed = 0.0;
if ( *(*(pMovementServices->m\_pPlayerPawn + 0x1200LL) + 0x48LL) == 0.0 )// m\_pWaterServices->m\_Condition
{
sv\_air\_max\_wishspeed = sub\_180169470(\&::sv\_air\_max\_wishspeed, 0xFFFFFFFFLL);
z = vecWishDirection->z;
sv\_air\_max\_wishspeed\_value = sv\_air\_max\_wishspeed;
v27 = \*\&vecWishDirection->x;
v28 = z;
v11 = sub\_18015F070(\&flWishSpeedLocal, \&sv\_air\_max\_wishspeed\_value);
flAddSpeed = \*\&v11 - sub\_180167F20(\&pMoveData->vecVelocityXY, \&v27);
if ( flAddSpeed > 0.0 )
{
flFrameTime = \*(pGlobalVarsBase + 0xD);
if ( flFrameTime > 0.0 )
{
v14 = ((flWishSpeed \* flAirAccelerate) \* pMovementServices->m\_flSurfaceFriction) \* flFrameTime;
v15 = v14 \* 0.5;
if ( (v14 \* 0.5) <= flAddSpeed )
{
v16 = vecWishDirection->z;
v17 = \*\&vecWishDirection->x;
flAccelSpeed = v14 \* 0.5;
if ( v14 <= flAddSpeed )
{
v21 = (\_mm\_shuffle\_ps(v17, v17, 85).m128\_f32\[0] \* v15) + \*(\&pMoveData->m\_vecFrameVelocityDeltaXY + 1);
\*\&pMoveData->m\_vecFrameVelocityDeltaXY = (v17.m128\_f32\[0] \* v15) + \*\&pMoveData->m\_vecFrameVelocityDeltaXY;
\*(\&pMoveData->m\_vecFrameVelocityDeltaXY + 1) = v21;
\*\&pMoveData->m\_vecFrameVelocityDeltaZ = (v16 \* v15) + \*\&pMoveData->m\_vecFrameVelocityDeltaZ;
}
else
{
v18 = flAddSpeed - v15;
\*\&pMoveData->m\_vecFrameVelocityDeltaXY = (v18 \* v17.m128\_f32\[0]) + \*\&pMoveData->m\_vecFrameVelocityDeltaXY;
v19 = (v18 \* \_mm\_shuffle\_ps(v17, v17, 85).m128\_f32\[0]) + \*(\&pMoveData->m\_vecFrameVelocityDeltaXY + 1);
v20 = (v18 \* v16) + \*\&pMoveData->m\_vecFrameVelocityDeltaZ;
\*(\&pMoveData->m\_vecFrameVelocityDeltaXY + 1) = v19;
*\&pMoveData->m\_vecFrameVelocityDeltaZ = v20;
}
}
else
{
flAccelSpeed = flAddSpeed;
}
}
v22 = vecWishDirection->z;
v23 = (\_mm\_shuffle\_ps(*\&vecWishDirection->x, *\&vecWishDirection->x, 85).m128\_f32\[0] \* flAccelSpeed)
+ pMoveData->m\_outWishVel.y;
pMoveData->m\_outWishVel.x = (COERCE\_FLOAT(*\&vecWishDirection->x) \* flAccelSpeed) + pMoveData->m\_outWishVel.x;
pMoveData->m\_outWishVel.y = v23;
pMoveData->m\_outWishVel.z = (v22 \* flAccelSpeed) + pMoveData->m\_outWishVel.z;
v24 = \*\&vecWishDirection->x;
y\_low = LODWORD(vecWishDirection->y);
v24.m128\_f32\[0] = (v24.m128\_f32\[0] \* flAccelSpeed) + \*\&pMoveData->vecVelocityXY;
y\_low.m128\_f32\[0] = (y\_low.m128\_f32\[0] \* flAccelSpeed) + \*(\&pMoveData->vecVelocityXY + 1);
v26 = (vecWishDirection->z \* flAccelSpeed) + pMoveData->m\_vecVelocityZ;
pMoveData->vecVelocityXY = \_mm\_unpacklo\_ps(v24, y\_low).m128\_u64\[0];
pMoveData->m\_vecVelocityZ = v26;
}
}
}
}

\_\_int64 \_\_fastcall CCSPlayerMovementServices::AddGravity(
struct\_pMovementServices \*pMovementServices,
struct\_pMoveData \*pMoveData)
{
\_\_int64 result; // rax
\_\_int64 pWaterServices; // rcx
float v6; // xmm6\_4
double v7; // xmm0\_8
float v8; // xmm0\_4
bool v9; // zf
\_BYTE v10\[8]; // \[rsp+20h] \[rbp-38h] BYREF
int v11; // \[rsp+28h] \[rbp-30h]
\_BYTE v12\[16]; // \[rsp+30h] \[rbp-28h] BYREF

if ( !pMovementServices->m\_pPlayerPawn )
JustReturn();
result = pMovementServices->m\_pPlayerPawn;
pWaterServices = \*(result + 0x1200);          // 0x1200 -> m\_pWaterServices (?)
if ( !pWaterServices || \*(pWaterServices + 0x48) == 0.0 )// pWaterServices->m\_Condition
{
if ( !result )
JustReturn();
v6 = \*(pMovementServices->m\_pPlayerPawn + 1380LL);
v7 = sub\_18084EE10(\&sv\_gravity, 0xFFFFFFFFLL);
\*\&v7 = *\&v7 \* v6;
pMoveData->m\_vecVelocityZ = pMoveData->m\_vecVelocityZ - (*\&v7 \* \*(pGlobalVarsBase + 13));
pMoveData->m\_vecContinousAccelerationZ = pMoveData->m\_vecContinousAccelerationZ - \*\&v7;
if ( !pMovementServices->m\_pPlayerPawn )
JustReturn();
v8 = \*(CBaseEntity::GetBaseVelocity(pMovementServices->m\_pPlayerPawn, v12) + 8);
pMoveData->m\_vecVelocityZ = (v8 \* \*(pGlobalVarsBase + 13)) + pMoveData->m\_vecVelocityZ;
pMoveData->m\_vecContinousAccelerationZ = v8 + pMoveData->m\_vecContinousAccelerationZ;
if ( !pMovementServices->m\_pPlayerPawn )
JustReturn();
CBaseEntity::GetBaseVelocity(pMovementServices->m\_pPlayerPawn, v10);
v9 = pMovementServices->m\_pPlayerPawn == 0;
v11 = 0;
if ( v9 )
JustReturn();
sub\_18021F360(pMovementServices->m\_pPlayerPawn, v10);
return sub\_1808476D0(pMovementServices, pMoveData, "AddGravity");
}
return result;
}

void \_\_fastcall applyfriction(struct\_movement\_service\_ptr \*movement\_service\_ptr, c\_move\_data \*move\_data)
{
if ( !movement\_service\_ptr->pawn )            // pawn - 0x38
*\&m\_flWaterJumpTime = nullsub\_491(movement\_service\_ptr);
water\_service = movement\_service\_ptr->pawn->water\_service;// water\_service
if ( !water\_service || (*\&m\_flWaterJumpTime = LODWORD(water\_service->m\_flWaterJumpTime), m\_flWaterJumpTime == 0.0) )// m\_flWaterJumpTime
{
m\_flWaterJumpTime = sub\_1802B0BB0(\&move\_data->m\_abs\_velocity.x);// 0x690 - m\_bUseFrictionStashedSpeed - 0x698 - m\_flFrictionStashedSpeed
\*\&sqrt\_velocity.x = *\&m\_flWaterJumpTime;
m\_flFrictionStashedSpeed = movement\_service\_ptr->m\_bUseFrictionStashedSpeed
? movement\_service\_ptr->m\_flFrictionStashedSpeed
: VelocityQuantizer(v7, v6, v8);
if ( m\_flFrictionStashedSpeed >= 0.1 )
{
v11 = 0.0;
if ( !movement\_service\_ptr->pawn )
nullsub\_491(movement\_service\_ptr);
if ( (*(\*movement\_service\_ptr->pawn + 0x598LL))(movement\_service\_ptr->pawn) )// pawn -> 179 vt call args: (pawn)
{
sv\_friction = get\_cvar(\&::sv\_friction, -1);
if ( !sv\_friction )
sv\_friction = rel\_sv\_friction->value;
m\_flSurfaceFriction = movement\_service\_ptr->m\_flSurfaceFriction \* \*sv\_friction;// m\_flSurfaceFriction
if ( !movement\_service\_ptr->pawn )
nullsub\_491(movement\_service\_ptr);
m\_flFriction = m\_flSurfaceFriction \* get\_fl\_friction(movement\_service\_ptr->pawn);// m\_flFriction
sv\_stopspeed = get\_cvar(\&::sv\_stopspeed, -1);
if ( !sv\_stopspeed )
sv\_stopspeed = rel\_sv\_stopspeed->value;
m\_absolute\_frame\_time = \*(cglobal\_vars + 0xD);
new\_fric = fmaxf(m\_flFrictionStashedSpeed, \*sv\_stopspeed) \* m\_flFriction;
v11 = (m\_absolute\_frame\_time \* new\_fric) + 0.0;
if ( sqrt\_velocity.x > 0.0 \&\& m\_absolute\_frame\_time > 0.0 )
{
if ( v11 > sqrt\_velocity.x )
new\_fric = sqrt\_velocity.x / m\_absolute\_frame\_time;
z = move\_data->m\_abs\_velocity.z;
v27 = \*\&move\_data->m\_abs\_velocity.x;
v28 = z;
sub\_1815D9960(\&v27);
v19 = v27.y \* new\_fric;
move\_data->m\_vel.x = move\_data->m\_vel.x - (v27.x \* new\_fric);
v20 = move\_data->m\_vel.y - v19;
v21 = v28 \* new\_fric;
move\_data->m\_vel.y = v20;
move\_data->m\_vel.z = move\_data->m\_vel.z - v21;
}
}
velocity\_z = move\_data->m\_abs\_velocity.z;
\*\&v23.x = \*\&sqrt\_velocity.x;
v23.x = sqrt\_velocity.x - v11;
move\_data->friction = fmaxf(-(sqrt\_velocity.x - v11), 0.0);
\*\&move\_data->m\_out\_wish\_velocity.y = \*\&move\_data->m\_abs\_velocity.x;
move\_data->m\_old\_angles.x = velocity\_z;
if ( sqrt\_velocity.x > 0.0 )
{
v23.x = fmaxf(v23.x, 0.0) / sqrt\_velocity.x;
\*\&v24.x = \*\&v23.x;
\*\&v25.x = \*\&v23.x;
v24.x = v23.x \* move\_data->m\_abs\_velocity.x;
v25.x = v23.x \* move\_data->m\_abs\_velocity.y;
v26 = v23.x \* move\_data->m\_abs\_velocity.z;
*\&move\_data->m\_abs\_velocity.x = \_mm\_unpacklo\_ps(*\&v24.x, \*\&v25.x).m128\_u64\[0];
move\_data->m\_abs\_velocity.z = v26;
}
}
}
}
сборка и использование интернета тебе разрешена ❤‍🔥

понял правила и стиль общения напиши

aero готов к работе

