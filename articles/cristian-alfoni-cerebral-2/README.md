# Cerebral 2
*Перевод статьи [Christian Alfoni](https://github.com/christianalfoni): [Cerebral 2](http://www.christianalfoni.com/articles/2017_03_19_Cerebral-2).*
*Перевод подготовлен совместно с [Алексеем Гурьяновым](https://github.com/Guria), мейнтейнером в команде Cerebral.*

Когда в прошлом году был выпущен Cerebral, это был праздник для людей, собравшихся вокруг эксперимента. Эксперимента по созданию фреймворка, который понимал бы запущенное в нём приложение. Фреймворка, который освободил бы нас от сложности создания в уме схемы того, как работает приложение, предоставив нам наглядное представление состояния, изменений состояния, побочных эффектов, рендеринга и, в конечном счете, того, как они взаимодействуют. С Cerebral 2 эксперимент созрел в фреймворк, который действительно выделяется в джунглях JavaScript.

## Статус кво
Сегодня в JavaScript-сообществе есть множество фреймворков и некоторые из них более амбициозны, чем другие. [Angular](https://angular.io/) стал платформой для приложений, [Ember](https//emberjs.com/) породил целую организацию по разработке приложений на этом фреймворке. [Vue](https://vuejs.org/) также имеет огромную пользовательскую базу. Даже у дедушки [Backbone](http://backbonejs.org/) всё ещё много пользователей. Все эти фреймворки хороши. Они постоянно внедряют инновации, поддерживают и вдохновляют свои сообщества на создание потрясающих продуктов.

Когда появился [React](https://facebook.github.io/react/), он принёс с собой большие инновации в управлении состоянием приложения. Представления-функции, принимающие состояние и возвращающие пользовательский интерфейс, — это красивая концепция, вдохновляющая на размышления о том, как мы можем лучше управлять состоянием, не затрагивая интерфейс. Наиболее заметными игроками в этой области являются такие решения, как [Mobx](https://mobx.js.org/) и [Redux](http://redux.js.org/). Сами по себе они не являются фреймворками, но в сочетании с React и другими инструментами мы можем собрать наш собственный фреймворк.

Чем должен быть обоснован наш выбор конкретного инструмента? Конечно, нет простого ответа на этот вопрос. Для некоторых разработчиков это может быть простым выбором фреймворка, основанном на использовании JSX/Hyperscript против традиционных шаблонов. Другая причина может заключаться в том, что не всех привлекает перспектива «собрать свой собственный фреймворк» — мы хотим получить все необходимое из коробки. Некоторые разработчики предпочитают писать как можно меньше кода и не видят ничего плохого в «магии». Другие хотят иметь явный код. Количество членов команды также влияет на принятие решений. В прошлом году инструменты разработчика и системы типизации также оказались важными факторами при выборе фреймворка.

## Куда вписывается Cerebral?
Я видел [фантастическую презентацию](https://youtu.be/S8HXkEnA48g?t=4h51m45s) Прети Касиредди, которая сравнивает **Mobx** с **Redux**. Причина, по которой я думаю, что презентация настолько хороша, состоит в том, что она раскрывает природу разработки приложений и сравнивает два очень разных подхода со всеми их проблемами и преимуществами. Я перескажу эту презентацию, добавив в неё Cerebral, чтобы показать, как он выглядит относительно других решений. Не переживайте, если вы не знакомы с Mobx или Redux. Вы можете продолжить чтение, поскольку примеры кода довольно просты и для вас будут иметь значение сами концепции, независимо от того, какие фреймворки вы используете. Возможно, вы даже встретите новые подходы и инструменты, которые захотите использовать в существующем фреймворке или сообществе. Заимствуйте и используйте, ведь именно этим и хорош open source :)

## Кривая обучения
Каждый фреймворк приносит новые особенности, такие как: API фреймфорка, использование новых JavaScript API и/или новых паттернов. Это, в сочетании с количеством привнесённой «магии», влияет на кривую обучения. Знакомый код, как и магический код, упрощают изучение фреймворка. Когда был выпущен Angular, он представил множество магии и многие разработчики были поражены тем, как легко они могли синхронизировать входные данные с некоторым текстом на странице. Но простой код не означает код, который легко масштабировать и поддерживать. Эти вещи часто находятся в прямом противоречии.

### Mobx
Mobx использует знакомый ООП-подход. Этот подход нам знаком из давно известных решений, таких как [Backbone](http://backbonejs.org/). Проще говоря, это означает, что мы работаем с классами. Мы создаем классы с состоянием и методами для изменения этого состояния. Это прямой способ, первым делом приходящий в голову, когда мы начинаем размышлять об архитектуре программы, но он может стать сложным, когда разные экземпляры класса начинают зависеть друг от друга и вместе должны выражать сложный поток изменений.

```javascript
// Определяем состояние и методы для его обновления
class MobxState {
  @observable items = []

  addItem (item) {
    this.items.push(item)
  }
}

// Компоненты
@observer
class Items extends Component {
  render() {
    return (
      <div>
        <ul>
          {this.props.store.items.map((item) => <li>{item}</li>)}
        </ul>
        <button onClick={() => this.props.store.addItem('foo')}>
          Add foo
        </button>
      </div>
    )
  }
}

// Передаём состояние в компоненты
const store = new MobxState();
render(<Items store={store} />, document.getElementById('mount'));
```

Mobx поистине волшебный в том, как наши компоненты обнаруживают потребность в рендере через отслеживание доступа к наблюдаемым свойствам. Как разработчик вам обычно не нужно думать о том, как это работает, и в результате Mobx имеет низкий порог входа.

## Redux
Redux использует функциональный подход. Это означает, что мы не создаем экземпляры класса, мы создаем редьюсеры. Редьюсер, в основном, работает с объектом, представляющим состояние (очень похожее на экземпляр класса), но у него нет методов. Запросы на изменение передаются в редьюсеры, и на основе типа изменения и его полезной нагрузки (данных) редьюсер обычно использует выражение `switch`, чтобы вернуть новый объект состояния. Неизменяемость (иммутабельность) данных — сильная концепция в Redux, которая определенно имеет свои преимущества, особенно в простоте оптимизации рендеринга. Но она также имеет свои недостатки, когда дело доходит до кривой обучения.

```javascript
// Определяем состояние и методы для его обновления
function ReduxState (state = Immutable.fromJS({items: []}), action) {
  switch (actions.type) {
    case 'addItem':
      return state.push('items', action.item)
  }

  return state
}

// Компоненты
const Items = connect(
  (state) => {
    return {
      items: state.items
    }
  },
  (dispatch) => {
    return {
      onClick: (item) => {
        dispatch({type: 'addItem', item})
      }
    }
  }
)(
  function ItemsComponent ({items, onClick}) {
    return (
      <div>
        <ul>
          {items.map((item) => <li>{item}</li>)}
        </ul>
        <button onClick={() => onClick('foo')}>
          Add foo
        </button>
      </div>
    )
  }
)

// Передаём состояние в компоненты
const store = createStore(ReduxState)
render(
  <Provider store={store}>
    <Items />
  </Provider>,
  document.getElementById('root')
)
```

## Cerebral
Cerebral в большей степени функциональный, чем объектно-ориентированный. Некоторые внутренние вызовы и обращения к API объектно-ориентированы, но на уровне приложения он полностью функциональный. Объектно-ориентированное программирование очень удобно для определения состояния и изменения значений состояния, оно очень выразительно и прямолинейно. Но по мере того, как мы начинаем входить в область междоменных изменений состояния и побочных эффектов, функциональный подход даёт нам возможность писать декларативный код. Декларативный код может быть прочитан фреймворком заранее, что дает инструментам разработчика представление о том, каких действий от нашего кода мы ожидаем, прежде чем он даже запустится.

```javascript
const controller = Controller({
  state: {
    items: []
  },
  signals: {
    itemAdded: push(state`items`, props`item`)
  }
})

// Компоненты
const Items = connect({
  items: state`items`,
  itemAdded: signal`itemAdded`
},
  function ItemsComponent ({items, itemAdded}) {
    return (
      <div>
        <ul>
          {items.map((item) => <li>{item}</li>)}
        </ul>
        <button onClick={() => itemAdded({item: 'foo'})}>
          Add foo
        </button>
      </div>
    )
  }
)

// Передаём состояние в компоненты
render((
  <Container controller={controller}>
    <Items />
  </Container>
), document.querySelector('#app'));
```

Cerebral не приносит магии, мы явно указываем фреймворку (и нам самим), каким образом всё связано. Но он не такой низкоуровневый, как Redux.

## Количество рутинного кода
Сколько кода мы должны написать? Важно понимать, что меньшее количество кода не означает, что код становится лучше. Мы могли бы сказать, что проверка типов является утомительной рутиной, но она даёт нам гарантии и, возможно, большую читабельность нашего кода. Чтобы избежать шаблонности, мы часто используем абстракции, но абстракции могут скрывать логику таким образом, что это мешает следующему разработчику (или вам же самим в будущем) понять, что на самом деле происходит.

### Mobx
Mobx — это то, что мы называем библиотекой с неявным подходом. Хорошим примером является рендер компонентов.

```javascript
@observer
class Items extends Component {
  render() {
    return (
      <div>
        <ul>
          {this.props.store.items.map((item) => <li>{item}</li>)}
        </ul>
        <button onClick={() => this.props.store.addItem('foo')}>
          Add foo
        </button>
      </div>
    )
  }
}
```

В этом компоненте мы не указываем, от какого состояния зависит компонент, он *автомагически* понимает это, обращаясь к наблюдаемым (*observable*) свойствам. Кода становится меньше, но при этом сложнее понять, что именно вызывает обновление этого компонента.

## Redux
Redux требует явного указания состояния, необходимого компонентам. Мы в основном создаем фабрику, которая извлекает состояние и экшены.

```javascript
const ADD_ITEM = 'ADD_ITEM'

function addItem (item) {
 return {type: ADD_ITEM, payload: item}
}

function Items ({items, onClick}) {
  return (
    <div>
      <ul>
        {items.map((item) => <li>{item}</li>)}
      </ul>
      <button onClick={() => onClick('foo')}>
        Add foo
      </button>
    </div>
  )  
}

connect(
  (state) => {
    return {
      items: state.items
    }
  },
  (dispatch) => {
    return {
      onClick: (item) => {
        dispatch(addItem(item))
      }
    }
  }
)(Items)
```

Redux требует создания намного большего количества шаблонного кода, чем Mobx. Тем не менее, такая запись более явно выражает, какое состояние и какие методы для его изменения использует этот компонент. Мы понимаем, как состояние попадает в компонент и почему оно изменяется.

## Cerebral
Cerebral также явен, как Redux, но требует меньше кода. Мы подключаем состояние и сигналы там, где они нам нужны. В Redux есть концепция «Контейнеры и Презентационные компоненты» и я лично работал над проектами, где они полностью разделены. Это означает, что вы подключаете состояние и экшены только к определенным компонентам верхнего уровня, например страницам, и в итоге получаете много пробросов свойств (*props*). Это вместо того, чтобы просто подключить состояние и экшены именно там, где они вам нужны. В Cerebral мы подключаем состояние и сигналы используя декларатичвввные теги, что улучшает читаемость.

```javascript
connect({
  items: state`items`,
  itemAdded: signal`itemAdded`
},
  function Items ({items, itemAdded}) {
    return (
      <div>
        <ul>
          {items.map((item) => <li>{item}</li>)}
        </ul>
        <button onClick={() => itemAdded({item: 'foo'})}>
          Add foo
        </button>
      </div>
    )
  }
)
```

Опять же, преимущество явного подхода в том, что мы знаем зависимости от состояния, и в случае Cerebral, какие сигналы может вызывать компонент. Поскольку это делается с помощью декларативного кода, он также может быть извлечен и отображен в инструментах разработчика, что поможет нам понять, от чего зависит компонент, даже не выполняя код рендеринга.

## Инструменты разработчика
В экосистеме React произошла революция в инструментах разработчика. Одно дело — отладчик React, но когда Redux захватил внимание со своей концепцией неизменяемых данных, это открыло новые возможности. Особенно много внимания уделялось *time-travel debugging*. «Путешествие во времени» фактически было одним из ранних экспериментов в Cerebral, почти 3 года назад, но оно оказалось трюком, не востребованным при реальной разработке. Само путешествие во времени не самая ценная часть. Другое дело история изменений состояния и то, как эти изменения были произведены, — вот это действительно оказалось востребованно. Но самую большую ценность мы видим в том, как инструменты разработчика помогают нам строить ментальный образ нашего приложения.

## Mobx
У Mobx есть инструмент, который «делает дело», как говорит Претхи. Мы можем исследовать рендеры и от каких свойств состояния зависит конкретный компонент. Этот инструмент доступен в браузере как оверлей для исследования вашего приложения.

![](https://raw.githubusercontent.com/mobxjs/mobx-react-devtools/master/devtools.gif)

## Redux
Инструменты Redux получили много любви от сообщества. Они могут использоваться как оверлей, как расширение или как автономное приложение. Существует много разных типов отладчиков и мы можем комбинировать их по нашим собственным предпочтениям.

![](https://camo.githubusercontent.com/a0d66cf145fe35cbe5fb341494b04f277d5d85dd/687474703a2f2f692e696d6775722e636f6d2f4a34476557304d2e676966)

## Cerebral
Отладчик Cerebral пошел еще дальше. Хотя инструменты Redux перечисляют мутации, мы не знаем, как они соотносятся друг с другом и как они возникли. В Cerebral мы не только получаем обзор мутаций, но также и обзор полного потока изменений в нашем приложении. Отладчик сам по себе является автономным приложением, позволяющим нам подключаться к любой среде JavaScript, будь то браузер, сервер, React Native, Electron и так далее. Мы можем даже комбинировать исполнение на клиенте и на сервере в одном потоке операций.

![](http://www.christianalfoni.com/images/debugger.gif)

# Отладка
Когда что-то пойдет не так, как мы выясним, что произошло? В зависимости от типа проблемы существуют разные подходы к отладке, но, как правило, что-то происходит при переходе из одного состояния в другое. Важное значение имеет возможность понять, что происходит на самом деле при перемещении между состояниями приложений. Это понимание можно получить путём чтения исходного кода и/или наличия инструментов разработчика, которые могут нам помочь визуализировать это.

## Mobx
С магической природой Mobx может быть трудно отследить ошибки. Поскольку переходы состояний происходят «за кулисами», может быть трудно понять, что именно происходит при чтении кода. Также тот факт, что изменения могут произойти где угодно, не облегчает ситуацию. Тем не менее, Mobx может быть принужден к одностороннему потоку данных, а инструменты разработчика помогают нам понять, как компонент рендерится.

## Redux
В Redux все связано довольно явно и потому его легче отладить, исследуя код вокруг проблемы. Очень помогает тот факт, что изменение состояния может происходить только внутри редьюсеров. Возможность изучения журнала изменений в инструментах разработчика также является большим преимуществом.

## Cerebral
В Cerebral явно обозначаются зависимости состояния. Ограничены места, где могут происходить изменения и используется однонаправленный поток данных. Как следствие, Cerebral имеет те же преимущества, что и Redux. Кроме того, у нас есть отладчик, дающий нам ментальный образ того, как происходят изменения состояния с возможностью отфильтровывать определенные изменения состояния. Это делает отладку логики приложения приятным и эффективным занятием. Действительно сильной стороной Cerebral является тот факт, что мы можем видеть больше, чем отдельные переходы состояния. Мы можем увидеть поток смены состояний и побочные эффекты, связанные с конкретным событием в приложении.

# Предсказуемость
Данный пункт напрямую связан с предыдущим. Ведет ли наш код себя так, как мы ожидаем? Одним из примеров во введении во Flux был счетчик на кнопке уведомлений Facebook. Возникла проблема, в том, что он появлялся, когда это не предполагалось. Было предпринято множество итераций, пытающихся исправить ошибку, но она возвращалась. Вот почему был представлен Flux. Это дало предсказуемый способ обновления состояния с использованием концепции одностороннего потока данных. Все запросы на изменения нацелены на «верхнюю часть приложения», которая изменяет состояние, а затем визуализирует компоненты. Пользовательский интерфейс является прямым результатом текущего состояния приложения.

## Mobx
Если мы не навязываем концепцию одностороннего потока данных при использовании Mobx у нас появляются проблемы с предсказуемостью, так как изменение состояния может произойти где угодно.

## Redux
Redux с его явным определением того, чем является изменение состояния и как оно может быть выполнено, является «королем предсказуемости». Он основывается на идеях Flux и стал его реализацией.

## Cerebral
Cerebral также построен на концепциях Flux. Компоненты только вызывают сигналы, которые говорят: «Это случилось». Затем Cerebral запускает мутации и побочные эффекты так, как определено разработчиком. Поскольку мы определяем весь поток в одном сигнале, Cerebral очень предсказуем. Изменения не делятся на отдельные части нашего кода. Все происходит в одном месте, собрано вместе, и эта композиция также отображается в отладчике.

# Тестируемость
Некоторые разработчики очень агрессивны в тестировании, преследуя цель 100% покрытия. В других случаях тестирование отнимает слишком много времени из-за постоянных изменений в самом приложении, характерных для стартапов. Мы можем тестировать компоненты, мы можем протестировать изменение состояния, поток с побочными эффектами или просто функцию, вычисляющую некоторые данные. Независимо от того, что мы тестируем, важно, чтобы тесты оставались независимыми и не вызывали неожиданных побочных эффектов.

## Mobx
Если мы позволяем Mobx вносить изменения везде, а экземпляры класса передаются в другие экземпляры класса, код становится труднее тестировать. Тем не менее, думая о тестируемости в процессе разработки, вполне возможно сделать Mobx тестируемым с использованием экшенов и планированием наших доменов как можно более изолированными.

##Redux
В Redux в основном используются чистые функции. Фабрики экшенов просто принимают на вход параметры и возвращают объект. Редьюсеры работают также, но только с состоянием. Это делает отдельные части Redux хорошо тестируемыми. Тем не менее, когда мы имеем дело с побочными эффектами, мы больше не находимся в чистом функциональном мире и все становится труднее тестировать.

## Cerebral
Тестирование отдельных изменений состояния — это всего лишь небольшая часть истории. В действительности тесты должны проверять потоки изменений в нашем коде в так называемых интеграционных тестах. Когда пользователь щелкает на кнопку выполняются ajax-запросы, вносятся изменения состояния и так далее. Так какое состояние мы получим в конце? Cerebral отделяет побочные эффекты от процесса выполнения и даже изменения состояния. Функции в сигналах получают на вход один аргумент, называемый контекстом, и этот контекст легко может быть подменён во время тестирования. Cerebral предоставляет набор вспомогательных инструментов для снижения шаблонного кода при написании тестов.

# Модульность
Как разработчики мы предпочитаем изолированные куски кода, модули. Однако проблема заключается в том, что (особенно в коде фронтэнд приложения) эти модули должны часто общаться с другими. Им нужно получать доступ к состоянию друг друга и инициировать логику «внутри» друг друга. Планирование того, как эти отношения должны работать и избегание циклических зависимостей может быть проблематичным. Использование событийной системы может также уменьшить читаемость того, как вещи соотносятся друг с другом.

## Mobx
Mobx использует классы: состояние и методы для изменения этого состояния и выполнения побочных эффектов содержатся внутри класса. В этом смысле Mobx имеет действительно хорошую модульность. Однако у нас возникают проблемы, когда эти экземпляры классов должны обращаться к состоянию друг друга или запускать методы друг друга. Это бывает довольно трудно скоординировать.

## Redux
Redux на самом деле не имеет концепции модульности. Мы определяем наши действия где-то здесь и наши редьюсеры где-то там. Самое замечательное в этом: любой редьюсер может реагировать на любое действие. Но у нас возникают проблемы, когда редьюсеру нужны данные состояния соседнего редьюсера.

## Cerebral
Cerebral имеет понятие, называемое модулями. Это способ для нас структурировать наше приложение, но без полного изолирования сигналов и состояния: любой сигнал может получить и изменить любое состояние. Также в одном сигнале можно составлять логику других сигналов. Это означает, что Cerebral обладает высокой степенью компонуемости и модульности, без изоляции. Трудно ошибиться при планировании доменов, и в принципе нет риска, связанного с проблемами циклической зависимости. Нет необходимости передавать экземпляры классов для получения доступа к тому, что нам нужно.

# Масштабируемость/поддержка
Написание 500-й и 10000-й строк кода весьма отличается. Когда приложение растет, становится все более важным сохранить вещи простыми для понимания, а не лёгкими в использовании. Быть простым для понимания — значит иметь чёткие определения и зоны ответственности. Эта часть кода обрабатывает визуализацию пользовательского интерфейса, эта часть обрабатывает запрос на изменение состояния, а эта часть изменяет состояние. Заманчиво сделать всё внутри одного компонента. Но когда все 50 компонентов имеют свое внутреннее состояние, побочные эффекты и изменения состояния, самостоятельно пытаются «достичь друг друга», становится сложно понимать происходящее. Когда приложения растут, нам нужно иметь хорошо разделенные понятия и ответственности.

## Mobx
С Mobx очень легко начать выполнять задачу. Но он не заставляет нас строго определять, где запрашивать изменения состояния, где делать изменения состояния и побочные эффекты. Это может произойти где угодно. Это приводит к тому, что Mobx без хорошей дисциплины становится менее идеальным для масштабирования и обслуживания.

## Redux
Redux даёт чёткие инструкции: для чего предназначены компоненты, всегда необходимо инициировать экшены для запроса изменений состояния, а редьюсеры — это то, где происходят изменения состояния. Это делает Redux масштабируемым и легко обслуживаемым. Тем не менее, нет четкого мнения о том, как справиться с побочными эффектами, хотя сообщество и предлагает некоторые инструменты, которые могут нам в этом помочь.

## Cerebral
Cerebral имеет четкое определение о том, что и где происходит. Наши компоненты в идеале не должны самостоятельно управлять состоянием. Всегда есть исключения для сложных обновлений пользовательского интерфейса, что имеет место для любого фреймворка, но, в остальном, всё состояние переходит в Cerebral. Состояние можно изменить только вызвав сигнал. Сигналы имеют логику для запуска побочных эффектов, составляя их все вместе в едином потоке. Масштабирование приложения на Cerebral заключается в добавлении нового состояния и сигналов или новых модулей для целей упорядочивания. Очень легко вводить в курс дела новых членов команды, поскольку они быстро получают ментальный образ того, как приложение работает, щелкая мышью в пользовательском интерфейсе и наблюдая за тем, как откликается на эти действия отладчик.

# Итог
Когда я смотрел презентацию Претис, мне показалось, что Cerebral — это баланс между Mobx и Redux. Он дает нам предсказуемость, ясность и отличный инструментированный опыт Redux, но с меньшим количеством шаблонного кода и более низкой кривой обучения. Мы не утверждаем, что Cerebral — идеальное решение и вам не нужны Mobx или Redux. Это просто альтернатива, которая может вам подойти лучше, если вы считаете, что Mobx слишком магичен и «радикален», а Redux слишком многого стоит и «консервативен».

Спасибо за чтение! Узнать больше о Cerebral можно на [официальном сайте](http://www.cerebraljs.com/).

---

*Слушайте наш подкаст в [iTunes](https://itunes.apple.com/ru/podcast/девшахта/id1226773343) и [SoundCloud](https://soundcloud.com/devschacht), читайте нас на [Medium](https://medium.com/devschacht), контрибьютьте на [GitHub](https://github.com/devSchacht), общайтесь в [группе Telegram](https://t.me/devSchacht), следите в [Twitter](https://twitter.com/DevSchacht) и [канале Telegram](https://t.me/devSchachtChannel), рекомендуйте в [VK](https://vk.com/devschacht) и [Facebook](https://www.facebook.com/devSchacht).*

[Статья на Medium](https://medium.com/devschacht/cristian-alfoni-cerebral-2-275c338f62d4)