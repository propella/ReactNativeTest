# react-native-web + TypeScript のサンプル

これが正しいやり方なのか良くわからないが、動いたのでメモ。目標は

* TypeScript でコードを書く。react-native-typescript-transformer を使う。
* 携帯 Native アプリとして実行する。Expo を使いたくないので create-react-native-app は使わない。
* Web アプリとして実行する。プロジェクトは create-react-app で作成。
    * create-react-app が対応しているので最低限の設定で react-native-web を利用可。

## 普通に react-native のプロジェクトを作る

    brew install node watchman yarn
    npm install -g react-native-cli
    react-native init ReactNativeTest
    cd ReactNativeTest
    react-native run-ios

## React Native を TypeScript で書く。

    yarn add --dev react-native-typescript-transformer typescript @types/react @types/react-native

あとは <https://github.com/ds300/react-native-typescript-transformer> の通りに

* tsconfig.json
* rn-cli.config.js

を作成

コンポーネントの拡張子を変える。

    mv App.js App.tsx

import 文をちょっと変えないとビルド出来ませんでした。

    import * as React from 'react';
    const Component = React.Component;

## React Native のテストを TypeScript で書く

基本 <https://github.com/kulshekhar/ts-jest#react-native> の通りだが、慣れてないと難しい。。。

    yarn add --dev ts-jest @types/jest @types/react-test-renderer

package.json の jest セクションはこんな感じ

```javascript
"jest": {
    "preset": "react-native",
    "transform": {
      "^.+\\.jsx?$": "<rootDir>/node_modules/babel-jest",
      "^.+\\.tsx?$": "ts-jest"
    },
    "testRegex": "(/__tests__/.*|(\\.|/)(test|spec))\\.(jsx?|tsx?)$",
    "moduleFileExtensions": [
      "ts",
      "tsx",
      "js",
      "jsx",
      "json",
      "node"
    ],
    "globals": {
      "ts-jest": {
        "useBabelrc": true
      }
    }
  }
```

サンプルでついてるテストケースも微妙にうごかないので修正

    mv __tests__/app.js __tests__/app.tsx

```typescript
import 'react-native';
import * as React from 'react';
import App from '../src/App';

// Note: test renderer must be required after react-native.
import { create } from 'react-test-renderer';

it('renders correctly', () => {
  create(
    <App />
  );
});
```

## react-native-web と react-script-ts で web 化

    yarn add react-native-web react-dom react-scripts-ts

別途 `npx create-react-app my-app --scripts-version=react-scripts-ts` で作っておいたプロジェクトから足りないファイルを手動で追加してゆく。

```
mkdir public
cp ../my-app/public/index.html public/index.html
cp ../my-app/tsconfig.json tsconfig.json 
cp ../my-app/tslint.json tslint.json
```

src/index.tsx の作成 

```typescript
import App from './App';
import { AppRegistry } from 'react-native';

// register the app
AppRegistry.registerComponent('App', () => App);

AppRegistry.runApplication('App', {
  initialProps: {},
  rootTag: document.getElementById('root')
});
```

    mv App.tsx src/

パスも合わせて修正する。

その他雑多な作業

* `Property 'geolocation' must be of type 'Geolocation'` エラーが出る場合 tsconfig.json に以下追加
    * `"skipLibCheck": true`
* src/App.tsx で styles の定義を前に持ってくる。
* tsconfig.json から `"rootDir": "src"` を削除

起動

    npx react-scripts-ts start

ソースコードを更新すると Web ブラウザと iPhone Simulator 同時に表示が変わる。これは面白い。

## 参考

* [React Native for WebとExpoを組み合わせてピコピコさせてみたよ](https://qiita.com/Nkzn/items/8e31efe0ebafa8038bde)
    * webpack を使った例。やりたい事が大体同じで大変役にたった。こっちの方が柔軟で良いかも。
