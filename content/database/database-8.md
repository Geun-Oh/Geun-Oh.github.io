+++
title = 'Database 8'
date = 2024-10-08T23:17:41+09:00
draft = false
+++

## Analyze sql.pest file

`.pest` file format is for files that are used for parser generator names **Pest**. It usually used in rust ecosystem, appropriate for define various language syntaxes and operates syntax analysis.

And so, Let's break down my `sql.pest` file.

## General Overview

sql.pest
```pest
/////// LIMITATIONS OF CURRENT GRAMMAR
// quote strings only allow certain characters
// column_names, etc. are defined as only allowing alpha 

//////// PUNCTUATIONS
// curly braces
open_brace = { "{" }
close_brace = { "}" }
open_paren = { "(" }
close_paren = { ")" }
comma = { "," }
star = { "*" }
quote = { "\"" }  // a single quote

//////// SPECIAL
WHITESPACE = _{ " " }

///////// DATATYPE
int = { "INT" }
text = { "TEXT" }

///////// VALUE
int_val = @{ ("+" | "-")? ~ ASCII_DIGIT+ }
// legal characters allowed in string
// text parsing will require more thought
char = ${ ASCII_ALPHANUMERIC | "+" | "-" | "." }
quoted_text_val = { quote ~ char* ~ quote }
literal_text_val = @{ char+ }
value = { (int_val | quoted_text_val | literal_text_val) }

///////// SQL KEYWORDS/IDENTIFIERS
// keywords will be case insensitive
// case insensitivity is specified through '^'
create_kw = { ^"CREATE" }
table_kw = { ^"TABLE" }
select_kw = { ^"SELECT" }
from_kw = {  ^"FROM" }
insert_kw = { ^"INSERT" }
into_kw = { ^"INTO" }
values_kw = { ^"VALUES" }

table_name = { (ASCII_ALPHA)+ }

///////// CREATE TABLE DEFINITION
column_name = ${ (ASCII_ALPHA)+ }

column_type = @{ ( int | text ) }
column_def = { column_name ~ column_type }

table_fields = { (column_def ~ comma)* ~ (column_def)? }

create_table_stmnt = { create_kw ~ table_kw ~ table_name ~
                       (open_brace ~ table_fields ~ close_brace) }

////////// SELECT STATEMENT

select_stmnt = { select_kw ~ star ~ from_kw ~ table_name }

////////// INSERT STATMENT

column_names = { (column_name ~ comma)* ~ (column_name)? }
values = { (value ~ comma)* ~ (value)? }

insert_stmnt = { insert_kw ~ into_kw ~ table_name ~ ( open_paren ~ column_names ~ close_paren ) ~
                 values_kw ~ ( open_paren ~ values ~ close_paren ) }

insert_grammar = { SOI ~ insert_kw ~ into_kw ~ table_name ~ ( open_paren ~ column_names ~ close_paren ) ~
                  values_kw ~ ( open_paren ~  values ~ close_paren ) ~ EOI }

////////// UNIFIED GRAMMAR

sql_grammar = {SOI ~ (create_table_stmnt | select_stmnt | insert_stmnt ) ~ EOI }
```

The file defines a grammar for parsing a subset of SQL that focuses on basic commands like `CRAETE`, `TABLE`, `SELECT`, and `INSERT`.
This grammar allows only specified characters, for example, limits strings wrapped with double quote, or enforces `column_names` to allow alphabet characters only.

## Take the grammar into rust code

Install `pest` and `pest_derive`

Cargo.toml
```toml
pest = "2.1"
pest_derive = "2.1"
```

Import crate and derive it into `SQLParser` struct.

```rust
use pest::Parser;

#[derive(Parser)]
#[grammar = "sql.pest"] // .pest defines grammar of SQL 
struct SQLParser;
```

Then we can use our grammar that defined in `sql.pest` in our code like below.

```rust
...
let pairs = SQLParser::parse(Rule::sql_grammar, query)
            .expect("Failed to parse SQL")
            .next()
            .unwrap();
...
```

We can also get the `Rule` from `Parser`. As we can see, `sql_grammar` syntax in `sql.pest` is written in UNIFIED GRAMMAR section. It's for defining top-level rule, set to locate only `create_table_stmnt`, `select_stmnt`, or `insert_stmnt` between `SOI`(start of input) and `EOI`(end of input). So we can notice that this grammar syntax can be composed with one of `CREATE_TABLE`, `INSERT`, and `SELECT`.

So the `sql_grammar` defines SQL query structure, and enforces to allow only queries that fits in it.