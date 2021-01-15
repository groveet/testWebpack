создаем папку с проектом

прикрепляем git

инициализируем npm

```jsx
npm init
```

при опросе в инициализации ставил точку входа webpack.config из корня проекта

Устанавливаем вебпак

```jsx
npm i webpack webpack-cli --save-dev
```



создаем в корне папку src и там index.js для теста напишем что то в console.log

настраиваем пути для исходников сборки в файле webpack.config.js

```jsx
const path = require('path')

module.exports = {
  context: path.resolve(__dirname, 'src'),
  mode: 'development',

  // главный файл с которого все начинается
  entry: './index.js',

  // выходной файл где будут js скрипты
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
}
```

добавляем скрипты в package.json для автоматизации

```jsx
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "webpack",
    "build": "webpack --mode production"
  },
```

затестим

Создадим в src файл module.js и пропишем что-нибудь в console.log

подключим этот новый файл в index.js

```jsx
import './module'
```

запустим сборку

```jsx
npm run start
```

и откроем наш собранный js

```jsx
node dist/bundle.js
```

должно написать оба console.log (из файла index.js и module.js)

в .gitignore добавим папки

- node_modules
- dist

файл webpack.config.js

пропишем добавление хешей в название файлов

```jsx
output: {
    filename: 'bundle[hash].js',
    path: path.resolve(__dirname, 'dist')
  },
```

### настроем плагины

скачаем плагины

```jsx
npm install --save-dev html-webpack-plugin

// для фавиконки
npm install copy-webpack-plugin --save-dev

// чистит к примеру старые хешированные файли от сборки
npm install clean-webpack-plugin --save-dev

npm install mini-css-extract-plugin --save-dev

```

пропишем плагины и настроем алиасы

файл webpack.config.js

```jsx
const path = require('path')
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const HTMLWebpackPlugin = require('html-webpack-plugin')
const CopyPlugin = require('copy-webpack-plugin')
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
	...,

  resolve: {
    extensions: ['.js'],
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@core': path.resolve(__dirname, 'src/core')
    }
  },

  plugins: [
    new CleanWebpackPlugin(),
    new HTMLWebpackPlugin({
      // откуда берем шаблон для html
      template: 'index.html'
    }),
    new CopyPlugin({
      patterns: [
        { from: path.resolve(__dirname, 'src/favicon.ico'), to: path.resolve(__dirname, 'dist') },
      ],
    }),
    new MiniCssExtractPlugin({
      // выходной файл где будут css стили
      filename: 'bundle.[hash].css'
    }),
  ]
}
```

в src создадим файл index.html в котором будет стандартная html разметка где в боди пропишем следующее

```html
<div id="app"></div>
```

и в хеде пропишем фавиконку

```html
<link rel="stylesheet icon" href="favicon.ico">
```

устанавливаем sass

```html
npm install sass-loader sass webpack --save-dev
npm install css-loader --save-dev
```

и прописываем его в конфиге вебпака

```html
module: {
    rules: [
      {
        test: /\.s[ac]ss$/i,
        use: [
          MiniCssExtractPlugin.loader,
          "css-loader",
          "sass-loader",
        ],
      },
    ],
  },
```

в папке src создаем папку scss а в ней файл index.scss

и подключаем его в index.js

```jsx
import './scss/index.scss'
```

затестим

в файле index.scss пишем

```html
$red: blue;

body {
  background: $red;
}
```

запускаем сборку и видем в папке дист сбилженный css с синим фоном у боди

### подключаем babel

качаем его и пресет

```html
npm install --save-dev babel-loader @babel/core
npm install @babel/preset-env --save-dev
```

в конфиге вепака modules - rules добавляем новое правило

```html
{
        test: /\.m?js$/,
        exclude: /node_modules/,
        use: {
          loader: "babel-loader",
          options: {
            presets: ['@babel/preset-env']
          }
        }
      }
```

в package.json добавляем

```html
"browserslist": "> 0.25%, not dead",
```

### настроим режимы сборки

ставим

```html
npm i cross-env --save-dev
```

меняем скрипты в package.json

```html
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "cross-env NODE_ENV=development webpack",
    "build": "cross-env NODE_ENV=production webpack --mode production"
  },
```

в конфиге вебпака пишем вверху

```html
const isProd = process.env.NODE_ENV === 'production'
const isDev = !isProd
```

и редактируем там же плагин html (настраиваем сжатие html)

```html
new HTMLWebpackPlugin({
      // откуда берем шаблон для html
      template: 'index.html',
      minify: {
        removeComments: isProd,
        collapseWhitespace: isProd
      }
    }),
```

чекаем сборку и в зависимости от режима сборки прописываем хеши в названии файлов

```html
const filename = *ext* => isDev ? `bundle.${ext}` : `bundle.[hash].${ext}`

// и меняем filemane
...
output: {
    filename: filename('js'),
    path: path.resolve(__dirname, 'dist')
  },
...
new MiniCssExtractPlugin({
      // выходной файл где будут css стили
      filename: filename('css'),
    }),
```

### добавляем SourseMap

в конфиге вебпака

```html
devtool: isDev ? 'source-map' : false,
```

### ставим DevServer

```html
npm install webpack-dev-server --save-dev
```

package.json

```html
"scripts": {
    ...
    "start": "cross-env NODE_ENV=development webpack serve --open",
		...
  },
```

config.webpack.js

```html
devServer: {
    port: 3000,
    hot: isDev
  },
```

```html
npm install --save @babel/polyfill
```

конфиг вебпака редактируем entry

```html
entry: ['@babel/polyfill', './index.js'],
```

фиксим обновление стилей в конфиге вебпака

```html
...
target: 'web'
...
```

### устанавливаем EsLint

```html
npm i eslint eslint-loader babel-eslint --save-dev
```

редактируем конфиг вебпака

```html
...
const jsLoaders = () => {
  const loaders = [
    {
      loader: 'babel-loader',
      options: {
        presets: ['@babel/preset-env']
      }
    }
  ]

  if (isDev) {
  
  }

  return loaders
}
...
это в module/rules
{
        test: /\.m?js$/,
        exclude: /node_modules/,
        use: jsLoaders(),
      }
...
```

ставим пресет eslint от гугла

```html
npm install --save-dev eslint-config-google
```

на будущее в корне создадим файл .eslintignore

пока что пустой)

инициализируем eslint

```html
./node_modules/.bin/eslint --init
//     или 
// eslint --init
```

и настраиваем rules в файлне .eslintrc.js

наводим на ошибку копируем название ошибки и вставляем в рулс (пример ниже)

```html
"rules": {
    "semi": "off",
    "quotes": "off",
    "arrow-parens": "off",
    "require-jsdoc": "off"
  },
```

если честно то хуета какая то этот еслинт, возможно как то настроить надо лучше, хз



### заливаем все это дело в гит

(командная строка)

```html
git add .

git commit -m "finish project conf"

git push -u origin webpack

```

на сайте мержим ветку в мастера (если есть она)

потом в редакторе кода переключаемся на ветку мастера

```html
git checkout master
```

и обновляем файлы

```html
git pull
```