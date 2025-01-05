---
title: Ликвидный Абсолютный Процент (LAP)
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
> Представим пример: IP-актив "C" является производным от "B", а "B" – производным от "A", то есть A▶️B▶️C. "A" указывает, что любой потомок должен делиться 5% своего дохода с ним. В то же время "B" указывает, что любой потомок должен делиться 10% дохода с ним.
>
> Рассмотрим два независимых сценария:
>
> 1. **Выпуск лицензии** – "C" выпускает лицензию от "B", которая стоит 100 USDC. Когда "C" платит "B" 100 USDC за выпуск лицензии, "A" получает 5 USDC от "B". В результате "B" получает только 95 USDC.
> 2. **Прямой донат** – "C" – это супер качественный комикс. Кто-то отправляет 100 USDC "C" в знак признания. "A" получает 5 USDC от "C". "B" получает 10 USDC от "C". В итоге "C" остаётся 85 USDC.

LAP определяет, что каждый родительский IP-актив может установить минимальный процент роялти, которым все его потомки в производной цепочке обязаны делиться от своей прибыли согласно лицензионному соглашению.

## Предварительные условия

Перед продолжением убедитесь, что вы прочитали раздел с терминологией [Хранилища Роялти IP](doc:ip-royalty-vault).

## Поток выплат и запрос роялти

На изображении ниже IPA 1 и IPA 2 – будучи предками IPA 3 – имеют экономическое право на долю дохода, который получает IPA 3. Ключевые термины для понимания производной цепочки:

* Процент лицензионных роялти – это процент, который пользователь выбирает для себя в соответствии с правилами LAP в обмен на разрешение другим пользователям использовать его IPA.
  * Стек роялти IPA 2 = стек роялти IPA 1 + процент лицензионных роялти между IPA 1 и IPA 2 = 0% + 5% = 5%
  * Стек роялти IPA 3 = стек роялти IPA 2 + процент лицензионных роялти между IPA 2 и IPA 3 = 5% + 10% = 15%
* Токены Роялти распределяются в IPA сразу при развёртывании хранилища. Эти токены можно передавать на другие адреса, и все будущие выплаты роялти будут доступны для получения этим новым владельцем.

![](https://files.readme.io/dcfb36162a224000d960fe9d6ca451bf7679500a31e727fb7205d22cb3581391-image.png)

Теперь рассмотрим сценарий, в котором новый IP-актив 4 хочет присоединиться к производной цепочке как производный от IP-актива 3. Пример последовательности действий:

1. Актив 4 выплачивает 1 млн USDC в роялти своему родителю IPA 3 через вызов функции `payRoyaltyOnBehalf`. Обратите внимание, что процесс роялти одинаков независимо от того, является ли это платой за выпуск лицензии или любой другой выплатой. Различие лишь в том, что лицензионная плата выплачивается через функцию `payLicenseMintingFee` и является обязательной при создании производного актива. После оплаты доля, равная проценту стека роялти IPA 3, отправляется в контракт политики роялти, а оставшаяся часть – в хранилище IPA 3.

![](https://files.readme.io/5b32b2bbdefe8e9d4c0a36016360b1b0bddaed899c889586df476ae736032478-image.png)

2. Каждый предок может вызвать функцию `transferToVault` в контракте политики роялти, чтобы получить свою долю, на которую он имеет право. Средства переводятся в хранилище роялти предков.
   1. 100k USDC переводится в хранилище IPA 2, так как оно имеет право на 10% от доходов всех потомков IPA 2.
   2. 50k USDC переводится в хранилище IPA 1, так как оно имеет право на 5% от доходов всех потомков IPA 2.

![](https://files.readme.io/76059536b28ba6daee35328f15baa4995850a5c54f00a1bcf863620e04a0eba3-image.png)

3. На финальном этапе любой владелец токенов роялти может вызвать функции `claimRevenueOnBehalfByTokenBatch`/`claimRevenueOnBehalf` (для адресов, не являющихся хранилищами) или `claimRevenueByTokenBatchAsSelf` (если запрос выполняется хранилищем роялти) для получения дохода. В текущем примере:
   1. 50k USDC запрашивается IPA 1, который владеет 100% RT1.
   2. 100k USDC запрашивается IPA 2, который владеет 100% RT2.
   3. 850k USDC запрашивается IPA 3, который владеет 100% RT3\
      Примечание: Любой адрес, владеющий Токенами Роялти, может запрашивать выплаты, будь то смарт-контракт, IPA или стандартный адрес EOA.

![](https://files.readme.io/cdc8c8569c1cb6106542c7f663ee06059a28105f5b32671dfa77331071f0d128-image.png)