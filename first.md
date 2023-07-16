# Корень - src/index.js

```js
// Нужно сделать проверку модалное окно ложной чтобы в начале всегда был false, при появлении модальности становится true
sessionStorage.setItem("modal", false);
```

```js # Функция для сравнения два версии
function versionCompare(v1, v2) {
  //...
}
// Если возвращает 1, это означает, что версия v1 больше, чем версия v2.
// Если возвращает -1, это означает, что версия v1 менше, чем версия v2.
// Если 0 означает, что две версии одинаковы
```

```js
// Исходная ссылка - которая берет все данные(*.js, *.css) оттуда
// localStorage.getItem("sourceUrl") если существует иначе указанная ссылка
var sourceURL =
  localStorage.getItem("sourceUrl") ||
  "https://solver44.github.io/TVCOM-deploy/static/";
```

!!!
[!badge variant="dark" text="sourceURL"] - ссылка источника
!!!

```js
// Функция для подключения всех файлов js и css к index.html из исходной ссылки(sourceURL)
function scriptLoader(scripts, callback, errorCallback) {
  //...
}
```

```js
// Функция для получения версии из источника (sourceURL) для сравнения с версией текущего приложения.
function getAppVersion() {
  //...
}
```

```js # Функция для проверки система Webos(LG)
function isWebOS() {
  //...
}
```

```js
// external-scripts если true значит данные нужно брать изнутри а не из источника(sourceURL)
// с этим можно принудительно отменить загрузка файлы из источника

// load-script если true значит все нужные файлы из источника(sourceURL) уже подключен
// и все данные берется оттуда(sourceURL)
if (
  localStorage.getItem("external-scripts") === "true" ||
  sessionStorage.getItem("load-script") === "true"
)
  renderApp(); // То просто запускает приложение

else
  getAppVersion() // Иначи возмет версия из источника
    .then(({ tizen_version, webos_version }) => {
      var isWEBOS = isWebOS();
      var version = isWEBOS ? webos_version : tizen_version; // для webos и tizen отделная версия, можно сделать одна и та же версия
      version = typeof version === "string" ? version : "0.0.0";
      var appVersion = isWEBOS
        ? process.env.REACT_APP_WEBOS_VERSION   // версия webos
        : process.env.REACT_APP_TIZEN_VERSION;  // версия tizen
      if (versionCompare(version, appVersion) > 0) { // проверяет две версии
      // если версия источника больше чем версия приложения
      // то начинает загружать ресурсы из источника(sourceURL) 
        console.log("start loading from server...");
      // если успешно загружает ресурсы то оставить load-script true
        sessionStorage.setItem("load-script", true);

        scriptLoader(
          [
            sourceURL + "css/chunk.css",
            sourceURL + "css/main.chunk.css",
            sourceURL + "js/runtime-main.js",
            sourceURL + "js/chunk.js",
            sourceURL + "js/main.chunk.js",
          ],
          function () {
            // успешно загружено из источника(sourceURL)
            console.log("loaded all");
            sessionStorage.removeItem("external-scripts");
          },
          function () {
            //если ошибка то все ресурсы берет изнутри
            console.log("error");
            sessionStorage.setItem("external-scripts", "true");
            sessionStorage.removeItem("load-script");
            renderApp();
          }
        );
      } else {
        // если версия приложения болше или равен
        // то загружает файлы изнутри
        console.log(
          "the application version is the same or newer than the server version"
        );
        sessionStorage.setItem("external-scripts", "true");
        renderApp();
      }
    })
    // Если ошибка во время получение версия то запускает изнутри
    .catch(() => { 
      sessionStorage.setItem("external-scripts", "true");
      renderApp();
    });
```
