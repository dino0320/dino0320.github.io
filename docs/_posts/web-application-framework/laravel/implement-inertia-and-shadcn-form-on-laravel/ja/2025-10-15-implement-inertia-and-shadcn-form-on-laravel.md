---
layout: my-post
title: "LaravelでInertia.js + shadcn/uiフォームを実装する"
date: 2025-10-15 00:00:00 +0000
categories: web-application-framework laravel
page_name: implement-inertia-and-shadcn-form-on-laravel
lang: ja
---

LaravelプロジェクトでInertia.jsとshadcn/uiを使ってフォームを実装する方法を説明します。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "サムネイル")

## 参考
- [shadcn/ui Form](https://ui.shadcn.com/docs/components/form)
- [inertia.js Pages](https://inertiajs.com/pages)

## 前提
- React環境でセットアップ済みのLaravelプロジェクトがあること  
詳細は[こちらの記事](/web-application-framework/laravel/set-up-laravel-with-react-project)を参照してください。

## 環境
- Windows 11
- Ubuntu 24.04.3 LTS (WSL2ディストリビューション)
- Docker Engine 28.4.0
- Amazon Linux 2023(DockerコンテナのOS)
- PHP 8.2.15 (fpm-fcgi)
- NGINX 1.24.0
- Laravel 12
- Node.js 20.14.0
- Vite 7.1.9
- React 19.2.0
- @inertiajs/react 2.2.6

## 実装手順
1. [shadcn/ui Formのインストール](#1-shadcnui-formのインストール)
2. [テストフォームの実装](#2-テストフォームの実装)

## 1. shadcn/ui Formのインストール
以下のコマンドを実行して、shadcn/uiのFormコンポーネントをインストールします。

```bash
npx shadcn@latest add form
```

実行後、`resources/js/components/ui` ディレクトリ内に `form.tsx` ファイルが生成されます。

## 2. テストフォームの実装
Inertia.jsとshadcn/uiのFormを使用してテストフォームを作成します。

まず、[shadcn/uiのドキュメント](https://ui.shadcn.com/docs/components/form)を参考に、`resources/js/Pages/TestForm.tsx` ファイルを作成します。

`TestForm.tsx`:

```tsx
import { zodResolver } from "@hookform/resolvers/zod"
import { useForm } from "react-hook-form"
import { z } from "zod"
 
import { Button } from "@/components/ui/button"
import {
  Form,
  FormControl,
  FormDescription,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from "@/components/ui/form"
import { Input } from "@/components/ui/input"
 
const formSchema = z.object({
  username: z.string().min(2, {
    message: "Username must be at least 2 characters.",
  }),
})
 
export default function TestForm() {
  // 1. フォームを定義
  const form = useForm<z.infer<typeof formSchema>>({
    resolver: zodResolver(formSchema),
    defaultValues: {
      username: "",
    },
  })
 
  // 2. 送信ハンドラを定義
  function onSubmit(values: z.infer<typeof formSchema>) {
    // フォームの値を処理
    // ✅ 型安全かつバリデーション済み
    console.log(values)
  }
 
  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-8">
        <FormField
          control={form.control}
          name="username"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Username</FormLabel>
              <FormControl>
                <Input placeholder="shadcn" {...field} />
              </FormControl>
              <FormDescription>
                This is your public display name.
              </FormDescription>
              <FormMessage />
            </FormItem>
          )}
        />
        <Button type="submit">Submit</Button>
      </form>
    </Form>
  )
}

```

未インストールのパッケージがあればインストールしてください。

次に、`onSubmit` 関数を修正して、Inertia経由でLaravelにデータを送信します。

```tsx
import { router } from '@inertiajs/react' // 追加

// ...

  // 2. 送信ハンドラを定義
    function onSubmit(values: z.infer<typeof formSchema>) {
    // フォームの値を処理
    // ✅ 型安全かつバリデーション済み
    console.log(values)
    // /show-username にデータを送信
    router.post('/show-username', values) // 追加
  }

// ...

```

次にArtisanコマンドでコントローラを作成します。

```bash
php artisan make:controller TestFormController
```

`app/Http/Controllers/TestFormController.php` を以下のように編集します。  
`showUsername` メソッドはフォームから送信された `username` を受け取り、`Welcome` ページに渡して表示します。

`TestFormController.php`:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Inertia\Inertia;
use Inertia\Response;

class TestFormController extends Controller
{
    /**
     * ユーザー名を表示
     *
     * @param Request $request
     * @return Response
     */
    public function showUsername(Request $request): Response
    {
        $request->validate([
            'username' => 'required|string|max:255',
        ]);

        return Inertia::render('Welcome', [
          'username' => $request->username,
        ]);
    }
}

```

次に、[Inertia.jsのドキュメント](https://inertiajs.com/pages)を参考に、`resources/js/Pages/Welcome.tsx` を作成します。

`Welcome.tsx`:

```tsx
import { Head } from '@inertiajs/react'

export default function Welcome({ username }) {
  return (
    <div>
      <Head title="Welcome" />
      <h1>Welcome</h1>
      <p>Hello {username}, welcome to your first Inertia app!</p>
    </div>
  )
}

```

最後に `routes/web.php` を以下のように編集します。

`web.php`:

```php
<?php

use App\Http\Controllers\TestFormController;
use Illuminate\Support\Facades\Route;
use Inertia\Inertia;

Route::get('/test-form', function () {
    return Inertia::render('TestForm');
});

Route::post('/show-username', [TestFormController::class, 'showUsername']);

```

ブラウザで `/test-form` ページを開くと次のようなフォームが表示されます。

![Test form](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Test form")

任意のユーザー名を入力して "Submit" ボタンをクリックすると、"Welcome" ページにリダイレクトされ、正しく動作していることが確認できます。

![Welcome page](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "Welcome page")