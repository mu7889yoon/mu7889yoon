よ〜んです。

今回は、Chrome Prompt APIを使って遊んでみようと思います

## Chrome Prompt APIとは

Chrome 131から利用可能になった、ブラウザ内蔵のAI機能です。Gemini Nanoがローカルで動作するため、APIキー不要・通信不要で推論ができます。

有効化するには以下のフラグを設定する必要があります：
- `chrome://flags/#prompt-api-for-gemini-nano`
- `chrome://flags/#optimization-guide-on-device-model`

さらに `chrome://components` から「Optimization Guide On Device Model」をダウンロードしておく必要があります。

マルチモーダル入力（画像など）を使う場合は Origin Trial への登録も必要です。

## IndexedDB

ブラウザ内蔵のNoSQLデータベースです。今回は画像のBlobデータをそのまま保存するために使っています。

```javascript
const req = indexedDB.open('image-db', 1)
req.onupgradeneeded = () => {
  req.result.createObjectStore('images', { keyPath: 'id' })
}
```

画像データは以下の形式で保存しています：

```javascript
{
  id: crypto.randomUUID(),
  fileName: file.name,
  mimeType: file.type,
  blob: file,           // 画像のBlobをそのまま保存
  annotation: '',       // AIが生成したタグ
  createdAt: Date.now()
}
```

## 今回作るアプリケーション

画像の保存はIndexedDB

画像のタグ生成にはPrompt API

Rest APIは使っていますが、Service Workerで動かしています

## 実装していく

### DB

IndexedDBのラッパーを作りました。Promise化して使いやすくしています。

```javascript
export async function getAllImages() {
  const db = await openDB()
  const store = db.transaction('images', 'readonly').objectStore('images')
  return new Promise((resolve, reject) => {
    const req = store.getAll()
    req.onsuccess = () => resolve(req.result)
    req.onerror = () => reject(req.error)
  })
}
```

### Routing

Service Worker内でHonoを使ってルーティングしています。`/api/*` へのリクエストをインターセプトして処理します。

```javascript
import { Hono } from 'https://esm.sh/hono@4'

const app = new Hono()

app.get('/api/images', async (c) => {
  const all = await getAllImages()
  return c.html(renderList(all))
})

app.post('/api/images', async (c) => {
  const files = (await c.req.formData()).getAll('file')
  for (const file of files) {
    await saveImage(file, await requestAnnotation(file))
  }
  return c.html(renderList(await getAllImages()))
})

self.addEventListener('fetch', (e) => {
  if (new URL(e.request.url).pathname.startsWith('/api')) {
    e.respondWith(app.fetch(e.request))
  }
})
```

ESM形式でHonoをインポートできるのが便利ですね。

### Prompt API

サービスワーカーでPrompt APIにアクセスできないのでメインスレッドにプロキシするようにしたよ

MessageChannelを使ってService Workerとメインスレッド間で通信しています。

```javascript
// メインスレッド側
const channel = new MessageChannel()
channel.port1.onmessage = handleMessage
navigator.serviceWorker.controller?.postMessage({ type: 'INIT_PORT' }, [channel.port2])

function handleMessage(e) {
  if (e.data.type === 'ANNOTATE') {
    annotateImage(e.data.blob).then(annotation => 
      e.currentTarget.postMessage({ id: e.data.id, annotation })
    )
  }
}
```

画像のアノテーション生成はこんな感じ：

```javascript
async function annotateImage(blob) {
  const session = await self.LanguageModel.create({
    expectedInputs: [{ type: 'image' }, { type: 'text' }],
    languageCode: 'ja'
  })
  
  const file = new File([blob], 'image.jpg', { type: blob.type })
  let result = ''
  
  for await (const chunk of session.promptStreaming([{
    role: 'user',
    content: [
      { type: 'text', value: 'この画像に含まれる要素をタグ形式で列挙してください。' },
      { type: 'image', value: file }
    ]
  }])) {
    result += chunk
  }
  
  session.destroy()
  return result
}
```

検索もAIを使って意味的なマッチングをしています。「生き物」で検索すると猫や犬の画像がヒットするようになっています。

### フロント

特になし、htmxとtailwindを使って実装した

htmxのおかげでJavaScriptをほとんど書かずにSPA的な動作を実現できています。

```html
<input type="file" accept="image/*" multiple
  hx-post="/api/images"
  hx-target="#imageList"
  hx-encoding="multipart/form-data">

<input type="search" name="q" placeholder="自然言語で検索..."
  hx-get="/api/search"
  hx-trigger="input changed delay:500ms"
  hx-target="#imageList">
```

## 使ってみる

ここは私が書きます

## まとめ

ローカルAI侮れない

ワクワクする

何しよかな

色々アイデアが生まれてきますね

ぜひ使ってみてね、よ〜んには何も飛んで来ませんが、変な画像とかは保存しない方がいいと思います
