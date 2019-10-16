[![CircleCI](https://circleci.com/gh/shigeru0215/sqlint/tree/develop.svg?style=svg)](https://circleci.com/gh/shigeru0215/sqlint/tree/develop)

# sqlint
This is a SQL parser and linter for Standard SQL(BigQuery).

## Install

pip, 

```bash
$ pip install sqlint
```

repository, 

```bash
$ git clone git@github.com:shigeru0215/sqlint.git
$ cd sqlint
$ python setup.py install
```

 - if you use pyenv,

```bash
$ pyenv rehash
```

## Usage

Command line

```bash
$ sqlint query/*sql
```

REPL

```bash
$ python
>>> from sqlint import parse, check
>>> stmt = 'SELECT id From user_table  where user_table.age >10'
>>>
>>> parse(stmt)
[[<Keyword: 'SELECT'>, <Whitespace: ' '>, <Identifier: 'id'>, <Whitespace: ' '>, <Keyword: 'From'>, <Whitespace: ' '>, <Identifier: 'user_table'>, <Whitespace: '  '>, <Keyword: 'where'>, <Whitespace: ' '>, <Identifier: 'user_table.age'>, <Whitespace: ' '>, <Operator: '>'>, <Identifier: '10'>]]
>>>
>>> check(stmt)
['(L1, 1): reserved keywords must be lower case: SELECT -> select', '(L1, 11): reserved keywords must be lower case: From -> from', '(L1, 26): too many spaces', '(L1, 49): whitespace must be after binary operator: >10']
```

Dockerfile

```bash
$ docker build -t sqlint:latest .
 ...
$ docker run -it sqlint:latset /bin/bash
xxxxx:/work # python3 -m sqlint sqlint/tests/data/query005.sql 
sqlint/tests/data/query005.sql:(L2, 6): comma must be head of line
sqlint/tests/data/query005.sql:(L7, 6): comma must be head of line
```

## Checking variations

Check if sql statement violates following rules.

- indent steps are N multiples (default: N = 4).

- duplicated whitespaces except indent.

- duplicated blank lines.

- reserved keywords is capital case or not (default: not capital).

- comma is head(or end) of the line which connects some columns or conditions (default: head).

- a whitespace are not before `)` or after `(`.

- a whitespace is before and after binary operators.
  - (e.g.) `=`, `<`, `>`, `<=`. `>=`. `<>`, `!=`, `+`, `-`, `*`, `/`, `%`

- the table name is at the same line as join context.

- join contexts are written fully, for example `left outer join, `inner join or `cross join`.

- whether new line starts at 'on', 'or', 'and' context (except `between`).

## Futures
- table_name alias doesn't equal reserved functions
- indent appropriately in reserved keywords.
- the order of conditions(x, y) at 'join on x = y'
- Optional: do not use sub-query
- Optional: do use hard-coding constant

## Sample

```
$ sqlint sqlint/tests/samples/*
tests/samples/query001.sql (L2, 1): indent steps must be 4 multiples, but 5 spaces
tests/samples/query002.sql (L6, 16): there are multiple whitespaces
tests/samples/query003.sql (L2, 7): whitespace must not be after bracket: ( 
tests/samples/query003.sql (L2, 22): whitespace must not be before bracket:  )
tests/samples/query004.sql (L3, 8): whitespace must be before binary operator: b+
tests/samples/query004.sql (L3, 8): whitespace must be after binary operator: +c
tests/samples/query005.sql (L2, 6): comma must be head of line
tests/samples/query005.sql (L7, 6): comma must be head of line
tests/samples/query006.sql (L1, 1): reserved keywords must be lower case: SELECT -> select
tests/samples/query006.sql (L3, 7): reserved keywords must be lower case: Count -> count
sqlint/tests/data/query007.sql:(L8, 16): table_name must be at the same line as join context
sqlint/tests/data/query008.sql:(L6, 5): join context must be [left outer join], [inner join] or [cross join]: join
sqlint/tests/data/query008.sql:(L10, 10): join context must be [left outer join], [inner join] or [cross join]: left join
sqlint/tests/data/query008.sql:(L14, 11): join context must be [left outer join], [inner join] or [cross join]: right join
sqlint/tests/data/query008.sql:(L16, 17): join context must be [left outer join], [inner join] or [cross join]: right outer join
sqlint/tests/data/query009.sql:(L6, 0): too many blank lines (2)
sqlint/tests/data/query009.sql:(L10, 0): too many blank lines (2)
sqlint/tests/data/query010.sql:(L6, 35): break line at 'and', 'or', 'on': on
sqlint/tests/data/query010.sql:(L11, 29): break line at 'and', 'or', 'on': and
sqlint/tests/data/query010.sql:(L12, 14): break line at 'and', 'or', 'on': or
```
