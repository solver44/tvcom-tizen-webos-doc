# Getting Started
### TVCOM created with React for Tizen and WEBOS

### Requirements
- `Node 16 version`
- `Tizen sdk`
- `WEBOS sdk`

### Source
 - [Tizen SDK](https://developer.tizen.org/development/tizen-studio/download)
 - [WEBOS](https://webostv.developer.lge.com/develop/tools/sdk-downloading-installer)


```bash
# install dependencies
$ npm install --force

# serve with hot reload at localhost:3000
$ npm start

# build for production and deploy to TVCOM-deploy
$ npm run build-rewire-mac #for windows <build-rewire>
# copy this files and folders to TVCOM-deploy/static folder

# build/runtime-main.js -> static/js
# build/static/media -> static
# build/main.chunk.css -> static/css

# and these are all the steps to deploy to production!
```

```bash
# собрать и запустить на устройстве Tizen (Samsung)
$ npm run build
$ npm run tizen-tv
```
!!!warning Warning
Для установки для webos(LG) потребуется WEBOS IDE
!!!