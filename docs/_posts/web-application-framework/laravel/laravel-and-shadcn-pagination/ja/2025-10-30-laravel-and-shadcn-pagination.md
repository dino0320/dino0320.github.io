---
layout: my-post
title: "Laravel + shadcn/ui ページネーション"
date: 2025-10-30 00:00:00 +0000
categories: web-application-framework laravel
page_name: laravel-and-shadcn-pagination
lang: ja
image: /assets/images/web-application-framework/laravel/laravel-and-shadcn-pagination/image1.png
---

この記事では、shadcn/uiを使用してLaravelのページネーションデータを表示する方法を解説します。

![サムネイル](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "サムネイル")

## 参考
- [Laravel Database: Pagination](https://laravel.com/docs/12.x/pagination)
- [shadcn/ui Pagination](https://ui.shadcn.com/docs/components/pagination)
- [カーソル vs. オフセットペジネーション](https://readouble.com/laravel/8.x/ja/pagination.html)

## 前提
- React環境でセットアップ済みのLaravelプロジェクトがあること。  
詳しくは[こちらの記事](/web-application-framework/laravel/set-up-laravel-with-react-project)を参照してください。

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

## 作業の流れ
1. [Laravelのページネーションを実装する](#1-laravelのページネーションを実装する)
2. [shadcn/uiのページネーションを実装する](#2-shadcnuiのページネーションを実装する)
3. [ページネーションを確認する](#3-ページネーションを確認する)

- [(おまけ) その他のページネーションメソッド](#おまけ-その他のページネーションメソッド)

## 1. Laravelのページネーションを実装する
Laravelには、query builderとEloquent ORMが使えるページネーション機能が備わっています。  
今回はEloquentを使用してみます。

Artisanコマンドで `UserController` を作成します。

```bash
php artisan make:controller UserController
```

次に、ページネーション処理を追加します。  
作成されたコントローラーファイルを以下のように編集します。

`UserController.php`:

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Inertia\Inertia;
use Inertia\Response;

class UserController extends Controller
{
    /**
     * すべてのユーザーを表示する
     */
    public function index(): Response
    {
        return Inertia::render('User/Index', [
            'users' => User::paginate(5)
        ]);
    }
}

```

[公式ドキュメント](https://laravel.com/docs/12.x/pagination)によると、

> paginateメソッドは、ユーザーが閲覧中のページに基づいて、クエリの "limit" および "offset" を自動的に設定します。(翻訳済み)

Laravelがページネーション処理をほとんど自動で行ってくれることがわかります。  
つまり `paginate` メソッドに表示する件数を指定し、フロントエンド側でUIを実装するだけでページネーション機能が完成します。

## 2. shadcn/uiのページネーションを実装する
shadcn/uiのPaginationコンポーネントをインストールします。

```bash
npx shadcn@latest add pagination
```

コマンドを実行すると、`resources/js/components/ui` ディレクトリ内に `pagination.tsx` ファイルが作成されます。

次に、Inertia.jsとshadcn/uiを使用してユーザー一覧ページを作成します。

`resources/js/Pages/User/Index.tsx` というファイルを作成します。

`Index.tsx`:

```tsx
import {
  Card,
  CardContent,
  CardHeader,
  CardTitle,
} from '@/components/ui/card'
import {
  Pagination,
  PaginationContent,
  PaginationItem,
  PaginationNext,
  PaginationPrevious,
} from "@/components/ui/pagination"

// Laravelのページネーションデータ構造(必要なプロパティのみ)
type PaginationData<T> = {
  data: T[],
  prev_page_url: string | null,
  next_page_url: string | null,
}

// usersテーブルのデータ構造(表示カラムのみ)
type User = {
  name: string,
  email: string,
}

export default function Index({ users }: { users: PaginationData<User>}) {
  // 前後ページのリンクが存在する場合に表示する
  const prevPageLink = users.prev_page_url === null ? null : (
    <PaginationItem>
      <PaginationPrevious href={users.prev_page_url} />
    </PaginationItem>
  )

  const nextPageLink = users.next_page_url === null ? null : (
    <PaginationItem>
      <PaginationNext href={users.next_page_url} />
    </PaginationItem>
  )

  return (
    <div>
      {/* ユーザーデータを表示する */}
      {users.data.map((user, index) => (
        <Card key={index}>
          <CardHeader>
            <CardTitle>{user.name}</CardTitle>
          </CardHeader>
          <CardContent>
            {user.email}
          </CardContent>
        </Card>
      ))}
      
      {/* ページネーションを表示する */}
      <Pagination>
        <PaginationContent>
          {prevPageLink}
          {nextPageLink}
        </PaginationContent>
      </Pagination>
    </div>
  )
}

```

もしCardコンポーネントをまだインストールしていない場合は以下のコマンドで追加します。

```bash
npx shadcn@latest add card
```

`users` のページネーションレスポンスには、以下のようなプロパティが含まれます。

| プロパティ名 | 説明 |
|----------|-------------|
| current_page | 現在のページ番号 |
| data | データベースから取得されたデータリスト |
| first_page_url | 最初のページのURL |
| from | 現在のページの開始アイテム番号 |
| last_page | 総ページ数 |
| last_page_url | 最後のページのURL |
| links | ページナビゲーション用リンクリスト |
| next_page_url | 次のページのURL |
| path | 現在のページのベースURL |
| per_page | 1ページあたりの表示件数 |
| prev_page_url | 前のページのURL |
| to | 現在のページの終了アイテム番号 |
| total | 総アイテム数 |

## 3. ページネーションを確認する
ルーティング設定を追加します。

`web.php`:

```php
Route::get('/users', [UserController::class, 'index']);
```

ブラウザで `/users` ページを開くと、次のようなページが表示されます。

![ユーザー一覧](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "ユーザー一覧")

`Next` ボタンをクリックすると、次のページが表示されます。

![ユーザー一覧の2ページ目](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "ユーザー一覧の2ページ目")

## (おまけ) その他のページネーションメソッド
Laravelには他にもいくつかのページネーションメソッドがあります。

### simplePaginate
[公式ドキュメント](https://laravel.com/docs/12.x/pagination)によると、`paginate` メソッドはデータ取得前に一致するレコードの総数をカウントします。  
もし総件数を表示する必要がない場合は、`simplePaginate` メソッドを使うとパフォーマンスが向上します。

```php
User::simplePaginate(5)
```

このメソッドは以下のプロパティを返します。

| プロパティ名 | 説明 |
|----------|-------------|
| current_page | 現在のページ番号 |
| current_page_url | 現在のページのURL |
| data | データベースから取得されたデータリスト |
| first_page_url | 最初のページのURL |
| from | 現在のページの開始アイテム番号 |
| next_page_url | 次のページのURL |
| path | 現在のページのベースURL |
| per_page | 1ページあたりの表示件数 |
| prev_page_url | 前のページのURL |
| to | 現在のページの終了アイテム番号 |

### cursorPaginate
[公式ドキュメント](https://laravel.com/docs/12.x/pagination)によると、`cursorPaginate` メソッドは最も効率的なページネーション方法です。

> paginateやsimplePaginateがSQLの "offset" 句を使ってクエリを生成するのに対し、cursorページネーションはクエリ内の並び替えカラムを比較する "where" 句を使うため、Laravelの中で最も効率的なデータベースパフォーマンスを発揮します。(翻訳済み)

オフセットとカーソルの違いは以下のようになります。([こちらのページ](https://laravel.com/docs/12.x/pagination)を参照)

```
# オフセットページング
select * from users order by id asc limit 15 offset 15;

# カーソルページング
select * from users where id > 15 order by id asc limit 15;
```

レスポンスには以下のプロパティが含まれます。

| プロパティ名 | 説明 |
|----------|-------------|
| data | データベースから取得されたデータリスト |
| next_cursor | 次のカーソル値 |
| next_page_url | 次のページのURL |
| path | 現在のページのベースURL |
| per_page | 1ページあたりの表示件数 |
| prev_cursor | 前のカーソル値 |
| prev_page_url | 前のページのURL |