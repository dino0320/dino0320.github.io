---
layout: my-post
title: "Understanding Rainbow Tables: Recovering Plaintext from Hashes (PHP)"
date: 2024-08-23 00:00:00 +0000
categories: security hash
page_name: getting-plaintext-from-hash-value-using-rainbow-table-en
lang: en
image: /assets/images/security/hash/getting-plaintext-from-hash-value-using-rainbow-table-en/image1.png
---
 
This article explores how rainbow tables work by experimentally recovering plaintext from hash values for educational purposes.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Thumbnail")

## References
- [レインボーテーブルとは？攻撃の仕組みや対策方法をわかりやすく解説 - ITトレンド](https://it-trend.jp/encryption/article/64-0067)
- [レインボーテーブルで使う還元関数の作り方 - Lazy Diary @ Hatena Blog](https://satob.hatenablog.com/entry/20120219/p1)

## Environment
- Debian GNU/Linux 12 (bookworm)
- PHP 8.2.17 (cli)

## Contents
- [What is the rainbow table?](#what-is-the-rainbow-table)
- [Creating a PHP program getting a plaintext from a hash value.](#creating-a-php-program-getting-a-plaintext-from-a-hash-value)

## What is the rainbow table?
According to a following web page, the rainbow table is used for getting a plaintext from a hash value.

> レインボーテーブルとは平文とハッシュ値の対応表であり、ハッシュ値を検索して対応する平文を探す手法として活用されます。
> 
> Source: [レインボーテーブルとは？攻撃の仕組みや対策方法をわかりやすく解説 - ITトレンド](https://it-trend.jp/encryption/article/64-0067)

## Creating a PHP program getting a plaintext from a hash value.
I created a PHP program getting a plaintext from a hash value.

The program followed steps below. 
1. Creating a rainbow table.
2. Getting a plaintext corresponding to a input hash value.

Typically, the rainbow table used may be stored in storages for example the database.  
However, this time I implemented a process of creating a rainbow table and a process of getting a plaintext in the same program.  
Therefore you may not get an output you expect.

The algorithm used was referred from [this web page](https://it-trend.jp/encryption/article/64-0067).  
The reduction function used was referred from [this web page](https://satob.hatenablog.com/entry/20120219/p1).  
I changed a part of the reduction function process from the right shift to the multiplication.

The program I created is a following.

```php
<?php

/**
 * Default number of chains
 */
const DEFAULT_CHAIN_COUNT = 100;
/**
 * Default number of reduction functions
 */
const DEFAULT_REDUCTION_COUNT = 100;
/**
 * A list of characters used in an output of a reduction function
 */
const FIGURE_LIST = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z'];

// A hash value converted back to a plaintext
$targetHashValue = $argv[1] ?? throw new Exception('Specify any hash value as the first argument.');
// Number of chains
$chainCount = $argv[2] ?? DEFAULT_CHAIN_COUNT;
// Number of reduction functions
$reductionCount = $argv[3] ?? DEFAULT_REDUCTION_COUNT;

// 1. Creating a rainbow table.
$rainbowTable = createRainbowTable($chainCount, $reductionCount);

// 2. Getting a plaintext corresponding to a input hash value.
$targetPlaintext = getTargetPlaintext($reductionCount, $targetHashValue, $rainbowTable);

if ($targetPlaintext === '') {
    echo 'Could not get plaintext.' . PHP_EOL;
    exit;
}

echo $targetPlaintext . PHP_EOL;

/**
 * Create a rainbow table.
 *
 * @param integer $chainCount Number of chains
 * @param integer $reductionCount Number of reduction functions
 * @return array A rainbow table
 */
function createRainbowTable(int $chainCount, int $reductionCount): array
{
    $rainbowTable = [];
    for ($i = 0; $i < $chainCount; $i++) {
        // The first plaintext of the chain is an integer between 0 and the number of chains.
        $chain = createChain($i, $reductionCount);
        // The start and end value of the chain is stored in the rainbow table.
        $rainbowTable[] = ['start' => $chain[0], 'end' => $chain[count($chain) - 1]];
    }

    return $rainbowTable;
}

/**
 * Create a chain.
 *
 * @param string $startText First plaintext of a chain
 * @param integer $reductionCount Number of reduction functions
 * @param integer $startNumber The number of the first reduction function in a chain
 * @return array A chain
 */
function createChain(string $startText, int $reductionCount, int $startNumber = 0): array
{
    $chain = [$startText];
    for ($i = $startNumber; $i < $reductionCount; $i++) {
        $hashValue = executeHash($startText);
        $plaintext = executeReduction($hashValue, $i + 1);

        // The hash value and the output of the reduction function are stored in the chain alternately.
        $chain[] = $hashValue;
        $chain[] = $plaintext;
    }

    return $chain;
}

/**
 * Execute a hash function.
 *
 * @param string $plaintext A plaintext
 * @return string A hash value
 */
function executeHash(string $plaintext): string
{
    // using md5.
    return md5($plaintext);
}

/**
 * Execute a reduction function.
 *
 * @param string $hashValue A hash value
 * @param int $multipliedValue A multiplied value
 * @return string A plaintext
 */
function executeReduction(string $hashValue, int $multipliedValue): string
{
    // The hash value converted to decimal is multiplied by different value for each reduction function.
    // So each reduction function is different.
    $hashValueDec = hexdec($hashValue) * $multipliedValue;

    // The above value is converted to base 36 (using 0-9 and a-z).
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
 * Get a plaintext from a rainbow table.
 *
 * @param integer $reductionCount Number of reduction functions
 * @param string $hashValue A hash value
 * @param array $rainbowTable A rainbow table
 * @return string A plaintext
 */
function getTargetPlaintext(int $reductionCount, string $hashValue, array $rainbowTable): string
{
    for ($i = $reductionCount; $i > 0; $i--) {
        // The reduction function is executed for the input hash value ​​in the reverse order that the rainbow table was created.
        $plaintext = executeReduction($hashValue, $i);
        // Creating a chain starting from the output above.
        // The chain is in the same order the rainbow table was created (from the next reduction function executed above to the final reduction function).
        // For example, in a case that number of reduction functions is five, if an output is the third reduction function, a chain that executes the fourth and fifth is created.
        $chain = createChain($plaintext, $reductionCount, $i);

        $endValue = $chain[count($chain) - 1];
        foreach ($rainbowTable as $startAndEndValueOfChain) {
            // The end value of the chain created above is compared to each end value of the chain in the rainbow table.
            if ($endValue === $startAndEndValueOfChain['end']) {
                // if they are same, a chain is created starting from the start value of that chain in the rainbow table.
                $chain = createChain($startAndEndValueOfChain['start'], $reductionCount);
                // The previous value of the input hash value in the chain above is an output plaintext.
                $indexOfChain = array_search($hashValue, $chain);
                return $chain[$indexOfChain - 1];
            }
        }
    }

    return '';
}
```

※First plaintext of a chain is an integer between `0 and number of chains`.  
※Using md5 as a hash function.

The program can be executed on the command line as a follow.  
Number of chains and reduction functions are optional (default is 100 for both).

```bash
$ php <A name of a file to save the above program to>.php <A hash value> [<Number of chains>] [<Number of reduction functions>]
```

The program takes too much time when you specify too much number of chains and reduction functions.  
Because in this case, a rainbow table is stored in the memory.  
But if they are too few, you will not get any output because there will be no collisions.  
If you want to see an output, specify a hash value of an integer between `0 and number of chains`.