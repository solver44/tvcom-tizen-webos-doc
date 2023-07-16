# Сам плеер

!!!
/src/components/UI/VideoPlayer.js
!!!

## Props

- `children`: React-компоненты, которые будут отображаться внутри компонента `VideoPlayer`.
- `onLoaded`: Функция, которая будет вызвана при загрузке и готовности видео к воспроизведению.
- `isTV`: Флаг типа boolean, указывающий, воспроизводится ли видео на телевизоре.
- `isSeries`: Функция, которая возвращает информацию о воспроизводимом видео, в частности, является ли оно частью сериала.

### States

- `isShowIcon`: Булевое состояние для управления видимостью иконки паузы/воспроизведения.
- `isPlaying`: Булевое состояние, указывающее, воспроизводится ли видео в данный момент.
- `isWaiting`: Булевое состояние, указывающее, ожидает ли видео загрузку.

### Refs

- `isWaitingTimeout`: Ссылка на таймер для задержки состояния ожидания.
- `isPlayingTimeout`: Ссылка на таймер для задержки состояния воспроизведения.
- `isPlayingVideo`: Ссылка на флаг, указывающий, воспроизводится ли видео в данный момент.
- `isStarted`: Ссылка на флаг, указывающий, началось ли воспроизведение видео.

### Обработчики событий

- `waitingHandler`: Функция-обработчик события "waiting" видеоэлемента.
- `playHandler`: Функция-обработчик событий "play" и "playing" видеоэлемента.
- `pauseHandler`: Функция-обработчик события "pause" видеоэлемента.
- `changedData`: Функция-обработчик события "progress" видеоэлемента.
- `canPlay`: Функция-обработчик события "canplay" видеоэлемента.

### Events

- Устанавливает слушатели событий для видеоэлемента, включая события "waiting", "play", "playing", "pause", "canplay" и "progress".

### Компонент SeriesNumber

- Вложенный компонент внутри `VideoPlayer`, отвечающий за отображение название серии.
- Использует функцию `isSeries` для определения эпизода.
- Использует таймаут для скрытия название серии.

## Структура компонента

```jsx
<div className={classes.player_wrapper}>
  {isSeries() && isPlaying && isStarted.current && <SeriesNumber />}
  <video
    className={classes.player_tv}
    ref={videoElementRef}
    onEnded={onVideoEnded}
    autoPlay
  ></video>
  {isWaiting && (
    <Loader
      style={{ top: 0, left: 0, bottom: 0, right: 0 }}
      transparent
      full
    />
  )}
  <CSSTransition
    classNames="bounce"
    unmountOnExit
    timeout={300}
    in={isShowIcon && !isPlaying}
  >
    <img
      className={classes.pauseAndPlay}
      src={require(`../../../assets/images/${
        isPlaying ? "playIcon.png" : "Pause-button.png"
      }`)}
    />
  </CSSTransition>
  {children}
</div>
```

## CSS Модули

- `/src/components/UI/VideoPlayer/style.module.css`: Файл CSS-модуля, содержащий стили для компонента `VideoPlayer`.
```