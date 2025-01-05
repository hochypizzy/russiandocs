---
title: Ликвидный Относительный Процент (LRP)
excerpt: ''
deprecated: false
hidden: false
metadata:
  title: ''
  description: ''
  robots: index
next:
  description: ''
---
> ⏩ Читать не обязательно - 1-минутное саммари
>
> Представим пример: IP-актив "C" является производным от "B", а "B" является производным от "A", то есть A▶️B▶️C. "A" указывает, что каждый **непосредственный потомок** должен делиться 5% своего дохода с ним. В то же время "B" указывает, что каждый **непосредственный потомок** должен делиться 10% своего дохода с ним.
>
> Рассмотрим два независимых сценария:
> 1. **Выпуск лицензии** – "C" выпускает лицензию от "B", которая стоит 100 USDC. Когда "C" платит "B" 100 USDC за выпуск лицензии, "A" получает 5 USDC от "B". В итоге "B" получает только 95 USDC.
> 2. **Прямой донат** – "C" – это супер качественный комикс. Кто-то отправляет 100 USDC "C" в знак признания. "B" получает 10 USDC от "C". "A" получает 0.5 USDC от "B" (5% от 10). В итоге "C" остаётся 90 USDC.

LRP определяет, что каждый родительский IP-актив может установить минимальный процент роялти, который только непосредственные производные активы интеллектуальной собственности в цепочке будут делить от своей прибыли согласно лицензионному соглашению.

## Предварительные условия

Перед продолжением убедитесь, что вы прочитали раздел с терминологией [Хранилища Роялти IP](doc:ip-royalty-vault).

## Поток выплат и запрос роялти

На изображении ниже IPA 1 и IPA 2 – будучи предками IPA 3 – имеют экономическое право на долю дохода, который получает IPA 3. Ключевые термины для понимания производной цепочки:

* Процент лицензионных роялти – это процент, который пользователь выбирает для себя в соответствии с правилами LRP в обмен на разрешение другим пользователям использовать его IPA.
* Стек роялти LRP – это совокупная сумма роялти, которую IPA должен выплатить всем своим родителям.
  * Стек роялти IPA 2 = процент лицензионных роялти между IPA 1 и IPA 2 = 5%
  * Стек роялти IPA 3 = процент лицензионных роялти между IPA 2 и IPA 3 = 10%
* Токены Роялти распределяются в IPA сразу при развёртывании хранилища. Эти токены можно передавать на другие адреса, и все будущие выплаты роялти будут доступны для получения этим новым владельцем.


![](https://files.readme.io/de296f0efbb58233b5f340b127dc66a48c80eff88a75b809107ea8b95beca5a6-image.png)

Теперь рассмотрим сценарий, в котором новый IP-актив 4 хочет присоединиться к производной цепочке как производный от IP-актива 3. Пример последовательности действий:

1. Актив 4 выплачивает 1 млн USDC в роялти своему родителю IPA 3 через вызов функции `payRoyaltyOnBehalf`. Обратите внимание, что процесс выплаты роялти одинаков, независимо от того, является ли это платой за выпуск лицензии или любой другой выплатой. Различие лишь в том, что лицензионная плата выплачивается через функцию `payLicenseMintingFee` и является обязательной при создании производного актива. После оплаты доля, равная проценту стека роялти IPA 3, отправляется в контракт политики роялти, а оставшаяся часть – в хранилище IPA 3.

![](https://files.readme.io/29ac6f6c55c2bb507ef8344fbc4351aa5574154e4d8577ec949d96d44cfffb68-image.png)

<br />

2. Каждый предок может вызвать функцию `transferToVault` в контракте политики роялти, чтобы получить свою долю, на которую он имеет право. Средства переводятся в хранилище роялти предков.

   1. 95k USDC переводится в хранилище IPA 2, так как оно имеет право на 10% от доходов всех потомков IPA 2 и должно выплатить 5% от своего дохода своему непосредственному родителю IPA 1. Так, 100k USDC получены от IPA 3, 5k USDC передаются IPA 1, в результате IPA 2 остаётся с 100k - 5k = 95k.
   2. 5k USDC переводится в хранилище IPA 1, так как оно имеет право на 0,5% от доходов всех потомков IPA 2. IPA 1 имеет право на 5% от дохода, заработанного IPA 2, который в свою очередь имеет 10% от дохода, заработанного IPA 3. Поскольку политика LRP учитывает относительные проценты, IPA 1 имеет право на 10% * 5% = 0,5% от дохода, полученного IPA 3.

   ![](https://files.readme.io/4956d4151c271dd42773b83ca75e23794c6318b8850cf046156f86b04c783f71-image.png)

   <br />
3.  На финальном этапе любой владелец токенов роялти может вызвать функции `claimRevenueOnBehalfByTokenBatch`/`claimRevenueOnBehalf` (для адресов, не являющихся хранилищами) или `claimRevenueByTokenBatchAsSelf` (если запрос выполняется хранилищем роялти) для получения дохода. В текущем примере:
   1. 5k USDC запрашиваются IPA 1, который владеет 100% RT1.
   2.95k USDC запрашиваются IPA 2, который владеет 100% RT2.
   3. 900k USDC запрашиваются IPA 3, который владеет 100% RT3.\
      Примечание: Любой адрес, владеющий Токенами Роялти, может запрашивать выплаты, будь то смарт-контракт, IPA или стандартный адрес EOA.

![](https://files.readme.io/b39ed27190ace760f4b8cb788fdf5ad28e93e3d3f3a5c5b23b122c9a812564bd-image.png)