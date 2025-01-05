---
title: Шаблон Лицензии
excerpt: >-
  Юридическая структура, написанная в виде программируемого кода, которая определяет различные условия лицензирования для интеллектуальной собственности (IP).
deprecated: false
hidden: false
metadata:
  title: ''
  description: ''
  robots: index
next:
  description: ''
---
> 🗒️ Контракт
>
> Ознакомиться со смарт контрактом [💊 Программируемой Лицензии IP (PIL)](doc:programmable-ip-license), первого и на данный момент единственного примера Шаблона лицензии, можно [тут](https://github.com/storyprotocol/protocol-core-v1/blob/main/contracts/modules/licensing/PILicenseTemplate.sol).

Шаблон лицензии — это юридическая структура, написанная в виде программируемого кода, которая определяет различные условия лицензирования для IP. Примеры таких условий:

* "Разрешено ли коммерческое использование?" - true/false (bool)
* "Можно ли передавать лицензию?" - true/false (bool)
* "Если коммерческое использование разрешено, какой процент роялти я получу?" - number

Эти условия и их значения различаются в зависимости от Шаблона лицензии.

Первый (и на данный момент единственный) пример Шаблона лицензии был разработан командой Story и называется Программируемая Лицензия интеллектуальной собственности (Programmable IP License, PIL 💊).

> 💊 Программируемая структура лицензии IP
>
> Чтобы узнать больше о первой реализации Шаблона лицензии [читайте эту страницу](doc:programmable-ip-license-pil).

## Требования к Шаблонам лицензий

Шаблоны лицензий обязаны:

* Предоставлять ссылку на фактический оффчейн-шаблон (англ. off-chain) юридического контракта со всеми параметрами, их возможными значениями и соответствующими юридическими формулировками в `licenseTextUrl`.
  * Для того чтобы структура лицензирования была совместима с протоколом Story, юридический текст **должен** быть четким и параметризованным. Каждый параметр лицензии должен устанавливать возможные варианты его значений.
  * Значения параметров в каждом Шаблоне Лицензии (так называемые "условия Шаблона Лицензии") определяют юридический текст для каждого лицензионного соглашения.
* Определять `struct` с конкретными определениями параметров в соответствии с шаблоном, который должен быть закодирован в структуре условий лицензии (описано ниже).
* Обеспечивать методы регистрации для условий лицензии и методы для их получения.
* **Проверять**, что как **минтер** (анг. minter), так и адрес, **связывающий производное произведение, имеют право совершать такие действия согласно условиям Шаблона Лицензии**.
  * Эти условия могут быть реализованы самим Шаблоном Лицензии или с использованием хуков. Они могут включать ограничения на создание производных произведений, ограничение доступа по токенам LNFT, творческий контроль от лицензиаров, KYC и т. д. Всё это зависит от реализации каждого конкретного Шаблона Лицензии.
* **Проверять совместимость условий лицензии, если у производного произведения есть или будет несколько родителей.**

## Создайте свой собственный Шаблон

Вы можете создать свой собственный Шаблон лицензии (как PIL), но он должен быть одобрен командой Story, чтобы быть полностью встроенным в протокол.