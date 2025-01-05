---
title: Конфигурация/Хук Лицензии
excerpt: >-
  Дополнительный хук, который можно прикрепить к IP-активу или конкретной лицензии для динамического определения комиссий за выпуск.
deprecated: false
hidden: false
metadata:
  title: ""
  description: ""
  robots: index
next:
  description: ""
---

## Конфигурация лицензии

> 🗒️ Контракт
>
> Ознакомьтесь со смарт-контрактом [тут](https://github.com/storyprotocol/protocol-core-v1/blob/main/contracts/lib/Licensing.sol).


Вы можете опционально прикрепить `LicensingConfig` к IP-активу (для определённых `licenseTermsId`, связанных с этим активом), который содержит такие параметры, как `mintingFee` и `licensingHook`, как показано ниже.

```sol Licensing.sol
/// @notice Эта структура используется владельцами IP для настройки конфигурации
/// при выпуске токенов лицензий для их IP через LicensingModule.
/// Когда вызывается функция `mintLicenseTokens` модуля LicensingModule, он считывает
/// эту конфигурацию для определения комиссии за выпуск и выполняет лицензионный хук, если он установлен.
/// Владельцы IP могут устанавливать эти конфигурации для каждой лицензии
/// или на уровне всей IP, чтобы конфигурация применялась ко всем лицензиям актива.
/// Если конфигурация установлена и на уровне лицензии, и на уровне IP, конфигурация лицензии имеет приоритет.
struct LicensingConfig {
  bool isSet;  // Установлена ли конфигурация
  uint256 mintingFee;  // Комиссия за выпуск токена
  address licensingHook;  // Адрес контракта хука, если он установлен, иначе address(0)
  bytes hookData;  // Данные, используемые хуком
  uint32 commercialRevShare;  // Доля коммерческих доходов в процентах
  bool disabled;  // Отключена ли лицензия
  uint32 expectMinimumGroupRewardShare;  // Минимальная доля вознаграждения группы
  address expectGroupRewardPool;  // Ожидаемый пул вознаграждений группы
}
```
Параметры, такие как `mintingFee` и `commercialRevShare`, могут переписывать свои дубликаты, указанные в условиях лицензии. Это позволяет производным IP-активам (которые обычно не могут изменить условия лицензии) изменить определённые поля.

Параметр `licensingHook` — это адрес смарт-контракта, реализующего интерфейс `ILicensingHook.sol`. В нём содержится функция `beforeMintLicenseTokens`, которая запускается перед выпуском токена лицензии, что позволяет вставить дополнительную логику.

Сам хук определен ниже в другом разделе. Он содержит информацию о лицензии, о том, кто выпускает токен лицензии и кто его получает.

### Настройка конфигурации лицензии

Вы можете настроить конфигурацию лицензии с помощью функции `setLicenseConfig` в контракте [LicensingModule.sol](https://github.com/storyprotocol/protocol-core-v1/blob/main/contracts/modules/licensing/LicensingModule.sol).

Вы можете прикрепить конфигурацию лицензии ко всему IP-активу, чтобы она применялась ко всем условиям лицензий, принадлежащих этой IP. Если установлены обе конфигурации (на уровне IP и на уровне лицензии), приоритет отдаётся конфигурации лицензии. Вы можете установить конфигурацию на IP в целом, передав `licenseTemplate == address(0)` и `licenseTermsId == 0` в функцию `setLicenseConfig`.



### Примеры логики с конфигурацией лицензии

1. **Ограничение количества лицензий**: Через `licensingHook` можно ограничить максимальное количество лицензий. Например, транзакция будет отменена, если достигнуто максимальное число лицензий.
2. **Запрет на производные**: Если вы зарегистрируете производный IP-актив, вы не сможете изменить его лицензионные условия, как описано [здесь] (https://docs.story.foundation/docs/license-terms#inherited-license-terms). Вы можете задаться вопросом: «А что, если я, как производный IP-актив, хочу запретить производные от себя, но мои Лицензионные условия разрешают производные, могу ли я это изменить?». Да, вы можете просто установить `disabled` в true.
3. **Комиссия за выпуск**: Аналагочино #2 выше... а что насчет комиссии? Хотя вы не можете изменить условия лицензии на родительском IP-активе (и, соответственно, плату за выпуск внутри него), вы можете изменить плату за выпуск для производного актива, изменив `mintingFee` в конфигурации лицензии или вернув `totalMintingFee` из `licensingHook` (описано в следующем разделе).
4. **Коммерческая доля дохода**: Аналогично #2 и #3 вы можете изменить `commercialRevShare` в конфигурации лицензии.
5. **Динамическое ценообразование**: Установите динамическую стоимость выпуска лицензии на основе количества уже выпущенных токенов, количества лицензий, которые запрашивает пользователь, или даже личности пользователя. Всё это доступно в `licensingHook` (далее).

... и так далее.

### Ограничения

После обновления конфигурации лицензии нельзя уменьшать процент `commercialRevShare`. Его можно только увеличивать.

## Хук Лицензирования

> 🗒️ Контракт
>
> Ознакомьтесь со смарт-контрактом [тут](https://github.com/storyprotocol/protocol-core-v1/blob/main/contracts/interfaces/modules/licensing/ILicensingHook.sol#L26).

Функция `beforeMintLicenseTokens`, которая выступает в качестве хука, вызывается перед выпуском токена лицензии для выполнения пользовательской логики и определения окончательной суммы комиссии `totalMintingFee` за выпуск токена. Владелец IP-актива должен настроить конфигурацию лицензии (в которой содержится хук) с собственной реализацией функции `beforeMintLicenseTokens`, чтобы она могла быть вызвана.

Хук также может быть использован для выполнения различных проверок и логики, как [описано выше](https://docs.story.foundation/docs/license-config-hook#logic-that-is-possible-with-license-config).

> 🚧 Внимание!
>
> Будьте осторожны с потенциально вредоносными реализациями внешних хуков лицензий. Сначала проверьте код выбранного хука, так как он может не быть проверен или аудирован командой Story.

```sol ILicensingHook.sol
/// @notice Функция вызывается LicensingModule при выпуске токенов лицензии.
/// @dev Хук может быть использован для различных проверок и определения стоимости выпуска.
/// @param caller Адрес вызывающего, который вызывает функцию mintLicenseTokens().
/// @param licensorIpId ID IP-актива лицензиара, из которого выпускаются токены.
/// @param licenseTemplate Адрес шаблона лицензии.
/// @param licenseTermsId ID условий лицензии в шаблоне.
/// @param amount Количество выпускаемых токенов.
/// @param receiver Адрес получателя токенов.
/// @param hookData Данные, используемые хуком.
/// @return totalMintingFee Итоговая сумма комиссии за выпуск.
function beforeMintLicenseTokens(
  address caller,
  address licensorIpId,
  address licenseTemplate,
  uint256 licenseTermsId,
  uint256 amount,
  address receiver,
  bytes calldata hookData
) external returns (uint256 totalMintingFee);
```

Обратите внимание, что функция возвращает `totalMintingFee`. Вы можете задаться вопросом: «Я могу установить комиссию за выпуск в условиях лицензии, в конфигурации лицензии и вернуть динамическую стоимость из `beforeMintLicenseTokens`. Какая комиссия будет в итоге?»
<Table align={["left","left"]}>
  <thead>
    <tr>
      <th style={{ textAlign: "left" }}>
        Комиссия
      </th>

      <th style={{ textAlign: "left" }}>
        Приоритет
      </th>
    </tr>

  </thead>

  <tbody>
    <tr>
      <td style={{ textAlign: "left" }}>
        `totalMintingFee` из `beforeMintLicenseTokens`
      </td>

      <td style={{ textAlign: "left" }}>
        Высочайший
      </td>
    </tr>

    <tr>
      <td style={{ textAlign: "left" }}>
       `mintingFee` из конфигурации лицензии
      </td>

      <td style={{ textAlign: "left" }}>
        :arrow_down:
      </td>
    </tr>

    <tr>
      <td style={{ textAlign: "left" }}>
        `mintingFee` из условий лицензии
      </td>

      <td style={{ textAlign: "left" }}>
        Низкий
      </td>
    </tr>

  </tbody>
</Table>