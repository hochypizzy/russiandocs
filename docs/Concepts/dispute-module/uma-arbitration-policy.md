---
title: Арбитражная политика UMA
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
> 🚧 Внимание: Только в версии 1.3
>
> Арбитражная политика UMA доступна только в версии 1.3 нашего протокола, которая пока не задокументирована.

> 📘 UMA
>
> Для подробной информации о том, как работает механизм разрешения споров UMA, [посетите их сайт](https://uma.xyz/).

Эта арбитражная политика представляет собой механизм разрешения споров, который следует правилам [UMA](https://uma.xyz/). Ниже мы приводим краткий обзор того, как работает процесс разрешения споров UMA.
## Схема Работы Смарт-контракта

![](https://files.readme.io/e0dfb0a226bdd29ab3adede7d1df7d6662497331e1b92319ee1ad8344dc5dfa3-image.png)

1. Возбуждение спора - Первый шаг для инициирования спора против IP-актива — это вызов функции `raiseDispute` в контракте [DisputeModule.sol](https://github.com/storyprotocol/protocol-core-v1/blob/main/contracts/modules/dispute/DisputeModule.sol). Эта функция вызывает метод `assertTruth` в контракте UMA `OptimisticOracleV3.sol`. Чтобы инициировать спор, инициатору необходимо внести залог, который должен быть не меньше минимальной суммы, определённой UMA для выбранной валюты.
2. (Опционально) Контрспор / Подтверждение спора - После вызова `raiseDispute` начинается период активности (анг. liveness), в течение которого можно подать контрспор. Период активности делится на две части:
(i) первая часть, в течение которой только владелец IP может подать контрспор;
(ii) вторая часть, в течение которой любой адрес может подать контрспор, вызвав функцию `disputeAssertion` в контракте `ArbitrationPolicyUMA.sol`.
Чтобы подать контрспор, необходимо внести залог в той же валюте и размере, что и инициатор спора.
3. Урегулирование заявления
  1. Если никто не подал контрспор, то после окончания периода активности любой адрес может вызвать функцию `settleAssertion` в контракте UMA `OptimisticOracleV3.sol`.
  2. Если контрспор был подан до окончания периода активности, спор передаётся на рассмотрение арбитров UMA, которые примут решение о том, нарушает ли IP права или нет. После вынесения решения любой адрес может вызвать функцию `settleAssertion` в контракте UMA `OptimisticOracleV3.sol`.

## Руководство по подаче доказательств спора

При возбуждении спора или подаче контрспора обе стороны могут предоставить доказательства спора. Доказательства спора — это текстовый документ, который будет использован UMA для принятия решения по спору.

### Требования к документу

Каждый документ должен соответствовать следующим требованиям:

* Это должен быть текстовый документ. При необходимости он может содержать изображения или видео.
* Документ должен быть загружен в IPFS.
* Его рассмотрение не должно занимать у арбитра больше двух часов. Время арбитра ограничено, и доказательства могут быть признаны недействительными, если их рассмотрение займёт слишком много времени. Старайтесь кратко излагать доказательства, чтобы они были действительными.

В зависимости от тега спора (анг. Dispute Tag) необходимо предоставить дополнительные доказательства:

<Table align={["left","left","left"]}>
  <thead>
    <tr>
      <th style={{ textAlign: "left" }}>
        Тег спора
      </th>

      <th style={{ textAlign: "left" }}>
        Содержание доказательств
      </th>

      <th style={{ textAlign: "left" }}>

        Процесс рассмотрения доказательств

      </th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td style={{ textAlign: "left" }}>
        `IMPROPER_REGISTRATION`
      </td>

      <td style={{ textAlign: "left" }}>

        Входные данные:
        A. Пример или ссылка на ранее существующую IP, которая нарушена спорной IP.
        B. Доказательства, что ранее существующая IP зарегистрирована раньше, чем спорная IP.

      </td>

      <td style={{ textAlign: "left" }}>
        1. Проверьте, является ли ранее существующая IP идентичной или очень похожей на спорную IP, используя данные A.
           * Микки Маус с разницей в 1 пиксель — это нарушение.
           * Микки Маус в новой шляпе — это нарушение, если это не производная IP от Микки Мауса.
        2. Проверьте дату регистрации ранее существующей IP, используя данные B.
        3. Подтвердите, что спорная IP зарегистрирована позже, проверив данные в Хабе.
        4. Подтвердите, что спорная IP не является производным от ранее существующей IP, проверив данные в Хабе. 

        <br />
      </td>
    </tr>

    <tr>
      <td style={{ textAlign: "left" }}>
        `IMPROPER_USAGE`

        Примеры (неисчерпывающие)\
        Территория,
        Каналы распространения,
        Срок действия,
        Безвозвратность,
        Атрибуция,
        Производные,
        Ограничения на создание производных,
        Коммерческое использование,  
        Сублицензируемая,
        Непередаваемые,
        Ограничение на кросс-платформенное использование
      </td>

      <td style={{ textAlign: "left" }}>

        Входные данные:\
        A. Текст: Указание нарушенного пункта лицензии PIL.
        B. Текст: Описание нарушения.
        C. Текст: Доказательства нарушения.

      </td>

      <td style={{ textAlign: "left" }}>
        1. Прочитайте описание нарушенного пункта в официальном документе лицензии PIL, используя данные A.
        2. Ознакомьтесь с описанием нарушения, используя данные B.
        3. Проверьте достоверность представленных доказательств, используя данные C.
      </td>
    </tr>

    <tr>
      <td style={{ textAlign: "left" }}>
        `IMPROPER_PAYMENT`
      </td>

      <td style={{ textAlign: "left" }}>

        Входные данные:\
        A. Текст: Описание каждого платежа, который должен был быть передан предшествующим IP (анг. ancestors), но не был.
        B. Текст: Доказательства по платежам.

      </td>

      <td style={{ textAlign: "left" }}>
        1. Проверьте достоверность доказательств платежей, используя данные A и B.
        2.  Если доказательства платежей реальны, подтвердите, что эти платежи действительно не были выполнены на блокчейне, используя блокчейн-обозреватель (анг. blockchain explorer). 

        <br />
      </td>
    </tr>

    <tr>
      <td style={{ textAlign: "left" }}>
        `CONTENT_STANDARDS_VIOLATION`

        Без ненависти\
        Подходит для всех возрастов
        Без наркотиков и оружия
        Без порнографии
      </td>

      <td style={{ textAlign: "left" }}>
        Входные данные:\
        A. Текст: Нарушенный пункт стандартов содержания.
        B. Текст: Описание нарушения.
        C. Текст: Доказательства нарушения.

      </td>

      <td style={{ textAlign: "left" }}>
        1. Прочитайте описание нарушенного пункта в официальном документе PIL, используйте данные A.
        2. Ознакомьтесь с описанием нарушения, используя данные B.
        3. Проверьте достоверность представленных доказательств, используя данные C. 

        <br />
      </td>
    </tr>
  </tbody>
</Table>

> 📘 Примечание
>
> Процесс пока экспериментальный, и формат/содержание доказательств может изменяться и дорабатываться.