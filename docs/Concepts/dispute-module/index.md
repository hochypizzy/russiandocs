---
title: ❌ Модуль Споров
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
Модуль споров даёт возможность пользователям начинать и разрешать споры через арбитраж.

## Терминология Споров

Основными компонентами арбитражной системы являются:


* **Арбитражная Политика:** Арбитражная политика относится к набору правил/процессов/субъектов, которые в совокупности будут принимать решение по спору. В настоящее время единственной поддерживаемой арбитражной политикой является [Арбитражная Политика UMA](doc:uma-arbitration-policy).
* **Арбитражный штраф:** что происходит с IP-активом после того, как он был «помечен». IPA не считается «помеченным» до тех пор, пока спор не будет признан корректным. После того, как IPA будет помечен, он не сможет:
  * создавать лицензии
  * ссылаться на родителей (анг. parents)
  * требовать роялти
  * и все его существующие лицензии становятся непригодными для использования
* **Тэги:** относится к тому, какие «метки» могут быть применены к IP-активам в протоколе. Разрешенные метки вносятся в белый список руководством протокола. Первоначальный набор меток планируется следующим образом:

<Table align={["left","left"]}>
  <thead>
    <tr>
      <th style={{ textAlign: "left" }}>
        Тэг
      </th>

      <th style={{ textAlign: "left" }}>
        Объяснение
      </th>
    </tr>
  </thead>

  <tbody>
    <tr>
      <td style={{ textAlign: "left" }}>
        `IMPROPER_REGISTRATION`
      </td>

      <td style={{ textAlign: "left" }}>
        Регистрация IP, которая уже существует
      </td>
    </tr>

    <tr>
      <td style={{ textAlign: "left" }}>
        `IMPROPER_USAGE`

        Примеры (неисчерпывающие):
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
        Относится к ненадлежащему использованию IP-актива по нескольким пунктам (примеры слева). Более подробно эти пункты можно найти в юридическом документе [💊 Программируемая IP Лицензия (PIL)](doc:programmable-ip-license).
      </td>
    </tr>

    <tr>
      <td style={{ textAlign: "left" }}>
        `IMPROPER_PAYMENT`
      </td>

      <td style={{ textAlign: "left" }}>
        Отсутствие платежей, связанных с IP
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
       Относится к «Без ненависти», «Подходит для всех возрастов», «Без наркотиков или оружия» и «Без порнографии». Более подробно эти пункты можно найти в юридическом документе [💊 Программируемая IP Лицензия (PIL)](doc:programmable-ip-license).
      </td>
    </tr>
  </tbody>
</Table>

## Процесс Рассмотрения Споров

![](https://files.readme.io/a1dc371-image.png)

### Начать Спор

Функция `raiseDispute` не требует особых разрешений и позволяет любому адресу возбудить спор против любого IP-актива, зарегистрированного в протоколе. Инициатор спора должен:

1. Выбрать «метку», по которой он поднимает спор, которая будет применена к IP-активу в случае положительного арбитражного решения. Это означает, что IP-актив официально «помечается» только тогда, когда предложенная метка подтверждается как правильная («положительное решение» на диаграмме выше).
2. Представить доказательства спора для оценки
3. Другие условия, индивидуальные для каждой арбитражной политики - например, правила оплаты и т. д.

### Вынести Решение По Спору

Функция `setDisputeJudgement` может быть вызвана только адресами из белого списка и позволяет вызывающему установить решение по спору. Может быть вызвана только один раз, так как решения по спорам неизменяемы. Если третьи лица хотят предложить обращение в суд, они могут сделать это на своей стороне и передать окончательное решение.

### Отметить Производные Если Родитель Нарушил Права

Если `setDisputeJudgement` пометила IP как нарушающую авторские права, то любой адрес может вызвать `tagDerivativeIfParentInfringed`, чтобы применить ту же метку, что и к родителю, к производным по всей цепочке производных.

> 📘 В перспективе
>
>  В будущем предполагается, что любой производный объект IP от нарушевшего правила объекта IP будет автоматически маркироваться без необходимости вызывать `tagDerivativeIfParentInfringed`. В настоящее время это ограничение, о котором мы знаем.

Тогда производные помечаются напрямую, без необходимости суждения, поскольку считается, что если родительская лицензия IP была нарушена, то все производные, которые происходят из этой лицензии, также неявно находятся в ситуации нарушения.

**Пример**: IP-актив 7 сначала помечается как нарушающий («PLAGIARISM») с помощью `setDisputeJudgement`. Только после того, как он прошёл через процесс спора, IPA 3, 1 и 0 могут быть помечены через `tagDerivativeIfParentInfringed` любым адресом без необходимости проходить через новый процесс спора.


![](https://files.readme.io/ee69754-image.png)

### Разрешение Спора

Разрешение спора удаляет метку с IP-актива. Поскольку существует два способа применения метки, есть два способа ее разрешения:

1. Метка была применена с помощью функции `setDisputeJudgement`.

В случае, если решение по спору было положительным, то есть была применена метка. После того как метка была применена к IP-активу, **инициатор спора** может, если он/она считает, что вопрос решен и метка больше не нужна, удалить ее, вызвав функцию `resolveDispute`. Например, если одна из сторон задолжала инициатору спора деньги и выплатила всю сумму после вынесения решения по спору, метка может быть удалена, и IP-актив снова будет чист.

Если инициатор спора решает не разрешать спор, то тег, определенный в `setDisputeJudgement`, остается в силе.

2. Метка была применена с помощью функции `tagDerivativeIfParentInfringed`.

Если IP была ранее помечена как нарушающая авторские права с помощью функции `tagDerivativeIfParentInfringed`, такая метка может быть удалена с помощью функции `resolveDispute` без каких-либо особых разрешений, если родитель больше не считается нарушающим права IP-активом.

Этот механизм безусловного разрешения споров существует для того, чтобы облегчить распространение по цепочке производных и удаление меток нарушения с производных IP, когда родитель разрешил свой первоначальный спор и больше не считается нарушающим права, а значит, и его производные тоже.

Если ни один из адресов не решит разрешить спор, то тег, который был применен от родителя к производному, останется в силе.

### Отмена спора

В случае, когда спор был поднят, но вопрос был решен до вынесения решения по спору, инициатор спора может отменить спор. Однако в зависимости от условий каждой арбитражной политики могут быть предусмотрены невозвратные сборы, которые не возвращаются при отмене.