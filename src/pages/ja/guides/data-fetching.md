---
layout: ~/layouts/MainLayout.astro
title: データフェッチ
description: Astroでfetch APIを使用してリモートのデータを取得する方法を学びましょう。
i18nReady: true
---
`.astro`ファイルはビルド時にリモートのデータを取得し、ページの生成に使います。

## Astroでの`fetch()`

すべての[Astroコンポーネント](/ja/core-concepts/astro-components/)は、そのコンポーネントスクリプトで[グローバルな`fetch()`function](https://developer.mozilla.org/ja/docs/Web/API/fetch)にアクセスし、APIにHTTPリクエストを行えます。このfetchはビルド時に実行され、そのデータは動的なHTMLを生成するためコンポーネントテンプレートで利用可能になります。

💡 Astroコンポーネントスクリプトの内部で、[**top-level await**](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Operators/await#top_level_await)を利用できます。

💡 取得したデータはAstroとフレームワークの両方のコンポーネントにpropsとして渡されます。

```astro
---
// src/components/User.astro
import Contact from '../components/Contact.jsx';
import Location from '../components/Location.astro';

const response = await fetch('https://randomuser.me/api/');
const data = await response.json();
const randomUser = data.results[0]
---
<!-- ビルド時に取得したデータでHTMLがレンダリングされます。 -->
<h1>ユーザー</h1>
<h2>{randomUser.name.first} {randomUser.name.last}</h2>

<!-- ビルド時に取得したデータがpropsとしてコンポーネントに渡されます。 -->
<Contact client:load email={randomUser.email} />
<Location city={randomUser.location.city} />
```

### GraphQLクエリ

Astroはまた、`fetch()`を使用して任意の有効なGraphQLクエリでGraphQLサーバーにクエリを投げられます。

```astro
---
const response = await fetch("https://graphql-weather-api.herokuapp.com",
  {
    method:'POST',
    headers: {'Content-Type':'application/json'},
    body: JSON.stringify({
      query: `
        query getWeather($name:String!) {
            getCityByName(name: $name){
              name
              country
              weather {
                summary {
                    description
                }
              }
            }
          }
        `,
      variables: {
          name: "Tokyo",
      },
    }),
  })

const json = await response.json();
const weather = json.data
---
<h1>天気をビルド時にフェッチします。</h1>
<h2>{weather.getCityByName.name}, {weather.getCityByName.country}</h2>
<p>天気: {weather.getCityByName.weather.summary.description}</p>
```
> 💡 Astroコンポーネントのすべてのデータは、コンポーネントがレンダリングされるときにフェッチされることを忘れないでください。

デプロイされたAstroサイトは、**ビルド時に一度だけ**データをfetchします。開発環境では、コンポーネントの更新時にfetchされたデータが表示されます。クライアントサイドで何度もデータを再取得する必要がある場合は、Astroコンポーネントで[UIフレームワーク](/ja/core-concepts/framework-components/)または[クライアントサイドスクリプト](/ja/core-concepts/astro-components/#クライアントサイドスクリプト)を使用します。

## UIフレームワークでの`fetch()`

`fetch()`関数は、任意の[UI フレームワーク](/ja/core-concepts/framework-components/)からもグローバルに利用できます。

```tsx
// Movies.tsx
import type { FunctionalComponent } from 'preact';
import { h } from 'preact';

const data = await fetch('https://example.com/movies.json').then((response) =>
  response.json()
);

// ビルド時にレンダリングされるコンポーネントはCLIにもログとして出力されます。
// client:*ディレクティブでレンダリングされた場合、ブラウザコンソールにもログが出力されます。
console.log(data);

const Movies: FunctionalComponent = () => {
  // 結果をページに出力する
  return <div>{JSON.stringify(data)}</div>;
};

export default Movies;
```
