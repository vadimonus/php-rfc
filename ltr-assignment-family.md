# PHP RFC: Left to right assignment operator

Version: 0.0.1

Date: 2025-06-23

Author: Vadim Dvorovenko, vadimon@mail.ru

Status: Draft

Implementation: https://github.com/vadimonus/php-src/commits/pipe-to-assign/

# Proposal

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
