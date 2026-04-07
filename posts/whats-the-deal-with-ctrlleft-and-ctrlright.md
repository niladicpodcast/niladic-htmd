---
author: Cody
title: What's the deal with Ctrl+Left and Ctrl+Right?
_hash: af142171a3809bf96ee1560e8d9f32e3b3e5a8da64e9fa1990cb8c510a731531
published: 2017-01-14T00:00:00+00:00
updated: 2026-04-07T09:03:47.412804+00:00
...

What is with the behaviour of `Ctrl+Left` and `Ctrl+Right`? Almost every editor behaves differently.

The different behaviour makes it hard to use efficiently.

Use this text as an example and put your cursor at the beginning and press `Ctrl+Right` until you reach the end.

```
simple sentence with words.
under_scores
hyphen-words
```

Where do you think the cursor will stop? Where does it stop?

On Windows in Notepad++ it stops where I've added the `|`

```
simple |sentence |with |words|.|
|under_scores|
|hyphen|-|words|
```

In Atom on Windows it stops:

```
simple| sentence| with| words|.|
under_scores|
hyphen|-|words|
```

In Atom it stops at the end of words, that's good if you are replacing the word, but if you are deleting the first word, you also want to delete the space after it.

Neither of them stopped at the underscore. That's not good if you want to replace the first part before the underscore. For example, changing the variable `file_name` to `folder_name` when you change your code.

This behaviour isn't in the preferences for editors and you wouldn't want the behaviour to be different in different editors.

Some of you might be thinking, just use `vim` or `emacs` and you don't have to worry about using `Ctrl+Left` and `Ctrl+Right` because there are better ways of navigation. But you are not always in `vim` or `emacs`, sometimes you are in a browser or an IDE.

Another strange thing is going backwards from the end of the text. With the cursor at the end of the text, if I repeatedly press `Ctrl+Left` in Atom I stop at the beginning of words like in Notepad++.

I end up just using `Ctrl+End` or `Ctrl + Home` because I know what it is going to do and for the most part I don't want to stop after each symbol. If I wanted to do that I would just use `Right` or `Left`.

For example:

```
test(False)
```

Stops at

```
test|(|False)
```

So I can be at the end of `test` and delete the rest of the word, but I have to hit `Ctrl+Right` three times.
I want to go the next whitespace, unless it is a hyphen or an underscore.

But inside the parentheses I want different behaviour.

```
test(False|, True|)
```
But if you are inside the parentheses I would want to stop before the closing parenthesis.
If the cursor is at the first `|`, then I want to go to the second one to delete the second parameter for example.

So maybe this isn't the correct solution, maybe a key that does the equivalent of double clicking the mouse is actually what I want.

I guess this is why people end up writing their own version of existing software. Maybe I should write my own editor, or at least try and then realize why what I want is not possible.

So really there are two problems, why isn't the `Ctrl+Left` and `Ctrl+Right` behaviour the same everywhere, and when we make it consistent can we make it good?

I'll just leave this here https://xkcd.com/927/.
