# webpack_practice
# Section 1

### nodejsのインストール

- nvm
  - https://github.com/nvm-sh/nvm
- node
  - https://nodejs.org/en/


#### nvm

```shell
# Mac
% curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.2/install.sh | bash

# 確認
% vim ~/.zshrc

# ターミナルを再起動
% nvm --version
=> 0.35.2
```


#### node

```shell
# インストール可能なnodeバージョンを確認
% nvm ls-remote

# 特定のnodeバージョンをインストール
% nvm install 14.15.0

# インストールされているnodeバージョンを確認
% nvm ls
->     v14.15.0

# 現在のnodeバージョンを確認
% node --version
v14.15.0
```
　　
#### Webpackのインストール

```shell
% npm init

% npm view webpack
# latest: 5.15.0

% npm view webpack-cli
# latest: 4.3.1

% npm install --save-dev webpack@5.15.0 webpack-cli@4.3.1
```

#### --save / --save-dev / オプションなしの違い

- `--save dependencies` を付けると `dependencies` に入る
- `--save-dev` を付けると `devDependencies` に入る

コードをパッケージ化して公開する時に影響する。
dependenciesは `npm install` の際に一緒にインストールされる。
devDependenciesは `npm install` してもインストールされない。

今回は自分用のプロジェクトであり、パッケージ化しないので、すべて `devDependencies` で良い。
もしモジュールとして他の人が使えるパッケージにしたい、という場合は `dependencies` を使っていく。


### 初めてのビルド

```shell
% mkdir src // Webpackのデフォルト設定（後で変更することもできます）

### VS Code が立ち上がります
% code index.js

% npx webpack
% npx webpack --mode=development
```

### 確認

```shell
% code dist/main.js
```

### モジュールを読み込む

```shell
% mkdir ./src/modules
% code ./src/modules/my.js
```

```js
// modules/my.js

export default () => {
  console.log('this is module');
};


// src/index.js

import my from './modules/my';

console.log('This is index.js');
my();
```


### gitignore

ここまで来たらGitにコミットしてGithubにプッシュしましょう。
その前に `.gitignore` を作成します。

```shell
% code .gitignore
```

```
# .gitignore

node_modules/
```

これで `node_module` をGitの追跡から外すことができます。










# Section 2

このブランチの前に行ったこと
--------------------------------

### 設定ファイルなしでのバンドル

```shell
% npx webpack --mode=development
```

- デフォルトのエントリーポイントは `index.js`
- デフォルトのアウトプットは `dist/main.js`

### 中身の確認

```shell
% code dist/.main.js
```

### Git

- .gitignoreの作成
- Githubへpush


このブランチで行ったこと
--------------------------------

### ビルドされたJSを使ってみる

#### HTMLファイル作成

```shell
% code ./dist/index.html
```

```shell
% code ./src/modules/my.js
```

```js
// index.js
import './modules/my.js'
```


```html
<!-- index.js -->
<script src="./main.js"></script>
```

#### ブラウザで確認

```shell
% open -a "Google Chrome" dist/index.html
```

```
// Chrome Console
This is index.js
main.js:1 this is module
```

### 設定ファイルを使用してビルドする

```shell
% code webpack.config.js
```

```js
module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, './dist'),
  },
};
```

### この状態でビルドするとエラー

```shell
% npx webpack --mode=development

Invalid configuration object. Webpack has been initialised using a configuration object that does not match the API schema.
 - configuration.output.path: The provided value "./dist/main.js" is not an absolute path!
   -> The output directory as **absolute path** (required).
```

#### output.path は絶対パスを指定する

```js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, './dist'),
  },
};
```

#### 出力されるファイル名を変更してみる

```js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: 'index.js', // テストした後は main.js 戻す
  },
};
```

### スタイルシートを読み込んで見る

```shell
% code ./src/modules/my.css
```

```css
/* ./src/modules/my.css */
body {
  color: lightblue;
}
```

```js
// ./src/index.js
import my from './modules/my';
import './modules/my.css'; // add

console.log('This is index.js');
my();
```

#### この状態でビルドするとエラー

```shell
% npx webpack --mode=development

ERROR in ./src/modules/my.css 1:5
Module parse failed: Unexpected token (1:5)
You may need an appropriate loader to handle this file type, currently no loaders are configured to process this file. See https://webpack.js.org/concepts#loaders
> body {
|   color: lightblue;
| }
```

#### ローダーをインストールする

Webpackはjavascript。
Javascript以外のファイルを読み込もうとするとエラーになる。
読み込めるようにするにはローダーと呼ばれるライブラリが必要。

- [css-loader](https://github.com/webpack-contrib/css-loader)
- [style-loader](https://github.com/webpack-contrib/style-loader)

```shell
% npm view css-loader
# latest: 5.0.1

% npm view style-loader
# latest: 2.0.0

% npm install --save-dev css-loader@5.0.1 style-loader@2.0.0
```

#### 確認

```shell
% code package.json
```

```json
{
  "devDependencies": {
    "css-loader": "^5.0.1",
    "style-loader": "^2.0.0",
    "webpack": "^4.41.5",
    "webpack-cli": "^3.3.10"
  }
}
```

### ローダーを使う

```shell
code webpack.config.js
```

```js
// webpack.config.js
module.exports = {
  entry: './src/index.js',
  output: {
    ...
  },
  // moduleを追加
  module: {
    rules: [
      {
        test: /\.css/,
        use: [{
          loader: 'css-loader',
        }],
      },
    ],
  },
};
```

- moduleを追加してruleを設定する。
- ruleは複数設定できるので配列で指定する。=> `rules`
- `test:`はどのファイルが対象になるのかを、正規表現で記述。
- `use:`はどのローダーを使用するかを設定します。

```shell
% npx webpack --mode=development
```

#### 動作確認

index.html をブラウザで確認。正常に動作しているはず。

#### main.js を確認してみる

```shell
% code ./dist/main.js

eval("// Imports\nvar ___CSS_LOADER_API_IMPORT___ = __webpack_require__(/*! ../../node_modules/css-loader/dist/runtime/api.js */ \"./node_modules/css-loader/dist/runtime/api.js\");\nexports = ___CSS_LOADER_API_IMPORT___(false);\n// Module\nexports.push([module.i, \"body {\\n  color: lightblue;\\n}\\n\", \"\"]);\n// Exports\nmodule.exports = exports;\n\n\n//# sourceURL=webpack:///./src/modules/my.css?");

# ↑
# CSSらしき記述は見つかった
#
```

```js
// evalの次に下のコマンドを入れてみる
console.log(exports[0][1]);
```

index.html を確認するとコンソールログが出力されている。

```shell
% open -a "Google Chrome" dist/index.html
```

```
# スタイルが出力された
body {
  color: lightblue;
}

index.js:8 This is index.js
my.js:3 this is module
```

しかしスタイルが反映されていない。
CSSは読み込まれているが、使用されていない状態。

### style-loader を使う

```js
// webpack.config.js
        use: [
          // 追記
          {
            loader: 'style-loader',
          },
          {
            loader: 'css-loader',
          }
        ],
```

再度ビルドする。

```shell
% npx webpack --mode=development
```

#### main.js を確認してみる

```shell
% code ./dist/main.js

# !*** ./node_modules/style-loader/dist/runtime/injectStylesIntoStyleTag.js ***!

# ↑
# Styleタグを注入しているような記述
#
```

#### HTMLを確認する

```shell
% open -a "Google Chrome" dist/index.html
```

index.htmlをブラウザで開くと、文字の色が変わっているはず。
開発者ツールでElementsを確認。

```html
<style>body {
  color: lightblue;
}
</style>
```

スタイルシートがHTMLにインジェクトされている。

### すべてJavascriptにバンドルする方法の問題点

#### CSSが適応される流れ
- css-loaderでCSSをJavascriptに読み込む。
- Webpackでビルドされた `.js` ファイルを `index.html` に配置。
- 読み込まれたJavascriptがstyle-loaderによりHTMLにインジェクトされる。










# Section 3

このブランチの前に行ったこと
--------------------------------

- Webpackの設定ファイルを使ったビルド
  - `./webpack.config.js`
- ローダーのインストール
  - `npm install`コマンド
- CSSを読み込んでスタイルを適応させる
  - css-loader / style-loader

このブランチで行ったこと
--------------------------------

#### 問題 1
技術的な観点：
たしかにスタイルは適応されているので、見た目は問題ない。
ただし、CSSが肥大化してくるとHTMLサイズが大きくなってしまうし、すべてのHTMLでスタイル定義が注入されるのは無駄。

- `.css` を別ファイルに切り出し、そのファイルを毎回参照すれば良い。すべてのHTMLファイルが軽量化する。
- しかもブラウザがCSSをキャッシュしてくれるので、初回アクセス以降はどのページを閲覧してもCSSファイル分の通信が節約できる。

#### 問題 2
ビジネス観点：
従来の静的ウェブサイトのファイル構造と異なるため、受託案件の納品の際に困ることがある。
クライアントサイドでサイト更新を行う場合に、Webpackの使用を強制することになる。

- `.html / .css / .js` という従来通りの構成にしておけば、納品トラブルが避けられる。
- その上で、ビルドツールも提供してあげると喜ばれる。


### プラグインのインストール

CSSを別ファイルに出力するには `MiniCssExtractPlugin` というプラグインを使用します。

```shell
% npm view mini-css-extract-plugin
# latest: 1.3.4

% npm install --save-dev mini-css-extract-plugin@1.3.4
```

### プラグインを使用する

Webpackの設定ファイルを編集します。

```shell
code ./webpack.config.js
```

```js
// webpack.config.js

// 追加
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {

  // ...

  module: {
    rules: [
      {
        test: /\.css/,
        use: [
          {
            // loader: 'style-loader', 削除
            loader: MiniCssExtractPlugin.loader, // 追加
          },
          {
            loader: 'css-loader',
          }
        ],
      },
    ],
  },
  // 追加
  plugins: [
    new MiniCssExtractPlugin(),
  ],
};
```

```shell
% npx webpack --mode=development
```

ビルドすると `/dist/main.css` が新規に出力される。
ファイルとして出力されるので、HTMLにインジェクトする役割の`style-loader`は不要になります。

中身を確認。

```shell
code ./dist/main.css
```

```css
body {
  color: lightblue;
}
```

先程までHTML内にインジェクトされていたスタイルシートがファイルとして抽出された。

### HTMLページに適応する

```shell
% open -a "Google Chrome" ./dist/index.html
```

このままだと、先程HTML内に記述されたスタイルシートが無くなっているだけなので、
文字の色はデフォルトに戻ってしまっている。

```shell
% code ./dist/index.html
```

```html
<!-- /dist/index.html -->
...

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Test Page</title>
  <link rel="stylesheet" href="./main.css"> <!-- 追加 -->
</head>

...
```

上のように出力されたCSSをHTMLに読み込んで、
ブラウザをリロードするとスタイルが適応されて文字の色が変更される。

`/dist/my.css` の内容がそのままコピーされている状態に見えますが、CSSが一旦モジュールとしてJavascriptに読み込まれ、Webpackのプラグインによって再度ファイルとして切り出されているので、完全に別のファイルです。

今の段階だと単に回りくどいだけのように思えますが、効率化のための準備ですので、このまま進めてください。

### HTMLファイルも自動で出力する

今のままだと `/dist/index.html` を編集して、JavascriptやCSSを読み込んでいます。
事情を知っている人は理解できますが、初めてこのファイルを見る人には親切ではない状態です。

編集は`/src`の中で全て行い、ビルドコマンドを実行して作成された`/dist`の中身は変更しないのがベスト。
混乱を避けるため、あるファイルは`/src`を編集し、別のファイルは`/dist`を編集するという状況を改善したいと思います。

#### html-webpack-plugin をインストール

- [html-webpack-plugin](https://github.com/jantimon/html-webpack-plugin)

このプラグインを使うと、HTMLが自動で出力されます。
出力されるHTMLにはWebpackでビルドされるJavascriptとCSSが配置されています。

```shell
% npm view html-webpack-plugin
# latest: 4.5.1
# next: 5.0.0-beta.5 // 5.x に対応するためこちらを使用します

% npm install --save-dev html-webpack-plugin@5.0.0-beta.5
```

#### プラグインを使用する設定を追加

```shell
% code webpack.config.js
```

```js
// webpack.config.js

// 追加
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  // ...
  plugins: [
    new MiniCssExtractPlugin(),
    // 追加
    new HtmlWebpackPlugin(),
  ],
}
```

#### ビルドして確認

```shell
% npx webpack --mode=development

     Asset       Size  Chunks             Chunk Names
index.html  219 bytes          [emitted]
  main.css   30 bytes    main  [emitted]  main
   main.js   5.17 KiB    main  [emitted]  main
```

今までなかった`index.html`が出力されているログが確認できます。
中身を確認しましょう。

```shell
% code ./dist/index.html
```

```html
<!-- /dist/index.html -->
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Webpack App</title>
  <link href="main.css" rel="stylesheet"></head>
  <body>
  <script type="text/javascript" src="main.js"></script></body>
</html>
```

自分で作成したファイルがなくなり、上の内容に置き換わっています。
ブラウザで確認します。

```shell
% open -a "Google Chrome" dist/index.html
```

今まであったテキストは無くなり、完全なブランクページが表示されます。
しかし、開発者ツールのConsoleを見ると、見覚えのあるログが出力されています。

```
This is index.js
this is module
```

#### なにが起こっているのか

- `html-webpack-plugin`によって自動でHTMLが出力されました。
- それによって自分で作成した`index.html`は削除されました（上書きされた）
- 新しい`index.html`にはエントリーポイントとして指定したJavascriptが自動で配置されています。
- エントリーポイントではCSSファイルも読み込んでいますので、先の`mini-css-extract-plugin`によって個別ファイルとして出力されています。
- CSSの読み込みは`html-webpack-plugin`によって検知されており、出力されたCSSも`index.html`に自動で配置されています。


`html-webpack-plugin`はビルドしたファイルを含んだ`.html`を自動で作成してくれるプラグインです。
ただし、これではコンテンツが空のままです。

#### テンプレートを利用する

`html-webpack-plugin`では、出力されるHTMLの雛形を指定することができます。
まずは雛形となるHTMLファイルを作成しましょう。

```shell
% code ./src/index.html
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Template HTML</title>
</head>
<body>
  <h1>Hello World!</h1>
</body>
</html>
```

`.css`や`.js`ファイルは配置されていません。
Webpackの設定ファイルを少し編集して、このHTMLをテンプレートとして使用します。

```js
// webpack.config.js

module.exports = {
  // ...
  plugins: [
    // ...
    // 編集
    new HtmlWebpackPlugin({
      template: './src/index.html',
    }),
  ],
}
```

この状態でビルドしてみましょう。

```shell
% npx webpack --mode=development

Child html-webpack-plugin for "index.html":
     1 asset
    Entrypoint undefined = index.html
    [./node_modules/html-webpack-plugin/lib/loader.js!./src/index.html] 486 bytes {0} [built]
    [./node_modules/webpack/buildin/global.js] (webpack)/buildin/global.js 472 bytes {0} [built]
    [./node_modules/webpack/buildin/module.js] (webpack)/buildin/module.js 497 bytes {0} [built]
```

`./src/index.html` が使用されているログが確認できます。
ブラウザで確認してみてください。

```shell
% open -a "Google Chrome" dist/index.html
```

今度は`Hello World!`という文字が表示されたかと思います。
Javascriptも読み込まれていますので、ブラウザのConsoleでも今まで通りのログが流れています。

ファイルも確認してみてください。

```shell
% code ./dist/index.html
```

この2行が自動で挿入されているのが確認できます。

```html
<!-- /dist/index.html -->
<link href="main.css" rel="stylesheet"></head>
<script type="text/javascript" src="main.js"></script>
```

テンプレートを使用することで`html-webpack-plugin`を使用しつつ、自分のコンテンツを表示することができました。

これによって`/dist`フォルダのすべてのファイルは、ビルドの結果になりました。
`/dist`をフォルダごと削除して、再度ビルドコマンドを実行してみても、常に同じファイルが生成されます。

制作者が変更ファイルは`/scr`フォルダの中に集まり、メンテナンス性が格段に向上した状態です。










# Section 4

このブランチの前に行ったこと
--------------------------------

- CSSファイルを別ファイルに出力する
  - `mini-css-extract-plugin`
- HTMLファイルを自動で生成する
  - `html-webpack-plugin`
- 全てのファイルを`/src`に集約させる

　　　
　　

このブランチで行ったこと
--------------------------------

### Webpackの設定をカスタマイズ

HTML / CSS / JS の構成でファイルが書き出せるようになりました。
`/dist`フォルダには下記のファイルが生成されています。

- index.html
- main.css
- main.js

ウェブサイトを構成する基本的なファイルです。
`/dist`フォルダは一切編集することなく、サイト制作の準備が整ったと言えます。

このセクションでは`webpack.config.js`をカスタマイズして、ファイル構成を改善していきます。

### clean-webpack-plugin

このセクションでは、ファイルの名前や構成を頻繁に変更することになります。
構成を変える前と後では出力されるファイル名が違うため、`dist`フォルダに使用していない不要なファイルが残ってしまいます。

`dist`フォルダを毎回削除しても良いのですが、自動化してしまいましょう。

- [clean-webpack-plugin](https://github.com/johnagan/clean-webpack-plugin) を使用します。

```shell
% npm view clean-webpack-plugin
# latest: 3.0.0

% npm install --save-dev clean-webpack-plugin@3.0.0
```

```js
// webpack.config.js

// 追加
const { CleanWebpackPlugin } = require('clean-webpack-plugin');

module.exports = {
  // ...
  plugins: [
    // ...
    // 追加
    new CleanWebpackPlugin(),
  ],
};
```

`clean-webpack-plugin`が動作しているか確認するために`dist/test.html`というファイルを作成します。

```shell
% npx webpack --mode=development
```

ビルドコマンドを実行すると、`dist`の中に作成した`test.html`ファイルが削除されたかと思います。
`clean-webpack-plugin`が一度`dist`フォルダの中身を全て削除しているのが確認できました。

### .cssファイルの名前を変える

生成されるHTMLにはCSSを読み込む記述が自動で書き込まれるので、意識することは少ないですが、CSSファイルの名前を変更することもできます。

```js
// webpack.config.js

module.exports = {
 // ...
  plugins: [
    new MiniCssExtractPlugin({
      filename: 'my.css', // 追加
    }),
    new HtmlWebpackPlugin({
      template: './src/index.html',
    }),
  ],
}
```

```shell
% npx webpack --mode=development
```

`filename`を指定したことによって、ファイルが`my.css`として保存されました。
それに伴い`index.html`に記述されているCSS読み込みの記述も変更されています。

```
# ファイル構成

dist/
    + -- my.css
    + -- index.html
    + -- main.js
```

自動的にフォルダをつくって、その配下に置くこともできます。

```js
// webpack.config.js

module.exports = {
 // ...
  plugins: [
    new MiniCssExtractPlugin({
      filename: './stylesheets/my.css', // 変更
    }),
    new HtmlWebpackPlugin({
      template: './src/index.html',
    }),
  ],
}
```

再度ビルドしてみると`/dist`フォルダの中はこのような構成に変更されます。

```
# ファイル構成

dist/
    + -- stylesheets/
        + -- my.css
    + -- index.html
    + -- main.js
```

### .jsファイルの名前を変える

同じようにJavascriptファイルの名前も変更していみます。
こちらも`filename`を指定することで変更可能です。

```js
module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: 'javascripts/my.js', // 変更
  },
  // ...
}
```

ビルドすると、下記のようなファイル構成になるはずです。
出力されたHTMLの記述も合わせて変更されます。

```
dist/
    + -- javascripts/
        + -- my.js
    + -- stylesheets/
        + -- my.css
    + -- index.html
```

ウェブサイトの標準的なファイル構成が実現できました。

### ファイル構成の改善

`dist`フォルダの中身は理想的な形になりましたが、`/src`の構成を改善できそうです。
現状では以下のようになっており、`dist`と`src`に一貫性がありません。

```
dist/
    + -- javascripts/
        + -- my.js
    + -- stylesheets/
        + -- my.css
    + -- index.html

src/
    + -- modules/
        + -- my.css
        + -- my.js
    + -- index.html
    + -- index.js
```

今までは`src/modules`にファイルを入れてきました。
WebpackではすべてJavascriptのモジュールとして扱われるということを強調するためです。

これまでのセクションでモジュールの概念を理解できたかと思います。
ここでは、最終的な出力結果`dist`フォルダの構成に近づけることによって、より直感的なファイル構成にしたいと思います。

```
dist/
    + -- javascripts/
        + -- my.js
    + -- stylesheets/
        + -- my.css
    + -- index.html

src/
    + -- javascripts/
        + -- index.js
        + -- my.js
    + -- stylesheets/
        + -- my.css
    + -- templates/
      + -- index.html
```

ビルドするともちろんエラーになります。

```shell
# ERROR in Entry module not found: Error: Can't resolve...
```

ファイルを移動しましたので、コードも修正しましょう。

```js
// webpack.config.js

module.exports = {
  entry: {
    main: './src/javascripts/index.js', // パスを変更
  },
  // ...
  plugins: [
    new MiniCssExtractPlugin({
      filename: './stylesheets/main.css',
    }),
    new HtmlWebpackPlugin({
      template: './src/templates/index.html', // パスを変更
    }),
  ],
};
```

```js
// src/javascripts/index.js

import my from './my';            // 変更
import '../stylesheets/my.css';   // 変更

console.log('This is index.js');
my();
```

これでビルドが通るはずです。

練習として `my.css` `index.js`を使用してきましたが、
ビルド後のファイルと合わせておいた方がより直感的ですので`main`というワードを使用したいと思います。


```
src/
    + -- javascripts/
        + -- main.js    # 変更
        + -- my.js
    + -- stylesheets/
        + -- main.css   # 変更
    + -- templates/
      + -- index.html
```

```js
// webpack.config.js

module.exports = {
  entry: {
    main: './src/javascripts/main.js',    // 変更
  },
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: 'javascripts/main.js', 　   // 変更
  },
  // ...
  plugins: [
    new MiniCssExtractPlugin({
      filename: './stylesheets/main.css', // 変更
    }),
    new HtmlWebpackPlugin({
      template: './src/templates/index.html',
    }),
  ],
}
```

ビルドすると`dist`は下記ようになり、`src`フォルダの構成と関連性が強まりました。

```
dist/
    + -- javascripts/
        + -- main.js
    + -- stylesheets/
        + -- main.css
    + -- index.html
src/
    + -- javascripts/
        + -- main.js
        + -- my.js
    + -- stylesheets/
        + -- main.css
    + -- templates/
      + -- index.html
```

このセクションで行ったようなファイル構成の変更は必須ではありませんが、プロジェクトが小さい段階から整理整頓に気を使っておくと、メンテナンス性が向上して後の作業が効率化します。特に、チームで作業したりクライアントに納品したりする場合は、他人が理解する時間を節約することができますので、関係者全員のメリットになります。
