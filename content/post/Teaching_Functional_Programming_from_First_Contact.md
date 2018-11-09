---
title: "Teaching Functional Programming from First Contact"
date: 2018-10-08T13:24:14-04:00
draft: true
---

```js
"use strict";
const fs = require("fs");

const BANNED_LETTERS = ["g", "k", "m", "q", "v", "w", "x", "z"];

const words = fs
  .readFileSync("./words.txt")
  .toString()
  .split("\n");

// function isLetterAllowed(word) {
//   BANNED_LETTERS.every(bannedLetter => bannedLetter != letter);
// }

const isLetterAllowed = letter =>
  BANNED_LETTERS.every(bannedLetter => bannedLetter != letter);

const isWordAllowed = word =>
  word.split("").every(letter => isLetterAllowed(letter));

const longerWord = (largest, word) =>
  largest.length > word.length ? largest : word;

const longestWord = words.filter(isWordAllowed).reduce(longerWord);

console.log(longestWord);
```


```hs
module Main where

import           Data.List
import           Data.Ord
import           Lib

isLetterAllowed :: Char -> Bool
isLetterAllowed letter =
  letter `notElem` ['g', 'k', 'm', 'q', 'v', 'w', 'x', 'z']

isWordAllowed :: String -> Bool
isWordAllowed = all isLetterAllowed

longestInList =
  maximumBy (\largest word -> length largest `compare` length word)

longestWord = readFile "./words.txt"
  >>= \dictStr -> print $ longestInList $ filter isWordAllowed $ lines dictStr

main = longestWord
```


```rs
use std::fs::File;
use std::io::prelude::*;
use std::io::BufReader;

const BANNED_LETTERS: [char; 8] = ['g', 'k', 'm', 'q', 'v', 'w', 'x', 'z'];

fn main() {
    let words = BufReader::new(File::open("./words.txt").expect("file does not exist")).lines();

    // fn is_letter_allowed(letter: char) -> bool {
    //     BANNED_LETTERS.into_iter().all(|banned| letter != *banned)
    // }
    let is_letter_allowed = |letter| BANNED_LETTERS.into_iter().all(|banned| letter != *banned);

    let longest_word = words
        .map(|word| word.expect("bad word encoding"))
        .filter(|word| word.chars().all(is_letter_allowed))
        .max_by(|a, b| a.len().cmp(&b.len()))
        .expect("no words found");

    println!("{}", longest_word);
}
```
#test

