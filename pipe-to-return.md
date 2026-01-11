# PHP RFC: Pipe to return

Version: 0.0.1

Date: 2026-01-08

Author: Vadim Dvorovenko, vadimon@mail.ru

Target Version: PHP 8.6

Status: Draft

Implementation: https://github.com/vadimonus/php-src/commits/pipe-to-return

# Introduction

Piping result of expression to `return` keyword can be used to reduce congnitive complexity.

```php
// Current syntax
return "Hello World" |> strlen(...);

// New Syntax
"Hello World" |> strlen(...) |> return;
```

# The problem

Pipe operator introduced in RFC (https://wiki.php.net/rfc/pipe-operator-v3) may lead to a complication in the perception of the flow due to mixing of left-to-right and right-to-left action directions. To avoid this, it is proposed to use the pipe operator for passing expression result to `return`.

Long-standing traditions of procedural code uses two main reading directions: top-to-bottom for procedural flow and right-to-left for function call and assignment flow in single line.
```php
// Top-to-bottom example
$temp = "Hello World";          //   -1-|
                                //      |
$temp = htmlentities($temp);    //   <--|    -2-|
                                //              |
$temp = str_split($temp);       //           <--|   -3-|
                                //                     | 
$temp = array_map(strtoupper(...), $temp); //       <--|    -4-|
                                //                             |
return $temp;                   //                          <--|

// Left-to-right example.
//  |-4-|     |----------3---------|     |-2-|        |-1-|  
//  V   |     V                    |     V   |        V   |
return array_map(strtoupper(...), str_split(htmlentities("Hello World")));
```

The programmer's brain is tuned to perceive exactly these directions. When we try to reverse this directions, we need to to introduce additional rules, such as idents, to make hints to our brain that we should use bottom-to-top direction instead of top-to-bottom.

```php
return 
//   ^
//   |
    array_map(
    //  ^
    //  |
        strtoupper(...),
        str_split(
        //   ^   
        //   |   
            htmlentities(
            //   ^   
            //   |   
                "Hello World"
            )
    )
);
```

Pipline operator and fluent pattern were invented to use top-to-bottom/left-to-right direction in function calls. Without assignments this gives perfect top-to-bottom and left-to-right code reading direction.

```php
// Pipeline
"Hello World"
    |> htmlentities(...)
    |> str_split(...)
    |> (fn($x) => array_map(strtoupper(...), $x))

// Example with some fluent helper 
 helper("Hello World")
    .htmlentities()
    .split()
    .map(fn($x) => strtoupper($x));
```

But things are getting worse, when we add return operator. We need to use top-to-bottom/left-to-righ reading direction for most parts of instruction, but bottom-to-top/right-to-left for last return action.
```php
// Single line case
//   |-------------------------------------------------------4-----------------------------------------------|
//   |        |-----1----|        |-----2-----|     |------3---|                                             |
//   V        |          V        |           V     |          V                                             |
return "Hello World" |> htmlentities(...) |> str_split(...) |> fn($x) => array_map(strtoupper(...), $x)  // -|

// Multiline case
return                              // <---------------------------|
                                    //                             |
    "Hello World"                   //   -1-|                      |
                                    //      |                      |
        |> htmlentities(...)        //   <--|   -2-|               |
                                    //             |               |
        |> str_split(...)           //          <--|    -3-|       |
                                    //                     |       |
        |> (fn($x) => array_map(strtoupper(...), $x)) // <--    -4-|
```

In the case of multiple lines, the situation becomes more complicated for long functions and pipelines, when everything does not fit to one screen and to read one instruction it is necessary to scroll the screen up and down.

The most natural and convenient reading direction for speakers of all LRT languages ​​(including but not limited to all languages ​​derived from Latin, such as English) is from left-to-right and top-to-bottom. With pipelining expression result to `return` keyword it would be posible to write code, that is read and perceived exactly this way, thereby reducing cognitive load.

# Proposal

This RFC introduces of ability to use new pipe assignment operator with `return` keyword.

```php
expr |> return
// equialent to
return expr
```

Example. That the code is always read in one direction, and there is no need to look back.
```php
"Hello World" |> htmlentities(...) |> str_split(...) |> (fn($x) => array_map(strtoupper(...), $x)) |> return;
// or
"Hello World"
    |> htmlentities(...)
    |> str_split(...)
    |> (fn($x) => array_map(strtoupper(...), $x))
    |> return;
```

# Backward Incompatible Changes

* Using `|>` with `return` currently causes syntax error. So it should not create backward incompatible changes.

# RFC Impact

## Perfomance
As new operator is just another syntax for assignemnt operator, it's usage should lead to exactly same bytecode, so no perfomance changes are expected.

## To the Ecosystem
All language tools should be extended to use new syntax.

## To Existing Extensions
Not affected

## To SAPIs
Not affected

# Further scope

The `return` and `throw` keywords are only keywords leading to function exit. Using long chains of pipe operators for exceptions is unlikely pattern. However it is logical to repeat everything that relates to the `return` keyword for the `throw` operator.

On the other hand, since PHP 8 (see https://wiki.php.net/rfc/throw_expression), throw is operator and not statement as `return`. So, operator precedence should be discussed. That's why, defferent RFC is planned for piping to `throw`.