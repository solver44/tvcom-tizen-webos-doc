# /src/components/PlayerTV/index.js

[hls.js - для стрима исползуется](https://nochev.github.io/hls.js/docs/html/)

```js
// Используется для переключения звуковых дорожек
function replaceIndexInUrl(url, newIndex) {
  //...
}
```

### Параметры

- [!badge variant="dark" text="controlPlugins"] - объект необходим для того, чтобы какие элементы управления отображались или скрывались (например: следующий, предыдущий, skip и другие)
- [!badge variant="dark" text="data"] - данные от фильма или телеканала
- [!badge variant="dark" text="TV"] - если true значит это телеканал
- [!badge variant="dark" text="programList"] - все телепередачи, для телеканала
- [!badge variant="dark" text="loading"] - данные загружаются или происходит какая-то загрузка?
- [!badge variant="dark" text="setPosition"] - функция для назначения позиции видео(в секундах), только для фильмов или сериалов используется
- [!badge variant="dark" text="currentPosition"] - текущая позиция видео (в секундах)
- [!badge variant="dark" text="currentProgram"] - текущая телепередача, для телеканала
- [!badge variant="dark" text="customTracks"] - кастомный звуковые дорожки, по умолчанию автоматически будет брать
- [!badge variant="dark" text="onAutoNext"] - функция нужна для перехода в начало текущей прямой телепередачи или перехода на прямой эфир, также нужна для автоматического переключения на следующую программу
- [!badge variant="dark" text="onPrevious"] - функция вызывается при нажатии предыдущей кнопки
- [!badge variant="dark" text="onNext"] - функция вызывается при нажатии следующей кнопки
- [!badge variant="dark" text="onShowControl"] - функция вызывается при появление управление плеера
- [!badge variant="dark" text="onHideControl"] - функция вызывается при исчезнавение управление плеера
- [!badge variant="dark" text="onShowAudioModal"] - функция вызывается при появление модальное окно (от аудио дорожек, субтитры, качество видео)
- [!badge variant="dark" text="onHideAudioModal"] - функция вызывается при исчезнавение модальное окно
- [!badge variant="dark" text="hideControl"] - принудительная проверка видеоконтроля, если true то контроль пропадает

```js
// Требуется для mp4, иначе используется hls
function addSourceToVideo(element, src, type) {
  //...
}
```

```js # Функция для инициализации видео
function initHLS(uri, langOrAudioId = null) {
  try {
    destroyHls(); // Уничтожаем текущий Hls

    if (!TV && !uri.includes("m3u8")) {
      // Если не TV и не содержит "m3u8" в URI
      addSourceToVideo(player.current, uri, "video/mp4"); // Добавляем источник к текущему видео в формате mp4
    } else if (Hls.isSupported()) {
      // Если поддерживается HLS
      var tempHls = new Hls(); // Создаем новый объект Hls
      hls.current = tempHls; // Устанавливаем текущий объект Hls
      tempHls.attachMedia(player.current); // Присоединяем текущее видео к объекту Hls
      tempHls.loadSource(uri); // Загружаем источник

      if (customTracks?.length) {
        // Если есть пользовательские треки
        console.log(customTracks); // Выводим пользовательские треки в консоль
        customTracks.map((c) => {
          c.id = c.index;
          c.lang = c.display_name;
          return c;
        }); // Обновляем id и lang пользовательских треков
        var currentTrack = langOrAudioId || customTracks[0].id; // Устанавливаем текущий трек
        setAudioTracks(customTracks); // Устанавливаем пользовательские треки
        setAudioTrack(currentTrack); // Устанавливаем текущий трек
      } else {
        tempHls.on(Hls.Events.AUDIO_TRACKS_UPDATED, () => {
          var currentTrack = langOrAudioId || Math.max(tempHls.audioTrack, 0); // Устанавливаем текущий трек
          setAudioTrack(currentTrack);

          var url = tempHls?.audioTracks.find((a) => a.id == currentTrack)?.url; // Получаем URL аудио трека
          if (url) {
            tempHls.audioTrack = currentTrack; // Устанавливаем текущий аудио трек
          }
          setAudioTracks(tempHls.audioTracks); // Устанавливаем доступные аудио треки
        });
      }

      tempHls.on(Hls.Events.SUBTITLE_TRACKS_UPDATED, function (event, data) {
        setSubtitles(tempHls.subtitleTracks); // Устанавливаем доступные субтитры
      });

      tempHls.once(Hls.Events.BUFFER_APPENDED, () => {
        // setErrorCountVideo(0);
        if (!currentLoading) return;
        setCurrentLoading(false);
      });

      tempHls.on(Hls.Events.ERROR, async function (event, dataEvent) {
        if (dataEvent.fatal) {
          switch (dataEvent.type) {
            case Hls.ErrorTypes.NETWORK_ERROR:
              console.log("fatal network error encountered, try to recover"); // Встретилась фатальная сетевая ошибка, попробуйте восстановить
              tempHls.startLoad(); // Начать загрузку
              break;
            case Hls.ErrorTypes.MEDIA_ERROR:
              console.log("fatal media error encountered, try to recover"); // Встретилась фатальная ошибка медиа, попробуйте восстановить
              tempHls.recoverMediaError(); // Восстановить ошибку медиа
              break;
          }
        }
      });

      tempHls.on(Hls.Events.MANIFEST_PARSED, (event, data) => {
        if (subtitleTrack && subtitleTrack !== "off") {
          setTimeout(() => {
            if (hls.current) hls.current.subtitleTrack = subtitleTrack;
            if (tempHls) tempHls.subtitleTrack = subtitleTrack; // Устанавливаем текущи субтитр
          }, 500);
        }

        if (level) {
          if (hls.current) hls.current.currentLevel = level;
          if (tempHls) tempHls.currentLevel = level;
        }

        var lvls = tempHls.levels.filter(
          (lvl, index, self) =>
            index === self.findIndex((t) => t.name === lvl.name)
        );
        setLevels(lvls); // Устанавливаем доступные уровни (битрейты видео)

        if (!level) setLevel(tempHls.currentLevel); // Устанавливаем текущий уровень (битрейт)
      });
    } else if (
      player.current &&
      player.current.canPlayType("application/vnd.apple.mpegurl")
    ) {
      player.current.src = uri; // Устанавливаем источник для текущего видео
    }
  } catch (error) {
    console.error(error);
  } finally {
    if (player.current) player.current.load(); // Загружаем текущее видео

    return tempHls; // Возвращаем объект Hls
  }
}
```

```js
useEffect(() => {
  var uri = data.url ?? data.uri; // Получаем URI для проигрывания из данных (data), если не определено, то используем URI
  if (loading || !player.current || !uri) return; // Если происходит загрузка или проигрыватель не определен
  // или URL не определен, то прекращаем выполнение

  var tempHls = initHLS(uri); // Инициализируем HLS с указанным URL
  return () => {
    destroyHls(); // Уничтожаем объект HLS при завершении useEffect
    if (tempHls) tempHls.destroy(); // Если объект HLS существует, то также уничтожаем его
  };
}, [loading, player, data.url, data.uri]);

// Получаем информацию о предыдущей телепрограмме для текущего телеканала
var leftProgram =
  !loading &&
  TV &&
  (nullLeft ? null : (
    <ProgramInfo
      last
      programList={programList}
      id={data.program_id}
      beginTime={data.program_begin_time}
      onChange={(program) => setPrevProgram(program)}
      onError={setNullLeft}
    />
  ));

// Получаем информацию о следующей телепрограмме для текущего телеканала
var rightProgram =
  !loading &&
  TV &&
  (nullRight ? null : (
    <ProgramInfo
      reverse
      // onChange={(program) => setNextProgram(program)}
      src={data.next_program_preview_url}
      title={data.next_program_name}
      times={[data.next_program_begin_time, data.next_program_end_time]}
      beginTime={data.program_begin_time}
      onError={setNullRight}
    />
  ));

// Функция для изменения аудио-трека
function changeTrack(id) {
  try {
    setAudioTrack(id); // Устанавливаем выбранный аудио-трек
    if (!TV) {
      var uri = data.url ?? data.uri; // Получаем URI для проигрывания из данных (data), если не определено, то используем URI
      if (customTracks?.length) uri = replaceIndexInUrl(uri, id); // Если есть пользовательские треки, заменяем индекс в URI

      setTempTrackPosition(player.current?.currentTime); // Устанавливаем текущую позицию проигрывания
      var tempHls = initHLS(uri, id); // Инициализируем HLS с указанным URI и аудио-треком
    } else {
      var tempHls = hls.current; // Используем текущий объект HLS
    }
    tempHls.audioTrack = id; // Устанавливаем выбранный аудио-трек в объекте HLS
  } catch (error) {}
}

// Функция для управления субтитрами
function handleSubtitle(track) {
  if (track === "off") {
    setSubtitleTrack("off"); // Устанавливаем отключение субтитров
    hls.current.subtitleTrack = -1; // Устанавливаем отключение субтитров в объекте HLS
  }
  if (!track?.id) return;
  setSubtitleTrack(+track.id); // Устанавливаем выбранный субтитр
  hls.current.subtitleTrack = +track.id; // Устанавливаем выбранный субтитр в объекте HLS
}

// Функция для управления уровнями качества видео (битрейтами)
function handleLevel(level) {
  if (!hls.current) return; // Если объект HLS не определен, прекращаем выполнение
  hls.current.currentLevel = level?.id; // Устанавливаем выбранный уровень качества видео в объекте HLS
  setLevel(level?.id); // Устанавливаем выбранный уровень качества видео
}

// Функция для изменения соотношения сторон видео
function onChangeAspect(param) {
  setAspect(param); // Устанавливаем выбранное соотношение сторон видео
}
```
