
# Patterns

В данном документе описаны реализации некоторых паттернов во фреймворке AngularJS

## Factory method
Используется методом $provide.provider для создания экземпляра типа объектов, который конфигурируется с помощью других методов провайдера.

```
function provider(name, provider_) {
    assertNotHasOwnProperty(name, 'service');
    if (isFunction(provider_) || isArray(provider_)) {
      provider_ = providerInjector.instantiate(provider_);
    }
    if (!provider_.$get) {
      throw $injectorMinErr('pget', 'Provider \'{0}\' must define $get factory method.', name);
    }
    return (providerCache[name + providerSuffix] = provider_);
}
```
Из приведенного кода видно, что если provider_ - функция, создается экземпляр типа объектов, определенного в provider_, у которого есть метод $get, который
и будет возвращать сервис, свойства которого могут быть сконфигурированы другими методами полученного объекта. Таким образом фабричный 
метод $get вернет сконфигурированый другими методами провайдера сервис.

## Decorator

Используется методом $provide.decorator для модификации\замены функционала сервисов.
```
function decorator(serviceName, decorFn) {
    var origProvider = providerInjector.get(serviceName + providerSuffix),
        orig$get = origProvider.$get;

    origProvider.$get = function() {
      var origInstance = instanceInjector.invoke(orig$get, origProvider);
      return instanceInjector.invoke(decorFn, null, {$delegate: origInstance});
    };
}
```
Из приведенного выше кода видно, что метод decorator модифицирует метод $get сервиса, впоследствии чего фабричный метод будет возвращать
сервис, свойства и методы которого будут отличными от origInstance.

## Facade

Шаблон Facade используется в реализации сервиса $http, где за простым интерфейсом 
```
$http.get('/someUrl', config).then(successCallback, errorCallback);
```

скрывается более сложная логика работы с объектом XMLHttpRequest. Пользователю не нужно знать, каким образом происходит отправка запроса и обработка
ответа, благодаря паттерну Facade вся логика скрыта, пользователю предоставляется только то, что необходимо - обработка полученных конечных данных.

