---
title: 👥 Модуль Группировки
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
Модуль Группировки позволяет создавать и управлять групповыми IP-активами, поддерживая пул роялти для группы.

## `GroupingModule.sol`

> 🗒️ Контракт
>
> Ознакомьтесь с кодом смарт-контракта [тут](https://github.com/storyprotocol/protocol-core-v1/blob/main/contracts/modules/grouping/GroupingModule.sol).

`GroupingModule.sol` это основная точка входа для всех операций с группами. Контракт не имеет состояния (stateless) и отвечает за:

* Регистрацию новой группы
* Добавление IP-актива в группу
* Удаление IP-актива из группы
* Проверку наличия определённого IP-актива в группе
* Получение общего числа IP-активов в группе

## Создание группового IP-актива

Так же, как и для регистрации IP-актива (сначала должен быть выпущен NFT, а затем только создаётся IP-аккаунт), процесс для групповых IP-активов аналогичен. Необходимо сначала выпустить NFT стандарта ERC-721 (который представляет право собственности на группу), а затем зарегистрировать его как группу, после чего для группы создаётся IP-аккаунт.

Создать новую группу может любой пользователь.

### Реестр групповых IP-активов 

Подобно тому, как при создании IP-актива развёртывается и регистрируется IP-аккаунт через [Реестр IP-активов](doc:ip-asset-registry), IP-аккаунт группы развёртывается и регистрируется через [Реестр Групповых IP-активов](doc:group-ip-asset-registry). Этот реестр отвечает за управление регистрацией и отслеживанием групповых IP-активов, включая участников группы и пул вознаграждений.

## IP-аккаунт группы

IP-аккаунт группы должен функционировать аналогично обычному IP-аккаунту, предоставляя возможность прикрепления лицензионных условий, создания производных активов, работы с модулями и других взаимодействий.

Он имеет тот же общий интерфейс, что и обычный IP-аккаунт. Таким образом, IP-аккаунт группы может применяться в тех же сценариях, где используется обычный IP-аккаунт.

Кроме того, IP-аккаунт группы имеет функции для управления добавлением/удалением отдельных IP-активов в/из группы.

## Добавление и удаление из группы

Только владелец группы может добавлять или удалять IP-активы. При этом вам **не обязательно** владеть IP-активом, чтобы добавить его в свою группу.

### Условия добавления в группу

Есть несколько условий, которым должен соответствовать IP-актив, чтобы быть добавленным в группу:

1. Комиссия за выпуск этого IP-актива должна быть равна 0.
2. ктив должен иметь такие же лицензионные условия, как у группы, в которую он пытается войти, **или** вообще не иметь лицензионных условий. Он также может иметь дополнительные лицензионные условия, при условии, что одно из них совпадает с группой.

### Случаи, когда группы становятся заблокированными

Есть несколько ситуаций, при которых группа становится "заблокированной", то есть добавление/удаление участников становится невозможным:

* Когда группа получает какой-либо платеж.
* Если к группе привязан производный актив (т.е. группа становится "родительским" IP).

Если это произошло, для добавления/удаления участников необходимо создать новую группу.

> 📘 Пример
>
> Представьте, что у вас есть ИИ-бот, который использует данные для обучения, чтобы улучшать своё содержание. Эти данные обучения – это групповой IP-актив, являющийся корневым, а сам ИИ-бот – это производный IP-актив от данных обучения.
Каждый раз, когда ИИ-бот получает доход, средства возвращаются в данные обучения в виде роялти.
>
> Теперь вы хотите добавить больше данных обучения в группу. Однако группа уже заблокирована (поскольку к ней привязан производный актив). В таком случае вам нужно зарегистрировать новый групповой IP-актив в качестве корневого, а затем создать новый ИИ-бот как производный актив от нового корня.