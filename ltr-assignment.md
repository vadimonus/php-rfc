# PHP RFC: Left-to-right assignment operator

Version: 0.0.1

Date: 2025-06-23

Author: Vadim Dvorovenko, vadimon@mail.ru

Status: Draft

Implementations:
    
* https://github.com/vadimonus/php-src/commits/ltr-assign/ 
* https://github.com/vadimonus/php-src/commits/pipe-to-assign/

# Introduction

Left-to-right assignment operator can be used to reduce congnitive complexity when using long chain of pipe operators.

```
// Current syntax
$result = "Hello World" \|> strlen(...);

// New Syntax
"Hello World" |> strlen(...) |> = $result;
// Or
"Hello World" |> strlen(...) |>= $result;
```

# The problem

Pipe operator introduced in RFC (https://wiki.php.net/rfc/pipe-operator-v3) may lead to a complication in the perception of the flow due to mixing of left-to-right and right-to-left action directions. To avoid this, it is proposed to use the pipe operator (or a new similar one) for left-to-right assigning.

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
$result = $temp;                //                          <--|

// Left-to-right example.
// 
//   |--4---|     |----------3---------|     |-2-|        |-1-|  
//   V      |     V                    |     V   |        V   |
$result := array_map(strtoupper(...), str_split(htmlentities("Hello World")));
```

The programmer's brain is tuned to perceive exactly these directions. When we try to reverse this directions, we need to to introduce additional rules, such as idents, to make hints to our brain that we should use bottom-to-top direction instead of top-to-bottom.

```php
$result = 
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
    |> fn($x) => array_map(strtoupper(...), $x)

// Example with some fluent helper 
 helper("Hello World")
    .htmlentities()
    .split()
    .map(fn($x) => strtoupper($x));
```

But things are getting worse, when we add asignment operator. We need to use top-to-bottom/left-to-righ reading direction for most parts of instruction, but bottom-to-top/right-to-left for last assignment action.
```php
// Single line case
//   |-----------------------------------------------------------4-------------------------------------------------|
//   |            |-----1----|        |-----2-----|     |------3---|                                               |
//   V            |          V        |           V     |          V                                               |
$result := "Hello World" |> htmlentities(...) |> str_split(...) |> fn($x) => array_map(strtoupper(...), $x)  // ---|

// Multiline case
$result :=                          // <---------------------------|
                                    //                             |
    "Hello World"                   //   -1-|                      |
                                    //      |                      |
        |> htmlentities(...)        //   <--|   -2-|               |
                                    //             |               |
        |> str_split(...)           //          <--|   -3-|        |
                                    //                    |        |
        |> fn($x) => array_map(strtoupper(...), $x) // <---     -4-|
```

In the case of multiple lines, the situation becomes more complicated for long functions and pipelines, when everything does not fit to one screen and to read one instruction it is necessary to scroll the screen up and down.

The most natural and convenient reading direction for speakers of all LRT languages ​​(including but not limited to all languages ​​derived from Latin, such as English) is from left-to-right and top-to-bottom. With pipeline operator and pipeline assignment operator it would be posible to write code, that is read and perceived exactly this way, thereby reducing cognitive load.

# Proposal

This RFC introduces of ability to use new pipe operator together with equal sign for assignment to variable.

```php
expr |> = $variable
expr |>= $variable
```

which is equialent to
```
$variable = expr
```

Examples:

```php
// Assignment
42 |> = $var;
42 |>= $var;

// Reference assignment
& $var1 |> = $var2;
&$var1 |>= $var2;

// Lists and array assignments
list(1, 2) |> = $arr;
list('a' => 'foo', 'b' => 'bar') |> = $arr;
[1, 2] |> = $arr;
['a' => 'foo', 'b' => 'bar'] |> = $arr;
$arr |> = list($a, $b);
$arr |> = list('a' => $a, 'b' => $b);
$arr |> = [$a, $b];
$arr |> = ['a' => $a, 'b' => $b];

// Adding elements to arrays
1 |> = $arr[];
```

Using left-to-right assignment together with pipe operator
```php
"Hello World"
    |> htmlentities(...)
    |> str_split(...)
    |> (fn($x) => array_map(strtoupper(...), $x))
    |> = $result;
```

# Operator selection

When choosing the form of surgery, I tried and rejected various options.
In this regard, I considered the following things to be especially useful:

* Possibility to create later a family of assignment operators, same as for right-to-left assignment
* Create as few new tokens as possible to leave room for further expansion of the language.
* Try to extend language only by extending grammar for cases that are currently syntax error, to provide full backward compatibility.

Options I rejected and reasons:

* `$a = |> $b` (composite of equal and pipe operator).
  * Since the discussion itself began with a discussion of left-to-right reading, this operator is perceived as an assignment operator when read from left to right, with the expression on the right simply having a very strange meaning. Especially with hyphens, this can lead to a misperception of the logic.
  * This grammar is quite difficult to implement; many different shift/reduce conflicts need to be resolved.
* `$a =|> $b` (allways without space)
  * Requires creation of new tokens.
  * If trying make same tokens for other assignment operators, some of them may look very confusing. E.g. `>>=|>`, `<<=|>`, `|=|>`, `>>|>`, `<<|>`, `||>`. It is better to abandon such strange and rare operators altogether than to bring them into the language.
* `$a |=> $b` (equal inside pipe operator)
  * Requires creation of new tokens.
  * If trying make same tokens for other assignment operators, some of them may look very confusing. E.g. `|>>=>`, `|<<=>`, `||=>`, `|>>>`, `|<<>`, `||>`.
  * If trying to highlight operation, operators may look cumbersome. E.g. `|(>>=)>`, `|(<<=)>`, `|(|=)>`, `|(>>)>`, `|(<<)>`, `|(|)>`, may look very confusing.
* `$a := $b`.
  * In many languages, like Algol, Pascal or Go this is right-to-left assignment operator. Using it as left-to-right assignment may confuse.
* `$a =: $b` (as reverse to `:=`)
  * Same problems as with `$a =|> $b`.
  * We perceive an arrow as a clear indication of the operator's direction. A colon doesn't convey any directional meaning, so this option is inferior to any arrow option.
* `$a |>= $b` (allways without space).
  * Same as for `$a =|> $b`.

So `$a |> = $b` is proposed. If reading from left to right, firstly we read pipe operator, which indicates the direction of action from left to right. After this we see the equality sign, which indicates that the action is assignment. Assignment may be treated as some other type of action, along with the actions performed by the functions, `return` and `throw` keywords.

So it can be threated like a pipe operator extension. This does not require the creation of new tokens; only existing ones are used. When pipe operator is followed by assignment, it works like left-to-right assignment. All other assignment operators can be easily used in the same way. This solves the problem of strange look of shift and or assignment operators.

On the other hand, you can always write two operators without a space, if you find it more beautiful (the same difficult choice as for `$a = & $b` / `$a = &$b` / `$a =& $b`).

# Precedence

Since the proposed operator consists of pipe and assignment operators, it is reasonable to suggest that priority should go to one or the other. The difference in the order of operations is shown in the following example.

Using right-to-left assignment after left-to-right assignment will lead to a syntax violation for both operators.
```php
$a |> = 42 = $b;
//      ^
//      |---- here should be variable

42 |> = $a = 42;
//      ^
//      |---- regardless of precedence, operator with lower priority
//            will allways have here expression, and not a variable, e.g.
//            (42 |> = $a) = 42
//            42 |> = ($a = 42)
```

Thus, only cases where the assignment operator is to the left and the pipe operator is to the right are valid.
```php
$a = 42 |> = $b;
```

The option where assignment operators with different direction and associativity have exactly the same priority and therefore are performed from left to right is impossible. This will lead to shift/reduce conflicts.

Since the operator can be perceived as a pipe operator, it is suggested to use the pipe operator precedence, that is higher, then for assignment.
```php
//(4)  (3)   (1)     (2)
$a = $b = 42 |> = $c |> = $d;
// equal to
$a = ($b = ((42 |> = $c) |> = $d));
```

# In other languages

There have long been known contradictions between mathematicians and programmers regarding the use of the equal sign for the assignment operator and real in the meanings of the expression `a = a + 1`. For example, in Pascal, to resolve this contradiction, `:=` is used for assignment. But only very few languages ​​went further and introduced the left-to-right assignment operator.

* COBOL has has left-to-right assignment statement `MOVE x TO y` (https://www.ibm.com/docs/en/cobol-zos/6.5.0?topic=statements-move-statement) and left-to-right addition assignment statement `ADD x TO y` (https://www.ibm.com/docs/en/cobol-zos/6.5.0?topic=statements-add-statement)
* Casio BASIC and Texas Instruments's BASIC have left-to-right assignment operator → (see in https://en.wikipedia.org/wiki/Casio_BASIC#Examples, https://en.wikipedia.org/wiki/TI-BASIC)
* Linux command line has redirection operator `>` that can be treated as file content assignment (`ls -l | grep ".txt" | wc -l > result`)
* C++ iostream uses >> operator for saving value from stdin to variable
* R language and it's predecessor S language have left-to-right `->` asignment operator alongside with right-to-left `<-` assignment operator.

# Backward Incompatible Changes

* `|> =` operator should not create backward incompatible changes, as currently causes syntax error.

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

Imlementing full familiy of left-to-right assignment operators.

| Existing operator | New operator    |
|-------------------|-----------------|
| `$b += $a`        | `$a \|> += $b`  |
| `$b -= $a`        | `$a \|> -= $b`  |
| `$b *= $a`        | `$a \|> *= $b`  |
| `$b /= $a`        | `$a \|> /= $b`  |
| `$b .= $a`        | `$a \|> .= $b`  |
| `$b /= $a`        | `$a \|> /= $b`  |
| `$b %= $a`        | `$a \|> %= $b`  |
| `$b &= $a`        | `$a \|> &= $b`  |
| `$b \|= $a`       | `$a \|> \|= $b` |
| `$b ^= $a`        | `$a \|> ^= $b`  |
| `$b <<= $a`       | `$a \|> <<= $b` |
| `$b >>= $a`       | `$a \|> >>= $b` |
| `$b ??= $a`       | `$a \|> ??= $b` |
| `$b **= $a`       | `$a \|> **= $b` |
