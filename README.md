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










# Section 5 

このブランチの前に行ったこと
--------------------------------

- `clean-webpack-plugin`で`dist`フォルダを空にする
- ファイル構成を改善してメンテナンス性を向上させる

　　
　　

このブランチで行ったこと
--------------------------------

これまで、HTML / CSS / Javascript を扱ってきましたが、今回は画像を扱う方法を解説します。

### 画像の追加

アイコン画像とサムネイル画像を追加したいと思います。

```
src/
    + -- images/
        + -- icon.png
        + -- thumbnail.jpg
    + -- javascripts/
    + -- stylesheets/
    + -- templates/
```

### 画像の配置

コンテンツを追加するには`templates/`を編集することを思い出してください。

```html
<body>
  <h1>Hello World!</h1>
  <!-- 追加 -->
  <img src="../images/icon.png" />
</body>
```

```shell
% npx webpack --mode=development
```

```
./stylesheets/main.css   30 bytes    main  [emitted]  main
            index.html  434 bytes          [emitted]
   javascripts/main.js   5.28 KiB    main  [emitted]  main
```

ビルドは成功しましたが、画像が見当たりません。
`dist`を確認しても`icon.png`はありませんので、ブラウザで開いてみても読み込みエラーになります。

```
GET file:///Users/ss/Dev/webpack_course/images/icon.png net::ERR_FILE_NOT_FOUND
```

### 画像を読み込む

画像を読み込む方法を変更しました。
これは`webpack-html-plugin`が[lodash](https://lodash.com/docs/4.17.15#template)というライブラリに依存しているためで、少し面倒な指定方法になりますが、後で改善していきます。

```html
<!-- src/template/index.html -->

<!-- 削除 -->
<!-- img src="../images/icon.png" / -->

<!-- 追加 -->
<img src="<%= require('../images/icon.png') %>" />
```

この状態でビルドするとエラーになります。
`.png`というファイルを扱う方法をWebpackが知らない状態です。


### url-loader

- [url-loader](https://github.com/webpack-contrib/url-loader) を使って画像を読み込んでみます。

```shell
% npm view url-loader
# latest: 3.0.0

% npm install --save-dev url-loader@3.0.0
```

moduleの設定にruleを追加しましょう。

```js
// webpack.config.js

  // ...

  module: {
    rules: [
      {
        test: /\.css/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader,
          },
          {
            loader: 'css-loader',
          }
        ],
      },
      // 追加
      {
        test: /\.png/,
        use: [
          {
            loader: 'url-loader',
            options: {
              esModule: false,
            },
          },
        ],
      },
    ],
  },

// ...

```

```shell
% npx webpack --mode=development

    Entrypoint undefined = index.html
    [./node_modules/html-webpack-plugin/lib/loader.js!./src/templates/index.html] 577 bytes {0} [built]
    [./node_modules/webpack/buildin/global.js] (webpack)/buildin/global.js 472 bytes {0} [built]
    [./node_modules/webpack/buildin/module.js] (webpack)/buildin/module.js 497 bytes {0} [built]
    [./src/images/icon.png] 13 KiB {0} [built] # icon.pngのログが出力されている


# ブラウザで確認してみてください
% open -a "Google Chrome" dist/index.html
```

```html
<!-- dist/index.html -->

<body>
  <h1>Hello World!</h1>
  <img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAFAAAABQCAYAAACOEfKtAAAAAXNSR0IArs4c6QAAAIRlWElmTU0AKgAAAAgABQESAAMAAAABAAEAAAEaAAUAAAABAAAASgEbAAUAAAABAAAAUgEoAAMAAAABAAIAAIdpAAQAAAABAAAAWgAAAAAAAABIAAAAAQAAAEgAAAABAAOgAQADAAAAAQABAACgAgAEAAAAAQAAAFCgAwAEAAAAAQAAAFAAAAAAwtoCpA//gpXEiDLWPBhEP8+wEKUUiAlzdnpubTi..........">
</body>
```

出力されたHTMLを確認すると、`img`タグが挿入されています。
そして`src`属性には何やらものすごく長い文字列が設定されています。

ブラウザを見ると想像できる通り、実はこの長い文字列が画像です。
`url-loader`が画像を文字列に変換し、`webpack-html-plugin`によってその文字列が`img`タグに反映されました。
なかなか見慣れないかもしれませんが、画像はこのように指定することもできるのです。

### file-loader

画像を文字列に変換して読み込みましたが、あまり標準的な設定方法とは言えません。
[file-loader](https://github.com/webpack-contrib/file-loader)を使ってファイルを配置する”普通”の設定をしていきます。

```shell
% npm view file-loader
# latest: 5.0.2

% npm install --save-dev file-loader@5.0.2
```

```js
// webpack.config.js

  // ...

  module: {
    rules: [
      {
        test: /\.css/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader,
          },
          {
            loader: 'css-loader',
          }
        ],
      },
      {
        test: /\.png/,
        use: [
          {
            loader: 'file-loader', // 変更
            options: {
              esModule: false,
            },
          },
        ],
      },
    ],
  },

// ...

```

```shell
% npx webpack --mode=development

              ./stylesheets/main.css   30 bytes    main  [emitted]  main
5de78d047f515312fd12d11f42bf86e9.png   9.75 KiB          [emitted]
                          index.html  452 bytes          [emitted]
                 javascripts/main.js   5.28 KiB    main  [emitted]  main
```

`5de78d047f515312fd12d11f42bf86e9.png`という見慣れないファイル名がログに出力されています。
ブラウザで見た目を確認しましょう。

```shell
% open -a "Google Chrome" dist/index.html
```

ちゃんと画像が表示されています。
HTMLを確認します。

```shell
% code dist/index.html
```

画像のファイル名がこのように変更されています。
そして`dist/5de78d047f515312fd12d11f42bf86e9.png`というファイルも存在していることが分かります。

```html
<!-- dist/index.html -->

<h1>Hello World!</h1>
<img src="5de78d047f515312fd12d11f42bf86e9.png" />
```

`icon.png`というファイルがWebpackによってモジュールとして読み込まれ、`5de78d047f515312fd12d11f42bf86e9.png`という新しいファイルとして出力されました。これでは`dist`フォルダが散らかってしまいますので、出力する場所と名前をコントロールしたいと思います。

```js
// webpack.config.js

  // ...

      {
        test: /\.png/,
        use: [
          {
            loader: 'file-loader',
            options: {
              esModule: false,
              // 追加
              name: 'images/icon.png',
            },
          },
        ],
      },

// ...
```

再度ビルドしてみると、下のようなファイル構成になったかと思います。

```
dist/
    + -- images/
        + -- icon.png  <- 同じ名前で保存された
    + -- javascripts/
        + -- main.js
    + -- stylesheets/
        + -- main.css
    + -- index.html
```

単にコピーされたように見えますが、一度Webpackにモジュールとして扱われ、新しく書き出された画像ファイルです。
次は複数の画像を配置したいと思います。

```html
<!-- src/templates/index.html -->

<img src="<%= require('../images/icon.png') %>" />
<!-- 追加 -->
<img src="<%= require('../images/thumbnail.jpg') %>" />
```


```shell
% npx webpack --mode=development

ERROR in ./src/images/thumbnail.jpg 1:0
    Module parse failed: Unexpected character '�' (1:0)
    You may need an appropriate loader to handle this file type, currently no loaders are configured to process this file.
```

ビルドコマンドを実行するとエラーになりました。
`./src/images/thumbnail.jpg`というファイルをどう取り扱ったら良いのか分からないというエラーです。
Webpackに`.jpg`を扱えるよう下記のように設定を変更します。`filename`ではなく`name`なのに注意してください。

```js
// webpack.config.js

  // ...

      {
        test: /\.png|\.jpg/,
        use: [
          {
            loader: 'file-loader',
            options: {
              esModule: false,
              // 追加
              name: 'images/icon.png',
            },
          },
        ],
      },

// ...
```

```shell
% npx webpack --mode=development

WARNING in Conflict: Multiple assets emit different content to the same filename images/icon.png
```

ビルドは成功したように見えますが、このような警告が出ました。
複数のファイルが`images/icon.png`という名前で書き出しを行っているというエラーです。
`dist/images`を見てみると分かりますが、`icon.png`の1ファイルしか存在しません。2つの画像を読み込んだはずなのに、どちらも`icon.png`という名前で保存されてしまい、どちらかが上書きされている状態です。

設定ファイルを次のように修正します。


```js
// webpack.config.js

  // ...

      {
        test: /\.png|.jpg/,
        use: [
          {
            loader: 'file-loader',
            options: {
              esModule: false,
              // 変更
              name: 'images/[name].[ext]',
            },
          },
        ],
      },

// ...
```

`[name].[ext]`という特殊な書き方が出てきました。
`[name]`はオリジナルファイルのファイル名、`[ext]`はオリジナルファイルの拡張子という意味になります。
再度ビルドしてみると、`dist/images`フォルダには`src`フォルダの画像と同名の画像ファイルが出力されるかと思います。


```shell
% open -a "Google Chrome" dist/index.html
```

ブラウザでも2つの画像が表示されていることを確認してください。


-------------------


## 5.x での新しい書き方

Webpack 5 では [Asset Modules](https://webpack.js.org/guides/asset-modules/) が導入されました。
4.x では url-loader や file-loader を使って画像ファイルをハンドリングしていましたが、それらがデフォルトでサポートされています。

`type: 'asset/resource'` を追加するだけで使用できます。
注意点は `filename` です。

file-loader では `[ext]` にはピリオドが含まれていませんでしたが、Asset Modules では含まれるようです。
ですので、ファイル名は `filename: 'images/[name][ext]'` となります。


```js
// webpack.config.js

  // ...

      {
        test: /\.png|.jpg/,
        // 追加
        type: 'asset/resource',
        generator: {
          filename: 'images/[name][ext]',
        },
        // 削除
        // use: [
        //   {
        //     loader: 'file-loader',
        //     options: {
        //       esModule: false,
        //       name: 'images/[name].[ext]',
        //     },
        //   },
        // ],
      },

// ...

```

ビルドが同様の結果になることを確認してください。

```bash
% npx webpack --mode=development

assets by path images/ 274 KiB
  asset images/thumbnail.jpg 264 KiB [emitted] [from: src/images/thumbnail.jpg]
  asset images/icon.png 9.75 KiB [emitted] [from: src/images/icon.png]
```

不要なライブラリを削除します。

```
% npm uninstall file-loader url-loader
```










# Section 6

このブランチの前に行ったこと
--------------------------------

- 画像をテンプレートに設置しました
- `url-loader`を使って画像を文字列として読み込みました
- `file-loader`を使って画像を任意の場所に出力しました
- Webpack 5 の新機能、Asset Module で `file-loader` を置き換えた

　　
　　

このブランチで行ったこと
--------------------------------

これまで、HTMLの書き出し、CSS/JS/画像の読み込み等を解説してきましたが、`src`の中身が`dist`に出力されるだけであり、メリットが明確ではありませんでした。今回は [pug](https://pugjs.org/api/getting-started.html) というパッケージを利用して、HTMLのマークアップを一気に効率化したいと思います。

```shell
% npm view pug-html-loader
# latest: 1.1.5

% npm view html-loader
# latest: 1.3.2

% npm install --save-dev pug-html-loader@1.1.5
% npm install --save-dev html-loader@1.3.2
```

```js
// webpack.config.js

  // ...
  module: {
    rules: [
      // 追加
      {
        test: /\.pug/,
        use: [
          {
            loader: 'html-loader',
          },
          {
            loader: 'pug-html-loader',
          },
        ],
      },
    ],
  },
  // ...
```

`.pug`ファイルを読み込んで`html-webpack-plugin`で利用するには2つのローダーを使用します。
まず`pug-html-loader`で`.pug`のファイルをHTMLに変換、そのHTMLを`html-loader`に受け渡しています。
`pug-html-loader`が2番目に設定されていますが、ローダーは下から順に適応されていくのがWebpackのルールです。

### `.pug`ファイルを作成

```shell
% code src/templates/index.pug
```

Pugはインデントを利用してDOMの階層構造を表現していきます。
HTMLの閉じるタグを書かなくて良いので、タイプする文字数と記述ミスが減り、効率化が可能です。

`img`の`src`属性は`html-loader`によってピックアップされ、その画像は`file-loader`によって処理されますので、`require()`の記述も削除することができます。

```pug
doctype html

html(lang="en")
  head
    meta(charset="UTF-8")
    meta(name="viewport" content="width=device-width, initial-scale=1.0")
    title Template Pug
  body
    h1 Hello Pug!
    img(src="../images/icon.png")
    img(src="../images/thumbnail.jpg")
```

### ビルドして確認

```shell
% npx webpack --mode=development

    Entrypoint undefined = index.html
    [./node_modules/html-webpack-plugin/lib/loader.js!./src/templates/index.pug] 337 bytes {0} [built]
    # index.pugが使用されているログが確認できます。
```

ビルドすると`dist/index.html`が作成されていますので、ブラウザでも今までと同じ表示が確認できます。
しかしコードは今までとは少し異なっていますので確認してみましょう。

```shell
% code dist/index.html
```

```html
<!-- dist/index.html -->

<!DOCTYPE html><html lang="en"><head><meta charset="UTF-8"><meta name="viewport" content="width=device-width, initial-scale=1.0"><title>Template Pug</title><link href="./stylesheets/main.css" rel="stylesheet"></head><body><h1>Hello Pug!</h1><img src="images/icon.png"><img src="images/thumbnail.jpg"><script type="text/javascript" src="javascripts/main.js"></script></body></html>
```

改行がすべて削除されています。
改行がなくなることによって、ファイルサイズが若干軽くなっています。バンドルサイズが軽量化するので悪くはないのですが、`dist`フォルダを納品する場合などには改行があった方が良い場合もあります。`pug-html-loader`のオプションを利用すると改行を復活させることが可能です。


```js
// webpack.config.js

  // ...
  module: {
    rules: [
      {
        test: /\.pug/,
        use: [
          {
            loader: 'html-loader',
          },
          {
            loader: 'pug-html-loader',
            // 追加
            options: {
              pretty: true,
            },
          },
        ],
      },
    ],
  },
  // ...
```

まったく同じではありませんが、人間が読みやすい形にはなりました。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Template Pug</title>
  <link href="./stylesheets/main.css" rel="stylesheet"></head>
  <body>
    <h1>Hello Pug!</h1><img src="images/icon.png"><img src="images/thumbnail.jpg">
  <script type="text/javascript" src="javascripts/main.js"></script></body>
</html>
```

- Pugの`pretty`オプション
  - https://pugjs.org/api/reference.html#options
  - 非推奨となっていますので将来削除された場合、prettier等の代替方法を選択しましょう。

### 複数のページを作成する

1ページだけで完結するウェブサイトもあるかもしれませんが、多くの場合、複数ページが必要です。
Pugファイルを追加して複数ページを出力するように設定していきます。

```
% code src/templates/access.pug
```

```pug
// src/templates/access.pug

doctype html

html(lang="en")
  head
    meta(charset="UTF-8")
    meta(name="viewport" content="width=device-width, initial-scale=1.0")
    title Access
  body
    h1 Access Page!
```

```js
// webpack.config.js

  // ...

  plugins: [
    new MiniCssExtractPlugin({
      filename: './stylesheets/main.css',
    }),
    new HtmlWebpackPlugin({
      template: './src/templates/index.pug',
      filename: 'index.html',
    }),
    // 追加
    new HtmlWebpackPlugin({
      template: './src/templates/access.pug',
      filename: 'access.html',
    }),
    new CleanWebpackPlugin(),
  ],

  // ...
```

`HtmlWebpackPlugin`を1つ追加するのですが、同じファイル名で上書きしてしまわないように、`filename`を指定します。

```shell
% npx webpack --mode=development
```

ビルドすると`dist/access.html`が生成されたかと思います。

### 共通のテンプレートを作成する

効率化の大きなポイントです。
複数のHTMLファイルで利用される小さなテンプレートを作成し、表示したい箇所に読み込んでいきましょう。
ページ数が多くなれば多くなるほどメリットが大きくなります。


```
% code src/templates/_menu.pug
```

```pug
// src/templates/_menu.pug

div
  a(href="./index.html") index.html
  a(href="./access.html") access.html
```

`_`をプリフィックスとして付けることによって、「これは他の手プレートから読み込まれるファイルである」ことを分かりやすくしています。必須ではないですが、ファイル構成がより明確になるかと思いますのでお勧めです。他にも`partials`のようのフォルダを作成してその中に保存するのも良いかと思います。

#### 部分テンプレートを読み込む

```pug
doctype html

html(lang="en")
  head
    meta(charset="UTF-8")
    meta(name="viewport" content="width=device-width, initial-scale=1.0")
    title Template Pug
  body
    // 追加
    include _menu.pug
    h1 Hello Pug!
    img(src="../images/icon.png")
    img(src="../images/thumbnail.jpg")
```

`access.html`をクリックすると、ページに遷移します。`access.html`にもメニューを配置しましょう。

```pug
// src/templates/access.pug

doctype html

html(lang="en")
  head
    meta(charset="UTF-8")
    meta(name="viewport" content="width=device-width, initial-scale=1.0")
    title Access
  body
    // 追加
    include _menu.pug
    h1 Access Page!
```

ビルドすると`access.html`にもメニューが表示され、ページを行き来できるようになりました。
今後、メニューに新しい要素を追加するには`_menu.pug`を更新するだけでOKです。その上でビルドすると、`_menu.pug`を読み込んでいる全てのテンプレートでHTMLがアップデートされるので、メンテナンスが非常に楽になります。

### Pugテンプレートの拡張

部分テンプレートを作成する方法は非常に強力ですが、ページのある部分を読み込む機能しか持ち合わせていません。すべてのページでメニューを表示したい場合は、それぞれのPugファイルで`include _menu.pug`と記述する必要があります。
部分テンプレートが増えれば増えるほど`include`するファイルが増えてしまいますので、ミスの原因にもなりかねません。

これを更に効率化するために、Pugには拡張という機能があります。
拡張を利用すると、ページの大枠を共有することができるので、`include`する手間さえ省くことができます。

```
% code src/templates/_layout.pug
```

```pug
// src/templates/_layout.pug

doctype html

html(lang="en")
  head
    meta(charset="UTF-8")
    meta(name="viewport" content="width=device-width, initial-scale=1.0")
    title Template Pug
  body
    // メニューをここで読み込んでいます
    include _menu.pug
    // ここを各ページのコンテンツで置き換えます
    block content
```

この`_layout.pug`というファイルを`index.pug`と`access.pug`で拡張します。

```pug
// src/templates/index.pug
extends _layout.pug
block content
  h1 Hello Pug!
  img(src="../images/icon.png")
  img(src="../images/thumbnail.jpg")
```

```pug
// src/templates/access.pug
extends _layout.pug
block content
  h1 Access Page!
```

ファイルが、随分コンパクトになりました。
ポイントは`extends`と`block`です。

- `extends`：ベースとなるテンプレートを指定します。
- `block content`：この中身が`_layout.pug`の`block content`と入れ替わります。

ビルドして問題なくHTMLが表示されることを確認してください。

#### 複数のブロック

`block content`と記述して、拡張したテンプレートで内容を置き換えましたが、`block`は複数設定することもできます。
次の例は、複数のブロック、そして変数を使う方法です。

変数は`var`で定義することができ、テンプレート内で使用できます。通常のHTMLにはない機能です。
ブロックの中で変数を定義することもできるので、各ページにて、下記のようにブロック内の変数を置き換えることができます。

```pug
// src/templates/_layout.pug

doctype html

block locals
  - var title = 'My Website'

html(lang="en")
  head
    meta(charset="UTF-8")
    meta(name="viewport" content="width=device-width, initial-scale=1.0")
    title #{title}
  body
    include _menu.pug
    h1 #{title}
    block content
```

```pug
// src/templates/index.pug

extends _layout.pug
block locals
  - var title = 'Hello Pug!'
block content
  img(src="../images/icon.png")
  img(src="../images/thumbnail.jpg")
```

```pug
// src/templates/access.pug

extends _layout.pug
block locals
  - var title = 'Access Page!';
block content
```

変数を置き換えることによって、HTMLを個別に書くことなくタイトルタグを出力しました。
また、`head`タグ内の`title`も同じ文字列を使用しようしています。
このように、内容が全く同じ文字列等は変数を利用することで、タイプミスを防ぐことができます。

　　

このセクションではPugを利用することで、HTMLを効率的に作成する方法を説明しました。
数十ページあるHTMLの共通要素を同期させるのは、こういったツールがないとなかなか大変ですし、間違いがないかチェックする工数もかかってしまいます。
共通化できるものはできるだけ共通化すると、作業効率が劇的に改善されるかと思います。

Pugには他にも色々な機能があります。ぜひ[公式ドキュメント](https://pugjs.org/api/getting-started.html)も読んでみてください。










# Section 7

このブランチの前に行ったこと
--------------------------------

- Pugの導入
- includeを使った共通化
- extendsを使った効率化
- 変数をテンプレートで使用する方法

　　

このブランチで行ったこと
--------------------------------

### ローカルサーバー

現状だとHTMLファイルを直接ブラウザが読み込んでいます。
ブラウザのアドレスバーを確認すると、`file:///....`のようなパスが指定されており、通常のサイトのURLのような形ではありません。

問題ないように見えますが、ウェブサイトは最終的にウェブサーバーが`http`プロトコルを使用して配信されます。ローカル環境でも同じように`http`を使用するにはローカル環境でもサーバーを立ち上げる必要があります。

ローカルサーバーのメリット:

- リリース後の問題が少ない
- ライブリロードが便利
- ルートパス`/`が使える

### ローカルサーバーを起動する

Webpackでは、ローカルサーバーをとても簡単に立ち上げることができます。

```shell
% npm view webpack-dev-server
# latest: 3.11.2

% npm install --savve-dev webpack-dev-server@3.11.2
```

```shell
# Webpack 5 では serve コマンドを使います
% npx webpack serve --mode=development

ℹ ｢wds｣: Project is running at http://localhost:8080/
ℹ ｢wds｣: webpack output is served from /
ℹ ｢wds｣: Content not from webpack is served from /Users/ss/Dev/webpack_course
```

たったこれだけです。
`http://localhost:8080/`にブラウザでアクセスすると、先程まで同じコンテンツが表示されました。
本番と同じhttpプロトコルを使用しますので、この部分で問題が発生する可能性が低くなりました。
リリース間近は何かと慌ただしいと思いますので、余計な心配事は取り除いておきましょう。

### 編集が反映される

```shell
% code src/templates/index.pug
```

```pug
// src/templates/index.pug

extends _layout.pug
block locals
  - var title = 'Hello Pug!'
block content
  // 追加
  h2 Test
  img(src="../images/icon.png")
  img(src="../images/thumbnail.jpg")
```

`src/templates/index.pug`を編集して保存すると、自動的にビルドが実行され、ブラウザもリロードされます。
2番目のメリット`ライブリロード`を利用することができました。

### ルートパスが利用できる

これを説明するまえに、HTMLをサブディレクトリにも作成してみましょう。
ページ構成に階層構造を作ります。

#### 現在の構造

```
- /
- /index.html
- /access.html
```

#### 変更後の構造

```
- /
- index.html
- access.html
- members/
  - taro.html
```

このように、`members`フォルダの中にさらにHTMLを配置します。

```shell
% code src/templates/members/taro.pug
```

```pug
extends ../_layout.pug
block locals
  - var title = 'Taro'
block content
  p メンバー紹介ページです
```

```js
// webpack.config.js
  // ...
  plugins: [

    // 追加
    new HtmlWebpackPlugin({
      template: './src/templates/members/taro.pug',
      filename: 'members/taro.html',
    }),

  ],
  // ...
```

### HTMLの階層化

複数のHTMLを作成できるようになりましたが、サブディレクトリに格納する場合はどうすれば良いでしょうか。
下記のような構成にしてみたいと思います。

```
src/
    + -- javascripts/
    + -- stylesheets/
    + -- templates/
      + -- access.pug
      + -- index.pug
      + -- members/
          + -- taro.pug
```

```pug
// src/templates/members/taro.pug

extends ../_layout.pug
block locals
  - var title = 'Taro'
block content
  p メンバー紹介ページです
```

```js
// webpack.config.js
  // ...
  plugins: [
    // 追加
    new HtmlWebpackPlugin({
      template: './src/templates/members/taro.pug',
      filename: 'members/taro.html',
    }),
  ]
  // ...
```

```shell
# control + c で webpack-dev-server を終了させる

# 再度起動
% npx webpack serve --mode=development

                 Asset       Size  Chunks             Chunk Names
./stylesheets/main.css   30 bytes    main  [emitted]  main
           access.html  449 bytes          [emitted]
       images/icon.png   9.75 KiB          [emitted]
  images/thumbnail.jpg    264 KiB          [emitted]
       images/user.png     24 KiB          [emitted]
            index.html  504 bytes          [emitted]
   javascripts/main.js    362 KiB    main  [emitted]  main
     members/taro.html  489 bytes          [emitted]
```

`dist`フォルダに注目してください。正しく設定したはずなのに`members/taro.html`は作成されていません。

#### 保存されないファイル

実は`webpack-dev-server`のビルドはファイルとして出力されず、すべてメモリ上に保存されますので、ファイルとして見ることができません。ファイルを確認するためには、今まで通り`npx webpack --mode=development`を実行すればOKです。

#### HTMLの確認

ファイルとしては存在しないのですが、メモリ上に保存されローカルサーバーによって配信されている状態です。ブラウザでアクセスすると確認することができます。

```shell
% open http://localhost:8080/members/taro.html
```

#### リンクの不具合(ローカルサーバー)

階層を作ってHTMLを配置することはできましたが、問題があります。
`index.html / access.html`というリンクをクリックすると下記のようなエラーになります。

```
Cannot GET /members/index.html
```

`src/tempates/_menu.pug`を確認してみましょう。

```pug
// src/tempates/_menu.pug

div
  a(href="./index.html") index.html
  a(href="./access.html") access.html
```

`href`属性を見ると`./index.html`となっていますね。「現在のディレクトリの`index.html`」という意味です。
結果として、`/members/index.html`が指定され、そんなファイルは存在しないのでエラーとなっています。

#### リンクの不具合(ファイルシステム)

同じように、ローカルサーバーを立ち上げなくてもエラーになります。

```shell
% npx webpack --mode=development

% open -a "Google Chrome" dist/members/taro.html
```

`index.html`のリンクをクリックすると下記のエラーになります。
理由は同じで、`/members`フォルダには`index.html`が存在しないからです。

```
Your file was not found
```


#### リンクを修正する

修正するにはルートから見たパスを指定しましょう。ついでにメンバーページへのリンクも追加します。

```pug
// src/tempates/_menu.pug

div
  a(href="/index.html") index.html
  a(href="/access.html") access.html
  a(href="/members/taro.html") members/taro.html
```

ローカルサーバーを起動して、3つのリンクが正常に動作していることを確認してください。

```shell
% npx webpack serve --mode=development
```

ルートから見たパスを指定してた上で、ファイルシステムを使ってテストしてもうまくいきません。
`file:///index.html`というファイルは存在しないので当然ですね。

ルートパスから指定することで、階層が1つ下のHTMLページからもリンクを貼ることができました。
この`taro.html`がさらに下の階層に移動したとしても問題ありません。リンクは常にルート`/`から指定されているからです。
これがローカルサーバーを使う3番目のメリット「ルートパス」になります。










# Section 8

このブランチの前に行ったこと
--------------------------------

- ローカルサーバーの起動
- HTMLページの階層化
- ルートパスの使用

　　
　　

このブランチで行ったこと
--------------------------------

### CSSの効率化

HTMLについてはPugを利用して効率化できました。
（少なくとも効率化する準備はできました）

つくるページが多ければ多い程、メリットが大きくなりますので、
10ページ以上あるようなウェブサイトの場合はぜひ最初から採用を検討してください。

今回は、[Sass](https://sass-lang.com/)をつかって、CSSを効率的にビルドしていきたいと思います。
~~`sass-loader`を使用しますが、`node-sass`に依存していますので、どちらもインストールしていきます。~~
**`node-sass`は非推奨となりました。`sass`を利用してください。**

- [sass-loader](https://github.com/webpack-contrib/sass-loader)
- ~~[node-sass](https://github.com/sass/node-sass)~~（非推奨）
- [sass](https://www.npmjs.com/package/sass)

```shell
% npm install --save-dev sass@1.49.9
% npm install --save-dev sass-loader@12.4.0
```

```js
// webpack.config.js

      // ...

      {
        test: /\.(css|scss|sass)$/, // 変更
        use: [
          {
            loader: MiniCssExtractPlugin.loader,
          },
          {
            loader: 'css-loader',
          },
          {
            loader: 'sass-loader', // 追加
          },
        ],
      },

      // ...
```

#### ファイル名を変更します

```shell
% mv src/stylesheets/main.css src/stylesheets/main.scss
```

```js
// src/javascripts/main.js

import my from './my';
import '../stylesheets/main.scss'; // 変更

console.log('This is index.js');
my();
```

#### ビルド

```shell
% npx webpack --mode=development
```

`dist/stylesheets/main.css`が生成されます。
`.scss`が`.css`として出力された結果です。

### Sassの拡張子

Sassは2つの拡張子を使用することができます。
先程のWebpackの変更でも対応しています。

```js
test: /\.(css|scss|sass)$/,
```

この正規表現は `.css` `.scss` `.sass`のどれかにマッチします。

### Sassの記述方法

`.scss`と`.sass`では、それぞれCSS記述方法が違います。

#### .scss

```scss
body {
  .purple {
    color: purple;
  }
}
```

#### .sass

```sass
body
  .purple
    color: purple
```

`.scss`は通常の`.css`ファイルに近く、`.sass`はインデントを利用して階層構造を表現するPugに近い書き方です。
どちらも最終的なアウトプットは同じですが、`.scss`をお勧めします。

[公式ページ](http://thesassway.com/editorial/sass-vs-scss-which-syntax-is-better)も`.scss`を推薦しています。

理由は色々ありますが、純粋なCSSとほとんど書き方が同じという点が大きいかと思います。
このコースでは`.scss`を採用していきます。

### Sassのメリット

- ネスト
- 変数
- ~~@import~~（@use に変更となりました）
- @extend
- @mixin
- etc...

簡単ですがSassのメリットを紹介します。

#### ネスト

階層構造がより明確になります。

```scss
.content {
  .title {
    font-size: 24px;
  }
}
```

```css
.content .title {
  font-size: 24px; }
```

#### 変数

```scss
$text-color: orange;
.content {
  .title {
    font-size: 24px;
    color: $text-color;
  }
}
```

```css
.content .title {
  font-size: 24px;
  color: orange; }
```

#### @use

```scss
@use './footer';
$text-color: orange;
.content {
  .title {
    font-size: 24px;
    color: $text-color;
  }
}
```

```scss
// footer.scss
.footer {
  background-color: #ddd;
}
```

```css
.footer {
  background-color: #ddd; }

.content .title {
  font-size: 24px;
  color: orange; }
```

#### @extend

```scss
@use './footer';
$text-color: orange;
.content {
  .title {
    font-size: 24px;
    color: $text-color;
  }
  .title-lg {
    @extend .title;
    font-size: 32px;
  }
}
```

```css
.footer {
  background-color: #ddd; }

.content .title, .content .title-lg {
  font-size: 24px;
  color: orange; }

.content .title-lg {
  font-size: 32px; }
```

#### @mixin

```scss
@use './footer';

@mixin set-margin($direction, $value) {
  margin-#{$direction}: $value;
}

$text-color: orange;
.content {
  .title {
    font-size: 24px;
    color: $text-color;
  }
  .title-lg {
    @extend .title;
    font-size: 32px;
  }
  @include set-margin(right, 16px);
}
```

```css
.footer {
  background-color: #ddd; }

.content {
  margin-right: 16px; }
  .content .title, .content .title-lg {
    font-size: 24px;
    color: orange; }
  .content .title-lg {
    font-size: 32px; }
```

その他、いろいろ便利な機能がSassにはあります。
ぜひ[公式ドキュメント](https://sass-lang.com/)を読んでみてください。

### ベンダープリフィクス

古いブラウザをサポートする場合にはCSSにベンダープリフィクスを使用する必要があるかと思います。
ベンダープリフィクスを手動で書いていくのは面倒な作業ですが、`Autoprefixer`を使用すれば自動化することができます。

`Autoprefixer`は便利ですが、最近ではベンダープリフィクスなしで多くのプロパティが利用可能になっており、対象ブラウザによっては不要なことも多いです。

例えば`flexbox`は、このようなカバレッジになっています。
https://caniuse.com/#feat=flexbox

今後もベンダープリフィクスは不要になっていくと思いますので、このコースでは`Autoprefixer`を導入していきませんが、古いブラウザ対応が必要な場合は、`postcss-loader`と`autoprefixer`を利用することで自動化することができます。必要に応じて調べてみてください。

- [postcss-loader](https://github.com/postcss/postcss-loader)
- [autoprefixer](https://github.com/postcss/autoprefixer)










# Section 9

このブランチの前に行ったこと
--------------------------------

- node-sass と sass-loader のインストール
- Sassの基本

　　
　　

このブランチで行ったこと
--------------------------------

### Javascriptの効率化

`my.js`ではアローファンクションを使用しています。

```js
// my.js
export default () => {
  console.log('this is module');
};
```

ビルド後のJSを見ると`(() => {\n  console.log('this is module');`この辺りが該当のコードであると推測できます。

```js
// dist/javascripts/main.js

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony default export */ __webpack_exports__[\"default\"] = (() => {\n  console.log('this is module');\n});\n\n\n//# sourceURL=webpack:///./src/javascripts/my.js?");
```

`() => {}`という記法ですが、ECMAScript 6 (ES6)から導入された関数定義の方法です。
比較的新しい記法（といっても数年前ですが）なので、IE11は対応していません。

参考：https://caniuse.com/#search=arrow%20function

IE11に対応するには、ES5で書く必要があります。

```js
// my.js
export default function() {
  console.log('this is module');
};
```

ビルドすると下のようになります。

```js
// dist/javascripts/main.js

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony default export */ __webpack_exports__[\"default\"] = (function() {\n  console.log('this is module');\n});;\n\n\n//# sourceURL=webpack:///./src/javascripts/my.js?");
```

`(function() {\n  console.log('this is module');` このように変更されました。
これでIE11でも動作するようになりましたが、ES6の便利な記法が使えないのは残念です。

Webpackを利用してES6を使いつつ、ES5に自動的に変換するよう設定していきましょう。

### Babel

- [babel-loader](https://github.com/babel/babel-loader)
- 依存ライブラリ
  - `@babel/core`
  - `@babel/preset-env`


```shell
% npm view babel-loader
# latest: 8.2.2

% npm view @babel/core
# latest: 7.12.10

% npm view @babel/preset-env
# latest: 7.12.11

% npm install --save-dev babel-loader@8.2.2
% npm install --save-dev @babel/core@7.12.10
% npm install --save-dev @babel/preset-env@7.12.11
```

```js
// webpack.config.js

    // ...

    rules: [
      {
        test: /\.js/,
        exclude: /node_modules/,
        use: [
          {
            loader: 'babel-loader',
            options: {
              presets: ['@babel/preset-env'],
            },
          },
        ],
      },
    ]

    // ...
```

`rules`に追加します。
`my.js`はアローファンクションに戻しておきましょう。

```js
// src/javascripts/my.js

export default () => {
  console.log('this is module');
};
```

ビルドすると、アローファンクションがES5の関数定義に変換されると思います。

```shell
% npx webpack --mode=development
```

```js
// dist/javascripts/main.js

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony default export */ __webpack_exports__[\"default\"] = (function () {\n  console.log('this is module');\n});\n\n//# sourceURL=webpack:///./src/javascripts/my.js?");
```

### プリセット

```js
options: {
  presets: ['@babel/preset-env'],
},
```

Babelはプラグインを追加して必要な機能を加えていくのですが、一つずつ追加していくのは面倒です。
そこで、あらかじめ用意されたプラグインのセットが使用する方法が`presets`オプションです。

プリセットには複数の種類がありますが、ここでは[@babel/preset-env](https://babeljs.io/docs/en/next/babel-preset-env.html)を使用しています。
その他は[こちら](https://babeljs.io/docs/en/presets)を参照してください。

### preset-env

今回使用した`preset-env`ですが、オプションを設定してターゲットのブラウザを指定することができます。

例えば、`0.25%以上のシェア`があり、`公式サポートが終了していない`ブラウザで動作するようにJavascriptをトランスパイルする設定は下記のようになります。

```js
options: {
  presets: [
    ['@babel/preset-env', { "targets": "> 0.25%, not dead" }],
  ],
},
```

試しに、シェアが30%あるブラウザだけを対象にしてみましょう。
2020年2月の段階だと、Chromeだけが対象になるはずです。

```js
options: {
  presets: [
    ['@babel/preset-env', { "targets": "> 30%, not dead" }],
  ],
},
```

```shell
% npx webpack --mode=development
```

結果を確認するとアローファンクションが復活しています。
最新ブラウザだけを対象にすれば良いので、Babelでの変換が不要になった結果です。

```js
// dist/javascripts/main.js

"use strict";
eval("__webpack_require__.r(__webpack_exports__);\n/* harmony default export */ __webpack_exports__[\"default\"] = (() => {\n  console.log('this is module', newObj);\n});\n\n//# sourceURL=webpack:///./src/javascripts/my.js?");

```

設定を戻しておきましょう。

```js
options: {
  presets: [
    ['@babel/preset-env', { "targets": "> 0.25%, not dead" }],
  ],
},
```

対象ブラウザの設定方法については、[こちら](https://github.com/browserslist/browserslist)を参考にしてください。

### ES6

他にもモダンなJavascriptでは便利が機能が使えますので、ご自身で調べてみてください。

#### Spread syntax

```js
// src/javascripts/my.js

export default () => {
  const obj = { a: 1, b: 2 };
  const newObj = { ...obj, c: 3 };
  console.log('this is module', newObj);
};
```

```js
// dist/javascripts/main.js

(function () {\n  var obj = {\n    a: 1,\n    b: 2\n  };\n\n  var newObj = _objectSpread({}, obj, {\n    c: 3\n  });\n\n  console.log('this is module', newObj);\n});
```

参考サイト：
https://www.taniarascia.com/es6-syntax-and-feature-overview/










# Section 10

このブランチの前に行ったこと
--------------------------------

- `babel`のインストール
- `@babel/preset-env`のインストール
- ECMAScript 6

　　
　　

このブランチで行ったこと
--------------------------------

### ソースマップ

ソースマップはデバッグに便利なツールです。
ビルドされたコードは人間の目では解読が難しいですが、ソースマップを配置することでブラウザでのデバッグが格段に効率化します。

ソースマップの種類はいろいろあるのですが、[詳しくは公式](https://webpack.js.org/configuration/devtool/)を確認してください。

今回は下記のソースマップオプションを使用します。

|devtool|build|build|production|quality|
|---|---|---|---|---|
|source-map|slowest|slowest|yes|original source|

Webpackの設定を修正しましょう。

```js
// webpack.config.js

module.exports = {
  devtool: 'source-map',
  // ...
}
```

これだけで、Javascriptのソースマップが表示できます。

`my.js`で`console.log`していますから、Chromeの開発者ツールを開いて`Console`を開くと、ログが表示されています。
右端にあるファイル名をクリックすることで、ソースマップによってビルド前のコードを確認することができます。

#### ブレイクポイント

これだけでも便利ですが、ブレイクポイントもこのビルド前コードに設定できますので、活用してみてください。

#### CSSのソースマップ

Sassを使用していますので、CSSのビルド前コードをChromeで確認するには、もうひとつオプションを追加します。

```js
// webpack.js.config

{
  test: /\.(css|scss|sass)$/,
  use: [
    {
      loader: MiniCssExtractPlugin.loader,
    },
    {
      loader: 'css-loader',
      options: { sourceMap: true }, // 追加
    },
    {
      loader: 'sass-loader',
    },
  ],
},
```

```shell
% npx webpack serve --mode=development
```

今度はCherome開発者ツールの`Elements`を開き、
同じく右端のファイル名をクリックすると、元の`.scss`ファイルを見ることができます。

### 本番環境要のコード

現在のビルドはこちらのコマンドです。

```shell
% npx webpack --mode=developemnt
```

本番用のコードをビルドするためには、モードを変更します。

```shell
% npx webpack --mode=production
```

ビルドすると、Javascriptがブラウザが解釈しやすいような形に変換されました。
このモードはコマンドで`--mode`を指定することもできますが、Webpackの設定ファイルで指定することができます。

```js
// webpack.config.js

module.exports = {
  mode: 'production', // 追加
  devtool: 'source-map',
  entry: {
    main: './src/javascripts/main.js',
  },
  // ...
```

```shell
% npx webpack
```

`--mode`オプションを指定しなくても本番用のコードが出力できました。

### コマンドの一元管理

```shell
% npx webpack
% npx webpack --mode=development # webpack.config.js を上書きできる
% npx webpack serve --mode=development
```

コマンドが増えてきたので`package.json`にまとめたいと思います。


```js
// webpack.config.js

module.exports = {
  // 変更
  mode: 'development',
  devtool: 'source-map',
  entry: {
    main: './src/javascripts/main.js',
  },
  // ...
```

```
{
  "name": "webpack_test",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    // 削除
    // "test": "echo \"Error: no test specified\" && exit 1"
    // 追加
    "start": "npx webpack serve --mode=development",
    "build": "webpack --mode=production",
    "build:dev": "webpack"
  },
  // ...
}
```

実行するには、下記のようになります。

```shell
% npm start
% npm run build
% npm run build:dev
```











# Section 11

このブランチの前に行ったこと
--------------------------------

- 画像の最適化

　　
　　

このブランチで行ったこと
--------------------------------

### React

- https://reactjs.org/

Reactは[create-react-app](https://github.com/facebook/create-react-app)コマンドで簡単にセットアップが完了しますが、SPA(Single Page Application)を前提としたファイル構成になっています。ファイル構成を自分で決めたい場合や、既存のウェブサイトにReactを導入したい場合などには、Webpackを自分で設定すると柔軟に対応することができます。

今まで説明してきたことの応用ですので、スムーズに導入できるかと思います。

#### @babel/preset-react

Babelのセクションで解説しましたが、presetとはBabelのプラグインリストです。
`@babel/preset-env`はES6/7の便利な機能を含んでいますが、Reactに必要なプラグインが不足しています。
Reactに対応するには`@babel/preset-react`をインストールします。

```shell
% npm view @babel/preset-react
# latest: 7.12.10

% npm install --save-dev @babel/preset-react@7.12.10
```

```js
// webpack.config.js

{
  test: /\.(js|jsx)/, // 変更
  exclude: /node_modules/,
  use: [
    {
      loader: 'babel-loader',
      options: {
        presets: [
          ['@babel/preset-env', { "targets": "> 0.25%, not dead" }],
          // 追加
          '@babel/preset-react',
        ],
      },
    },
  ],
},
```

ビルドしてエラーがないか確認します。

#### react/react-dom

Reactの本体をインストールします。

```shell
% npm view react
# latest: 17.0.1

% npm view react-dom
# latest: 17.0.1

% npm install --save-dev react@17.0.1
% npm install --save-dev react-dom@17.0.1
```

#### Reactコンポーネント

Reactのコンポーネントを作成します。

```jsx
// reactApp.jsx

import ReactDom from 'react-dom';
import * as React from 'react';

const App = (props) => {
  return (
    <div>
      Hello, React App!
    </div>
  );
};

const reactRoot = document.getElementById('react-root');
if (reactRoot) {
  ReactDom.render(<App />, reactRoot);
} else {
  console.log('No root element found');
}
```

```shell
% npm start
```

ブラウザのコンソールに`No root element found`と出ていれば成功です。

#### コンポーネントをマウント

```pug
// src/templates/index.pug

extends _layout.pug
block locals
  - var title = 'Hello Pug!'
block content
  // 追加
  div#react-root
  img(src="../images/icon.png")
  img(src="../images/thumbnail.jpg")
```

これでReactのコンポーネントがマウントできました。
