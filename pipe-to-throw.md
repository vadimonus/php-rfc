# PHP RFC: Pipe to throw

Version: 0.0.1

Date: 2026-01-08

Author: Vadim Dvorovenko, vadimon@mail.ru

Target Version: PHP 8.6

Status: Draft

Implementation: https://github.com/vadimonus/php-src/commits/pipe-to-throw

# Introduction

Piping result of expression to `throw` keyword can be used to reduce congnitive complexity. Piping to throw is needed for consistency with piping to `return`.

```php
// Current syntax
throw new Exception();

// New Syntax
new Exception() |> throw;
```

# The problem

The `return` and `throw` keywords are only keywords leading to function exit. So behaviour between theese keywords expected to be consistent.

# Proposal

This RFC introduces of ability to use new pipe assignment operator with `throw` keywords.

```php
expr |> throw
// equialent to
throw expr
```

## Examples

That the code is always read in one direction, and there is no need to look back.
```php
$exceptionBuilder = fn($message) => new Exception($message);
"Exception message" |> $exceptionBuilder |> throw;
// or
"Exception message"
    |> (fn($message) => new Exception($message))
    |> throw;
```

## Precedence

Precedence for pipe to throw is selected same as for pipe fo callable operator. It is expected that it will lead to less less confusion, because pipe operators are expected to be used in chain, and it would be strange if last (piping to throw) will have other precedence. 

# Backward Incompatible Changes

* Using `|>` with `throw` currently causes syntax error. So it should not create backward incompatible changes.

# RFC Impact

## Perfomance
As new operator is just another syntax for assignemnt operator, it's usage should lead to exactly same bytecode, so no perfomance changes are expected.

## To the Ecosystem
All language tools should be extended to use new syntax.

## To Existing Extensions
Not affected

## To SAPIs
Not affected

# References

* https://wiki.php.net/rfc/throw_expression