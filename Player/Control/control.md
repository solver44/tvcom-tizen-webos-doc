# /src/components/PlayerTV/Control.js
### Компонент для отображения элементов управления плеера

```js
var SKIP_TIME = 15; // Время пропуска вперед/назад (в секундах)
var HEIGHT_ICON = is4k === 2 ? 28 : is4k === 1 ? 68 : 34; // Высота иконки (в пикселях), в зависимости от типа видео (4K, HD или SD)
var CONTROLS_HIDE_TIMEOUT = 3000; // Время появления элементов управления (в миллисекундах)

const aspects = [
  { text: "Auto", default: true }, // Автоматический режим соотношения сторон (по умолчанию)
  { text: "FitToScreen" }, // Подгон под экран
  { text: "FillToWidth" }, // Заполнение по ширине
  { text: "ZoomToScreen" }, // Увеличение до размера экрана
  { text: "ZoomOffScreen" }, // Увеличение за пределы экрана
  { text: "16:9" }, // Соотношение сторон 16:9
  { text: "21:9" }, // Соотношение сторон 21:9
  { text: "18:9" }, // Соотношение сторон 18:9
  { text: "shrink" }, // Уменьшение размера видео
];
```

### Параметры

- [!badge variant="dark" text="loadingState"] - состояние загрузки плеера.
- [!badge variant="dark" text="startTime"] - время начала контента.
- [!badge variant="dark" text="idChannel"] - идентификатор телеканала.
- [!badge variant="dark" text="endTime"] - время окончания контента.
- [!badge variant="dark" text="player"] - объект проигрывателя.
- [!badge variant="dark" text="plugins"] - объект, определяющий, какие элементы управления отображаются или скрываются (например: следующий, предыдущий, skip и другие).
- [!badge variant="dark" text="TV"] - флаг, указывающий, является ли проигрывание на телевизоре.
- [!badge variant="dark" text="isLive"] - флаг, указывающий, является ли контент прямой трансляцией.
- [!badge variant="dark" text="setPosition"] - функция для установки текущей позиции проигрывания.
- [!badge variant="dark" text="currentPosition"] - текущая позиция проигрывания.
- [!badge variant="dark" text="isCurrentProgram"] - флаг, указывающий, является ли текущая программа активной.
- [!badge variant="dark" text="children"] - дочерние элементы.
- [!badge variant="dark" text="right"] - компонент следующей телепрограмме.
- [!badge variant="dark" text="left"] - компонент предыдущей телепрограмме.
- [!badge variant="dark" text="aspect"] - выбранное соотношение сторон видео.
- [!badge variant="dark" text="onChangeAspect"] - функция для изменения выбранного соотношения сторон.
- [!badge variant="dark" text="track"] - выбранный аудио-трек.
- [!badge variant="dark" text="tracks"] - список доступных аудио-треков.
- [!badge variant="dark" text="onChangeAudioTrack"] - функция для изменения выбранного аудио-трека.
- [!badge variant="dark" text="subtitleTrack"] - выбранный субтитр.
- [!badge variant="dark" text="subtitles"] - список доступных субтитров.
- [!badge variant="dark" text="onChangeSubtitle"] - функция для изменения выбранного субтитра.
- [!badge variant="dark" text="level"] - выбранный уровень качества видео (битрейт).
- [!badge variant="dark" text="levels"]- список доступных уровней качества видео.
- [!badge variant="dark" text="onChangeLevel"] - функция для изменения выбранного уровня качества видео.
- [!badge variant="dark" text="onPrevious"] - обработчик события "предыдущий".
- [!badge variant="dark" text="onNext"]` - обработчик события "следующий".
- [!badge variant="dark" text="onAutoNext"]` - обработчик события "автоматический переход к следующему".
- [!badge variant="dark" text="onClickLive"]` - обработчик события "переход в режим прямого эфира".
- [!badge variant="dark" text="onClickRestart"]` - обработчик события "перезапуск контента".
- [!badge variant="dark" text="onShowModal"]` - обработчик события "открытие модального окна".
- [!badge variant="dark" text="onHideModal"]` - обработчик события "закрытие модального окна".
- [!badge variant="dark" text="tempTrackPosition"]: tempTrackPosition - временная позиция аудио-трека.
- [!badge variant="dark" text="onShow"]: onShow - обработчик события "показ элементов управления".
- [!badge variant="dark" text="onHide"]: onHide - обработчик события "скрытие элементов управления".
- [!badge variant="dark" text="hideControl"]: hideControl - флаг, указывающий, следует ли скрыть элементы управления.


==- Часть 1
```js
// Обновление значения modifiedPlugins путем объединения объектов Plugins и plugins
var modifiedPlugins = { ...Plugins, ...plugins };

// Состояние для управления позицией телевизора
var [TVPos, setTVPos] = useState();

// Состояние для управления модальным режимом (для субтитров, аудио дорожек и для битрейтов)
var [modalMode, setModalMode] = useState(false);

// Создание референса для записи значения record
var record = useRef(false);

// Состояние для управления проигрыванием видео
var [isPlay, setIsPlay] = useState(true);

// Состояние для управления пропуском контента
var [isSkip, setIsSkip] = useDebounce({ show: false, back: false }, 600);

// Состояние для управления отображением элементов управления
var [showControl, setShControl] = useState({
  show: TVPos && !loadingState ? true : false,
  clear: false, // clear - при первом открытии телеканал появляется без элементов управления, будет только информация
  quick: false, // quick - если true появляется только слайдер(ползунок видео) при нажатия вперед назад 
});

// Создание референса для showControl
var showControlEl = useRef(showControl);

// Создание референса для timeout
var timeout = useRef(null);

// Состояние для хранения длительности видео в секундах
var [durationOfSeconds, setDurationOfSeconds] = useState(0);

// Создание референсов для элементов управления
var durationOfSecondsEl = useRef();
var sliderThumb = useRef();
var progressBar = useRef();
var seekToolTip = useRef();
var currentDurationEl = useRef();
var videoDurationEl = useRef();
var isSkipEl = useRef({});
var rangeVideo = useRef();
var loading = useRef(loadingState);
var startTimeEl = useRef(startTime);
var playerRef = useRef(player);
var isPlayRef = useRef(isPlay);

// Обновление значения isPlayRef.current при изменении состояния isPlay
useEffect(() => {
  isPlayRef.current = isPlay;
}, [isPlay]);

// Установка позиции проигрывания при изменении временной позиции
useEffect(() => {
  if (!tempTrackPosition) return;
  seekTo(tempTrackPosition);
}, [tempTrackPosition]);

// Фокусировка на ползунке видео при показе элементов управления
useEffect(() => {
  if (!rangeVideo.current) return;
  if (showControl.show && !showControl.clear) rangeVideo.current.focus();
}, [showControl.show, showControl.clear]);

// Установка progressBar.current равным rangeVideo.current
useEffect(() => {
  progressBar.current = rangeVideo.current;
}, [showControl]);

// Обновление playerRef.current при изменении значения player
useEffect(() => {
  playerRef.current = player;

  window.removeEventListener("keydown", keydownHandle);
  // Добавление события keydown для обработки нажатий клавиш
  window.addEventListener("keydown", keydownHandle);
  return function () {
    window.removeEventListener("keydown", keydownHandle);
  };
}, [player, isSkip]);

// Обновление startTimeEl.current при изменении значения startTime
useEffect(() => {
  startTimeEl.current = startTime;
  if (!startTime || !player) return;
  // Сброс слайдера (ползунок видео)
  resetSlider();
}, [startTime]);

// Установка durationOfSecondsEl.current равным 0 при изменении значения isLive
useEffect(() => {
  durationOfSecondsEl.current = 0;
  if (player) resetSlider();
  setTimeout(() => {
    if (!TVPos) return;
    setTVPos(null);
  }, 500);
}, [isLive]);

useEffect(() => {
  if (!hideControl) return;
  // Установка значения showControl.show в false
  setShowControl({ ...showControl, show: false });
}, [hideControl]);

// Обновление loading.current при изменении значения loadingState
useEffect(() => {
  loading.current = loadingState;
  if (TVPos) {
    // Показ элементов управления
    setShControl({ show: true, clear: false, quick: false });
    showControlEl.current = { show: true, clear: false, quick: false };
  } else if (!loadingState) {
    // Перезагрузка таймаута для скрытия элементов управления
    reloadTimeout(TV ? true : false);
  }
  if (loadingState) return;
  durationOfSecondsEl.current = 0;
  if (!isPlay) setIsPlay(true);
  if (TV) return;
  setTimeout(() => {
    console.log("record position started");
    if (record.current) return;
    record.current = true;
  }, 10000);
}, [loadingState]);

// Обновление значения TVPos при изменении значения idChannel
useEffect(() => {
  if (!TVPos) reloadTimeout(TV ? true : false);
}, [idChannel]);

// Обновление слайдера и запуск обновления позиции
useEffect(() => {
  if (!player || loading.current) return;
  if (player) resetSlider();
  // Запуск обновления позиции
  return startUpdate();
}, [player, loadingState, durationOfSeconds, isLive]);

// Обновление видимой продолжительности при изменении значения durationOfSeconds
useEffect(() => {
  if (!endTime && videoDurationEl.current)
    videoDurationEl.current.innerText = toHHMMSS(durationOfSeconds);
}, [durationOfSeconds, player]);

// Обновление класса для элементов управления в зависимости от значения showControl.quick
useEffect(() => {
  if (!playerControls.current) return;
  if (showControl.quick) playerControls.current.classList.add(classes.no_bg);
  else playerControls.current.classList.remove(classes.no_bg);
}, [showControl.quick]);

// Обработка нажатий клавиш
function keydownHandle(e) {
  // Обработка событий нажатия клавиш

  // ...

  // Обработка событий нажатия клавиш на progressBar и seekToolTip

  // ...
}

// Перезагрузка таймаута для скрытия элементов управления
function reloadTimeout(clear = false, quick = false) {
  if (loading.current) return;
  // Обновление progressBar
  // ...
  // Остановка предыдущего таймаута и установка нового значения для showControl
  if (TV) {
    clearTimeout(timeout.current);
  } else {
    window.clearTimeout(timeout.current);
  }
  setShowControl({ show: true, clear, quick });
}

// Установка значения для showControl
function setShowControl(value) {
  showControlEl.current = value;
  setShControl(value);
  if (!value.show) {
    // Если showControl.show равно false, вызывается onHide
    if (onHide) onHide({ clear: value.clear });
    return;
  } else if (onShow && !loading.current) {
    // Вызов onShow для показа элементов управления
    onShow({ clear: value.clear });
  }

  if (TV) {
    // Установка таймаута для скрытия элементов управления
    timeout.current = setTimeout(() => {
      setShowControl({ ...value, show: false });
      clearTimeout(timeout.current);
    }, CONTROLS_HIDE_TIMEOUT);
  } else {
    timeout.current = window.setTimeout(() => {
      setShowControl({ ...value, show: false });
      clearTimeout(timeout.current);
    }, CONTROLS_HIDE_TIMEOUT);
  }
}
```

===
==- Часть 2
```js
// Получение текущего времени проигрывателя
function getCurrentTimePlayer(ply = null) {
  // Если передан параметр ply, используется его значение, иначе используется значение player или playerRef.current
  var temp = ply ?? (player || playerRef.current);
  // Округление текущего времени проигрывателя
  temp = Math.floor(temp.currentTime);
  return temp;
}

// Перемещение проигрывателя на указанное количество секунд
function seekTo(sec) {
  player.currentTime = sec;
}

// Перемещение проигрывателя на позицию пропуска контента
function jumpVideo() {
  player.currentTime = isSkip.pos ?? 0;
}

// Проверка, находится ли проигрыватель на паузе
function paused(ply) {
  var temp = ply ?? (player || playerRef.current);
  return temp.paused;
}

// Получение длительности видео
function getVideoDuration() {
  var dur = player?.duration;
  // Если значение длительности некорректное (NaN), используется значение durationOfSeconds
  dur = Number.isNaN(dur) ? durationOfSeconds : Math.floor(dur);
  return dur;
}

// Сброс слайдера
function resetSlider() {
  var videoDuration = getVideoDuration();

  var oldPos = TVPos
    ? TVPos > videoDuration
      ? videoDuration - 30
      : TVPos
    : null;

  var playerCurrentTime = getCurrentTimePlayer();

  if (playerCurrentTime <= 0 && (oldPos || currentPosition != 0))
    seekTo(oldPos ?? currentPosition);

  var c = videoDuration;

  if (TV) {
    var start = unixTimeToFormatted(startTimeEl.current).split(":");
    var t1 = start[0] * 60 + +start[1];
    var end = unixTimeToFormatted(endTime).split(":");
    var t2 = end[0] * 60 + +end[1];
    durationOfSecondsEl.current = (t2 - t1) * 60;
    setDurationOfSeconds((t2 - t1) * 60);
    return;
  }

  durationOfSecondsEl.current = Number.isNaN(c) ? 0 : c;
  setDurationOfSeconds(Number.isNaN(c) ? 0 : c);

  updateProgressBar();
}

// Запуск обновления позиции проигрывателя
function startUpdate() {
  player.removeEventListener("timeupdate", updateProgressBar);
  player.addEventListener("timeupdate", updateProgressBar, false);

  return () => {
    player.removeEventListener("timeupdate", updateProgressBar);
  };
}

// Обновление прогресс-бара
function updateProgressBar() {
  // Проверка условий для обновления прогресс-бара
  if (
    !progressBar.current ||
    showControlEl.current?.clear ||
    loading.current ||
    paused() ||
    isSkipEl.current?.show
  )
    return;

  var playerCurrentTime = getCurrentTimePlayer();

  var percentage = isLive ? 0 : playerCurrentTime;

  if (isLive) {
    var t = getDateWithTimezone();
    var start = unixTimeToFormatted(startTime).split(":");
    var t1 = (start[0] * 60 + +start[1]) * 60;
    percentage = (t.getHours() * 60 + t.getMinutes()) * 60 + t.getSeconds();
    percentage -= t1;
  }

  if (setPosition && record.current)
    setPosition(
      playerCurrentTime <=
        (durationOfSecondsEl.current > 0
          ? durationOfSecondsEl.current
          : getVideoDuration()) -
          60
        ? playerCurrentTime
        : -1
    );

  setSeekToolTip(percentage);
  setTrackValue(percentage);
}

// Функция для переключения состояния проигрывания
var togglePlay = () => {
  // Изменение состояния проигрывания
  setIsPlay(!isPlayRef.current);
  // Постановка проигрывателя на паузу или воспроизведение
  if (isPlayRef.current) playerRef.current.pause();
  else playerRef.current.play();
};

// Референс для значения longJumping
var longJumping = useRef({ long: false, time: false });

// Обработка события нажатия клавиши на прогресс-баре
var progressKeyDown = (e) => {
  // Проверка, является ли видео прямой трансляцией
  if (isLive) return;
  // Проверка нажатых клавиш LEFT и RIGHT
  if (e.keyCode !== 37 && e.keyCode !== 39) return;
  // Если долгое удержание еще не активировано, активируем его через 1 секунду
  if (!longJumping.current.time)
    longJumping.current.time = setTimeout(
      () => (longJumping.current.long = true),
      1000
    );

  switch (e.keyCode) {
    case 37:
      // Перемотка назад с использованием долгого удержания или обычной длительности SKIP_TIME
      skipBack(longJumping.current.long ? 45 : SKIP_TIME);
      break;
    case 39:
      // Перемотка вперед с использованием долгого удержания или обычной длительности SKIP_TIME
      skip(longJumping.current.long ? 45 : SKIP_TIME);
      break;
  }
};

// Обработка события отпускания клавиши на прогресс-баре
var progressKeyUp = (e) => {
  // Проверка нажатых клавиш LEFT и RIGHT
  if (e.keyCode !== 37 && e.keyCode !== 39) return;
  // Очистка таймаута для долгого удержания
  clearTimeout(longJumping.current.time);
  longJumping.current = { long: false, time: false };
};

// Функция для установки пропуска контента
function isSkipFunc(value, custom = null, pos = 0) {
  // Установка состояния пропуска контента
  var timeV = value == isSkip.forward ? isSkip.time ?? 0 : 0;
  setIsSkip((prev) => {
    return {
      show: true,
      forward: value,
      time: timeV + (custom ?? SKIP_TIME),
      pos,
    };
  }, false);
  isSkipEl.current = {
    show: true,
    forward: value,
    time: timeV + (custom ?? SKIP_TIME),
    pos,
  };
  setIsSkip((prev) => {
    return {
      show: false,
      forward: value,
      time: timeV + (custom ?? SKIP_TIME),
      pos,
    };
  });
}

// Функция для перемотки назад
var skipBack = (custom = null) => {
  var inter = custom ?? SKIP_TIME;
  if (isLive) {
    var t = getDateWithTimezone();
    var start = unixTimeToFormatted(startTime).split(":");
    var t1 = (start[0] * 60 + +start[1]) * 60;
    var currentTime =
      (t.getHours() * 60 + t.getMinutes()) * 60 + t.getSeconds();
    currentTime -= t1;

    setTVPos(currentTime - inter);
    onClickRestart();
    return;
  }
  progressBar.current.value -= inter;
  isSkipFunc(false, custom, progressBar.current.value);
  setSeekToolTip(progressBar.current.value, true);
  setTrackValue(progressBar.current.value);
};

// Функция для перемотки вперед
var skip = (custom = null) => {
  var inter = custom ?? SKIP_TIME;
  progressBar.current.value = +progressBar.current.value + inter;
  isSkipFunc(true, custom, progressBar.current.value);
  setSeekToolTip(progressBar.current.value, true);
  setTrackValue(progressBar.current.value);
};
```
===

==- Часть 3
```js
// Установка значения трека (прогресс-бара)
function setTrackValue(value) {
  // Вычисление процента на основе переданного значения
  var percent = isNaN(value) ? 0 : value < 0 ? 0 : value;
  // Если текущее время проигрывателя меньше или равно 0, устанавливаем процент в 0
  if (getCurrentTimePlayer() <= 0) percent = 0;
  // Установка значения прогресс-бара
  progressBar.current.value = percent;
  // Вычисление значения прогресса в процентах
  var progValue =
    durationOfSecondsEl.current === 0
      ? 0
      : parseFloat((100 * percent) / durationOfSecondsEl.current).toFixed(2);
  // Установка размера фона прогресс-бара
  progressBar.current.style.backgroundSize = progValue + "% 100%";

  // Вычисление нового значения для ползунка трека (sliderThumb) и подсказки (seekToolTip)
  var newValue =
    progValue === 0
      ? 0
      : ((percent - progressBar.current.min) * 100) /
        (progressBar.current.max - progressBar.current.min);

  // Установка позиции ползунка трека
  if (sliderThumb.current)
    sliderThumb.current.style.left = `${Math.min(newValue, 100)}%`;
  // Установка позиции подсказки
  if (seekToolTip.current)
    seekToolTip.current.style.left = `${Math.min(newValue, 100)}%`;
}

// Установка значения подсказки (seekToolTip)
function setSeekToolTip(durationFilm, line = false) {
  // Вычисление процента на основе переданного значения длительности фильма
  var percentage = isLive
    ? 0
    : TV && !line
    ? getCurrentTimePlayer()
    : durationFilm;

  var fPercentage = percentage;
  if (isLive) {
    var t = getDateWithTimezone();
    percentage = (t.getHours() * 60 + t.getMinutes()) * 60 + t.getSeconds();
  } else if (TV) {
    var times = toHHMMSS(percentage).split(":");
    var startTimes = unixTimeToFormatted(startTimeEl.current).split(":");
    var t1, t2;
    if (times.length > 2) {
      t1 = (startTimes[0] * 60 + +startTimes[1]) * 60;
      t2 = (times[0] * 60 + +times[1]) * 60 + +times[2];
      var t = t1 + t2;
      percentage = t;
    } else {
      t1 = (startTimes[0] * 60 + +startTimes[1]) * 60;
      t2 = times[0] * 60 + +times[1];
      var t = t1 + t2;
      percentage = t;
    }

    var t = getDateWithTimezone();
    var currentTime =
      (t.getHours() * 60 + t.getMinutes()) * 60 + t.getSeconds();
    currentTime -= t1;
    var ct = fPercentage;
    if (
      ct >= currentTime ||
      (isCurrentProgram && ct >= getVideoDuration())
    ) {
      setTVPos(0);
      onClickLive();
    }
  }

  // Если текущее время превышает длительность видео и имеется следующая программа, выполняется переключение на следующую программу
  if (
    TV &&
    (isNaN(fPercentage) ? 0 : fPercentage) > durationOfSeconds &&
    durationOfSeconds > 0
  ) {
    setTVPos(1);
    onAutoNext();
  }

  // Форматирование длительности в часы, минуты и секунды
  var formattedDuration = toHHMMSS(percentage);
  seekToolTip.current.innerText = formattedDuration;

  // Обновление текущей длительности видео
  if (currentDurationEl.current && !startTime)
    currentDurationEl.current.innerText = formattedDuration;
}

// Обработчик клика по рекламе
function onClickAD(mode) {
  if (!mode) return;
  // Установка режима модального окна
  setModalMode(mode);
  // Показ модального окна
  if (onShowModal) onShowModal(mode);
  toggleModal();
}

// Состояние отображения модального окна и функция для его переключения
var [isShowingModal, toggleModal] = useModal(false);

// Референс для элемента управления проигрывателя (playerControls)
var playerControls = useRef();
```
===


==- Часть 4
```js
return (
  <React.Fragment>
    {/* Модальное окно */}
    <Modal
      bottom={() => null}
      show={isShowingModal}
      long
      hide={() => {
        // Скрытие модального окна и выполнение onHideModal и toggleModal
        if (onHideModal) onHideModal();
        toggleModal();
      }}
    >
      <div className={"col " + classes.audioTracks}>
        {/* Отображение списка аудиодорожек, субтитров, уровней или форматов видео */}
        {(modalMode === "aspect"
          ? aspects
          : modalMode === "levels"
          ? Array.isArray(levels)
            ? levels
            : []
          : modalMode === "audio"
          ? Array.isArray(tracks)
            ? tracks
            : []
          : Array.isArray(subtitles)
          ? subtitles
          : []
        ).map((tr, i) => {
          // Проверка текущего режима модального окна
          const isAudio = modalMode == "audio";
          const isSub = modalMode == "subtitles";
          const isLevel = modalMode == "levels";
          const isAspect = modalMode == "aspect";

          // Определение текущего выбранного идентификатора
          const currentId = isAspect
            ? aspect
            : isAudio
            ? track
            : isLevel
            ? level
            : subtitleTrack;

          // Определение, является ли текущий элемент выбранным
          const isSelected =
            currentId === (isAspect ? tr.text : isLevel ? i : tr.id) ||
            (!currentId && tr?.default);

          return (
            <Focus
              key={tr.id || i}
              onHandleEnter={() => {
                // Обработчик выбора элемента
                if (isSub) onChangeSubtitle(tr);
                else if (isAudio) onChangeAudioTrack(tr.id);
                else if (isAspect) onChangeAspect(tr.text);
                else onChangeLevel({ ...tr, id: i });
                // Выполнение onHideModal и toggleModal
                if (onHideModal) onHideModal();
                toggleModal();
              }}
              className={[
                classes.track,
                isLevel || isAspect ? classes.lvl : "",
                isSelected ? classes.trackSelected : "",
              ].join(" ")}
              makeFocus={isSelected}
              focusClass="focus-link"
              tab={1}
            >
              {isAspect
                ? tr.text.includes(":")
                  ? tr.text
                  : t(tr.text)
                : isLevel
                ? tr.height + "p"
                : tr?.lang || t("undefined")}
            </Focus>
          );
        })}
        {/* Отображение опции "Выключено" для субтитров */}
        {modalMode === "subtitles" ? (
          <Focus
            onHandleEnter={() => {
              // Выключение субтитров
              onChangeSubtitle("off");
              // Выполнение onHideModal и toggleModal
              if (onHideModal) onHideModal();
              toggleModal();
            }}
            className={
              classes.track +
              " " +
              (subtitleTrack === undefined || subtitleTrack === "off"
                ? classes.trackSelected
                : "")
            }
            makeFocus={subtitleTrack === "off" || subtitleTrack === undefined}
            focusClass="focus-link"
            tab={1}
          >
            {t("Off")}
          </Focus>
        ) : (
          // Отображение опции "Авто" для уровней видео
          modalMode === "levels" && (
            <Focus
              onHandleEnter={() => {
                // Выбор опции "Авто"
                onChangeLevel({ id: -1 });
                // Выполнение onHideModal и toggleModal
                if (onHideModal) onHideModal();
                toggleModal();
              }}
              className={
                classes.track +
                " " +
                (level === -1 ? classes.trackSelected : "")
              }
              focusClass="focus-link"
              tab={1}
            >
              {t("Auto")}
            </Focus>
          )
        )}
      </div>
    </Modal>

    {/* Анимация перехода */}
    <CSSTransition
      unmountOnExit
      classNames="fade2"
      in={isSkip.show}
      timeout={200}
      onExit={() => {
        // Переход к указанной позиции в видео
        jumpVideo();
      }}
      onExited={() => {
        // Сброс информации о переходе
        isSkipEl.current = {};
        if (isSkip?.time === 0) return;
        setIsSkip({ ...isSkip, time: 0 }, false);
      }}
    >
      <div
        style={{
          position: "absolute",
          top: "0",
          [isSkip.forward ? "right" : "left"]: "0",
          height: "100%",
          width: "50%",
          display: "flex",
          justifyContent: "center",
          alignItems: "center",
          background: "#00000059",
          zIndex: 2,
        }}
      >
        <div>
          <img
            width={is4k === 2 ? 140 : is4k === 1 ? 360 : 180}
            src={require(`../../assets/images/${
              isSkip.forward ? "rightSkipNew.png" : "leftSkipNew.png"
            }`)}
          />
          <p className="fs-50 control">
            {isSkip.forward ? "+" : "-"}
            {Math.floor(isSkip.time)} {t("sec.")}
          </p>
        </div>
      </div>
    </CSSTransition>
    {/* Отображение элементов управления плеером */}
    {showControl.show && (
      <div
        id="player-controls"
        ref={playerControls}
        className={classes.background}
      >
        <div
          style={showControl.clear ? { padding: 0 } : {}}
          className={
            classes.controls +
            (levels?.length > 1 || tracks?.length > 1 || subtitles?.length > 0
              ? " " +
                (TV ? classes.control_padding_tv : classes.control_padding)
              : "")
          }
        >
          <div className={classes.progress_wrapper}>
            <h1
              style={showControl.clear ? { display: "none" } : {}}
              ref={currentDurationEl}
              id="current-duration"
            >
              {unixTimeToFormatted(startTime)}
            </h1>
            <Space
              style={showControl.clear ? { display: "none" } : {}}
              width={30}
            />
            <div className={classes.video_progress}>
              {!showControl.quick && children}
              <div
                style={showControl.clear ? { display: "none" } : {}}
                className={classes.slider_wrapper}
              >
                <Focus
                  ref={rangeVideo}
                  onFocus={() => {
                    return loading.current || !seekToolTip.current?.style
                      ? null
                      : (seekToolTip.current.style.display = "block");
                  }}
                  onUnfocus={() => {
                    return loading.current || !seekToolTip.current?.style
                      ? null
                      : (seekToolTip.current.style.display = "none");
                  }}
                  tag="input"
                  className={classes.seek}
                  onKeyDown={progressKeyDown}
                  onKeyUp={progressKeyUp}
                  focusClass="focus-progress"
                  id="progress-bar-player"
                  max={durationOfSeconds}
                  min="0"
                  type="range"
                />
                <span
                  ref={sliderThumb}
                  className={classes.slider_thumb}
                ></span>
              </div>
              <div
                style={showControl.clear ? { display: "none" } : {}}
                className={classes.current}
                ref={seekToolTip}
                id="seek-tooltip"
              >
                00:00
              </div>
            </div>
            <Space
              style={showControl.clear ? { display: "none" } : {}}
              width={30}
            />
            <h1
              style={showControl.clear ? { display: "none" } : {}}
              ref={videoDurationEl}
              id="video-duration"
            >
              {endTime
                ? unixTimeToFormatted(endTime)
                : durationOfSeconds
                ? toHHMMSS(durationOfSeconds)
                : "00:00"}
            </h1>
          </div>
          <div
            style={showControl.clear ? { display: "none" } : {}}
            className={[
              "row",
              (left || right) && !loading.current ? "between" : "center",
              classes.control_bottom,
            ].join(" ")}
          >
            {!showControl.quick &&
              (left ?? <span style={{ width: "300px" }}></span>)}
            <span
              className={"row " + classes.control_buttons}
              style={
                showControl.quick
                  ? { display: "none" }
                  : { alignItems: "center" }
              }
            >
              <FocusImage
                data-margin
                onHandleEnter={onPrevious}
                heightImg={HEIGHT_ICON}
                img={
                  TV
                    ? modifiedPlugins.previous.show && left
                      ? require("../../assets/images/Left.png")
                      : ""
                    : modifiedPlugins.previous.show
                    ? require("../../assets/images/Left.png")
                    : ""
                }
                focusImg={
                  TV
                    ? modifiedPlugins.previous.show && left
                      ? require("../../assets/images/left1.png")
                      : ""
                    : modifiedPlugins.previous.show
                    ? require("../../assets/images/left1.png")
                    : ""
                }
              />
              {modifiedPlugins.back.show && (
                <FocusImage
                  onHandleEnter={() => skipBack()}
                  heightImg={HEIGHT_ICON}
                  img={require("../../assets/images/leftSkip.png")}
                  focusImg={require("../../assets/images/leftSkip1.png")}
                />
              )}
              {modifiedPlugins.playStop.show && (
                <div className={classes.center_buttons}>
                  <FocusImage
                    data-play
                    onHandleEnter={togglePlay}
                    heightImg={is4k === 2 ? 50 : is4k === 1 ? 120 : 86}
                    widthImg={is4k === 2 ? 45 : is4k === 1 ? 100 : 75}
                    img={require(`../../assets/images/${
                      !isPlay ? "startIcon" : "Pause-button"
                    }.png`)}
                    focusImg={require(`../../assets/images/${
                      !isPlay ? "startIcon1" : "Pause-button1"
                    }.png`)}
                  />
                  <div className={classes.tracks}>
                    {tracks?.length > 1 && (
                      <Focus
                        onHandleEnter={() => onClickAD("audio")}
                        focusClass="focus-secondary"
                        className={classes.trackBtn}
                      >
                        <img src={audioTrack} alt="audio-track" />
                      </Focus>
                    )}
                    {subtitles?.length > 0 && (
                      <Focus
                        onHandleEnter={() => onClickAD("subtitles")}
                        focusClass="focus-secondary"
                        className={classes.trackBtn}
                      >
                        <img
                          src={subtitleTrackIcon}
                          alt="subtitle-track"
                        />
                      </Focus>
                    )}
                    {levels?.length > 1 && (
                      <Focus
                        onHandleEnter={() => onClickAD("levels")}
                        focusClass="focus-secondary"
                        className={classes.trackBtn}
                      >
                        <img
                          src={require("../../assets/LevelTrack.png")}
                          alt="video-level"
                        />
                      </Focus>
                    )}
                  </div>
                </div>
              )}
              {modifiedPlugins.skip.show && (
                <FocusImage
                  onHandleEnter={() => skip()}
                  heightImg={HEIGHT_ICON}
                  img={require("../../assets/images/rightSkip.png")}
                  focusImg={require("../../assets/images/rightSkip1.png")}
                />
              )}

              {!isLive && !isCurrentProgram && (
                <FocusImage
                  data-margin
                  onHandleEnter={onNext}
                  heightImg={HEIGHT_ICON}
                  img={
                    modifiedPlugins.next.show
                      ? require("../../assets/images/Right.png")
                      : ""
                  }
                  focusImg={
                    modifiedPlugins.next.show
                      ? require("../../assets/images/right1.png")
                      : ""
                  }
                />
              )}

              {isLive && (
                <FocusImage
                  data-margin
                  onHandleEnter={() => {
                    setTVPos(1);
                    onClickRestart();
                  }}
                  heightImg={45}
                  img={require("../../assets/images/restart0.png")}
                  focusImg={require("../../assets/images/restart1.png")}
                />
              )}
              {TV && (
                <React.Fragment>
                  <Focus
                    onHandleEnter={() => !isLive && onClickLive()}
                    focusClass="focus-secondary"
                    className={[
                      classes.liveButton,
                      isLive ? classes.liveOn : "",
                    ].join(" ")}
                  >
                    <div>
                      <span></span>
                      <p>{t("Live")}</p>
                    </div>
                  </Focus>
                </React.Fragment>
              )}
              {/* <Focus
                onHandleEnter={() => onClickAD("aspect")}
                focusClass="focus"
                className={classes.aspectBtn}
              >
                <img src={aspectRatioIcon} alt="." />
              </Focus> */}
            </span>
            {!showControl.quick &&
              (right ?? <span style={{ width: "300px" }}></span>)}
          </div>
        </div>
      </div>
    )}
  </React.Fragment>
);
```
===
