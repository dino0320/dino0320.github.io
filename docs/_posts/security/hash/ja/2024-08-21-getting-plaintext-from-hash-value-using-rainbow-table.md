---
layout: my-post
title: "レインボーテーブルを使ってハッシュ値から平文を取得する"
date: 2024-08-21 00:00:00 +0000
categories: security hash
title_eng: getting-plaintext-from-hash-value-using-rainbow-table
lang: ja
---

PHPでレインボーテーブルを使ってハッシュ値から平文を取得する処理を試しました。

## 参考ページ
- [レインボーテーブルとは？攻撃の仕組みや対策方法をわかりやすく解説 - ITトレンド](https://it-trend.jp/encryption/article/64-0067)
- [レインボーテーブルで使う還元関数の作り方 - Lazy Diary @ Hatena Blog](https://satob.hatenablog.com/entry/20120219/p1)

## 環境
- Debian GNU/Linux 12 (bookworm)
- PHP 8.2.17 (cli)

## もくじ
- [レインボーテーブルとは](#レインボーテーブルとは)
- [ハッシュ値から平文を取得するPHPプログラムを作成する](#ハッシュ値から平文を取得するphpプログラムを作成する)

## レインボーテーブルとは
引用させていただいた以下の文章のように、レインボーテーブルはハッシュ値から平文を求めるために利用されます。

> レインボーテーブルとは平文とハッシュ値の対応表であり、ハッシュ値を検索して対応する平文を探す手法として活用されます。
> 
> 引用：[レインボーテーブルとは？攻撃の仕組みや対策方法をわかりやすく解説 - ITトレンド](https://it-trend.jp/encryption/article/64-0067)

## ハッシュ値から平文を取得するPHPプログラムを作成する
レインボーテーブルを使ってハッシュ値から平文を取得するPHPプログラムを作成します。

大まかな処理の流れは以下の通りです。
1. レインボーテーブルを作成する。
2. 入力したハッシュ値に対応する平文を取得する。

本来はレインボーテーブルをデータベース等に保存して使うと思いますが、今回はレインボーテーブルの作成処理と平文の特定処理を同じプログラムとして実装します。(そのため思ったように結果は出ません)

上記の処理のアルゴリズムは[こちらのページ](https://it-trend.jp/encryption/article/64-0067)を参考に作成しています。  
還元関数は[こちらのページ](https://satob.hatenablog.com/entry/20120219/p1)を参考に、右シフトの処理を乗算に変えています。

以下作成したプログラムのソースコードになります。

```php
<?php

/**
 * デフォルトのチェインの数
 */
const DEFAULT_CHAIN_COUNT = 100;
/**
 * デフォルトの還元関数の数
 */
const DEFAULT_REDUCTION_COUNT = 100;
/**
 * 還元関数の結果に使う文字のリスト
 */
const FIGURE_LIST = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z'];

// 平文に戻すハッシュ値
$targetHashValue = $argv[1] ?? throw new Exception('ハッシュ値を第一引数に指定してください。');
// チェインの数
$chainCount = $argv[2] ?? DEFAULT_CHAIN_COUNT;
// 還元関数の数
$reductionCount = $argv[3] ?? DEFAULT_REDUCTION_COUNT;

// 1. レインボーテーブルを作成する。
$rainbowTable = createRainbowTable($chainCount, $reductionCount);

// 2. 入力したハッシュ値に対応する平文を取得する。
$targetPlaintext = getTargetPlaintext($reductionCount, $targetHashValue, $rainbowTable);

if ($targetPlaintext === '') {
    echo '平文を取得できませんでした。' . PHP_EOL;
    exit;
}

echo $targetPlaintext . PHP_EOL;

/**
 * レインボーテーブルを作成する
 *
 * @param integer $chainCount チェインの数
 * @param integer $reductionCount 還元関数の数
 * @return array レインボーテーブル
 */
function createRainbowTable(int $chainCount, int $reductionCount): array
{
    $rainbowTable = [];
    for ($i = 0; $i < $chainCount; $i++) {
        // チェインの最初の平文は 0～チェインの数 の整数にする
        $chain = createChain($i, $reductionCount);
        // レインボーテーブルにはチェインの最初の値と最後の値のみを保存する
        $rainbowTable[] = ['start' => $chain[0], 'end' => $chain[count($chain) - 1]];
    }

    return $rainbowTable;
}

/**
 * チェインを作成する
 *
 * @param string $startText チェインの最初の平文
 * @param integer $reductionCount 還元関数の数
 * @param integer $startNumber チェインの最初の還元関数の番号
 * @return array チェイン
 */
function createChain(string $startText, int $reductionCount, int $startNumber = 0): array
{
    $chain = [$startText];
    for ($i = $startNumber; $i < $reductionCount; $i++) {
        $hashValue = executeHash($startText);
        $plaintext = executeReduction($hashValue, $i + 1);

        // ハッシュ値と還元関数の出力を交互にチェインに入れていく
        $chain[] = $hashValue;
        $chain[] = $plaintext;
    }

    return $chain;
}

/**
 * ハッシュ関数を実行する
 *
 * @param string $plaintext 平文
 * @return string ハッシュ値
 */
function executeHash(string $plaintext): string
{
    // ハッシュ関数はMD5を使用する
    return md5($plaintext);
}

/**
 * 還元関数を実行する
 *
 * @param string $hashValue ハッシュ値
 * @param int $multipliedValue 乗算される値
 * @return string 平文
 */
function executeReduction(string $hashValue, int $multipliedValue): string
{
    // ハッシュ値を10進数にしたものに異なる値を乗算することで、チェインの各還元関数を異なるものにする
    $hashValueDec = hexdec($hashValue) * $multipliedValue;

    // 上記の値を36進数(0～9 と a～z の種類の合計)に変換する
    $figureCount = count(FIGURE_LIST);
    $plaintext = '';
    while($hashValueDec >= $figureCount) {
        $currentDigitFigure = FIGURE_LIST[(int)fmod($hashValueDec, $figureCount)];
        $plaintext = "{$currentDigitFigure}{$plaintext}";
        $hashValueDec /= $figureCount;
    }

    $currentDigitFigure = FIGURE_LIST[(int)$hashValueDec];

    return "{$currentDigitFigure}{$plaintext}";
}

/**
 * レインボーテーブルから平文を取得する
 *
 * @param integer $reductionCount 還元関数の数
 * @param string $hashValue ハッシュ値
 * @param array $rainbowTable レインボーテーブル
 * @return string 平文
 */
function getTargetPlaintext(int $reductionCount, string $hashValue, array $rainbowTable): string
{
    for ($i = $reductionCount; $i > 0; $i--) {
        // レインボーテーブルを作成した順番とは逆に、還元関数を入力したハッシュ値に実行していく
        $plaintext = executeReduction($hashValue, $i);
        // 上記の還元関数の結果から始めたチェインを作成する
        // レインボーテーブルを作成したときと同じ順番に作成する(上記で実行した還元関数の次の関数から最後の還元関数まで)
        // 例えば還元関数を5回実行するチェインのとき、3番目の還元関数の結果だったら4番目と5番目を実行するチェインを作成する
        $chain = createChain($plaintext, $reductionCount, $i);

        $endValue = $chain[count($chain) - 1];
        foreach ($rainbowTable as $startAndEndValueOfChain) {
            // 上記で作成したチェインの最後の値とレインボーテーブルの各チェインの最後の値を比較する
            if ($endValue === $startAndEndValueOfChain['end']) {
                // 同じだったらレインボーテーブルのそのチェインの最初の値から再びチェインを作成する
                $chain = createChain($startAndEndValueOfChain['start'], $reductionCount);
                // 上記のチェインの、入力したハッシュ値の一個前の値が求める平文になる
                $indexOfChain = array_search($hashValue, $chain);
                return $chain[$indexOfChain - 1];
            }
        }
    }

    return '';
}
```

※チェインの最初の数は `0～チェインの数` の整数にしています。  
※ハッシュ関数にはMD5を使用しています。

コマンドラインで以下のように実行できるようにしています。  
チェインの数と還元関数の数はオプションです。(デフォルトで両方とも100)

```bash
$ php <上記を保存したファイル名>.php <任意のハッシュ値> [<任意のチェインの数>] [<任意の還元関数の数>]
```

レインボーテーブルをメモリで保持するので、チェインの数と還元関数の数を増やしすぎるとオーバーフローしたり時間がかかりすぎたりします。  
しかし少ないとハッシュ値が衝突しないので結果がでません。。。  
確実に結果が見たい場合は `0～チェインの数` の整数のハッシュ値を入力してください。