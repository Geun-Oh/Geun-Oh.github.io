+++
title = 'Database 7'
date = 2024-10-08T01:27:48+09:00
draft = false
+++

## SQL Optimization

Before executing SQL, subdivide optimization process is as follows.

1. Parsing SQL

SQL Parser parses SQL when parser gets it from user. To summmarize SQL parsing process, it is as belows.

- Generate parsing tree: generate parsing tree from whole SQL code by analyzing it into individual components.
- Checking syntax: Checks if it has inaccurate syntax error. Like using unavailable or missing keywords, written with wrong sequence...etc.
- Checking semantic: Check if it has inaccurate semantic error. If it used table or column that does not exist. Or has right permissions to use the target object.

2. Optimize SQL

The next step is SQL optimization, and the Optimizer takes the role. SQL Optimizer selects one efficient way from various execution path that generated with precollected statistic information of systems and objects. It's the most important core engine that determines database performance.

3. Generate Row-Sources

The step that SQL Optimizer forms new executable code or procedure with selected execution path. Row-Source Generator takes that role.

## Execution Plan

It's a visible tree structure that user can check optimizer generated procedure. In other words, a preview of procedure.
With preview, user notices whether SQL scans tables or indexes, and if it scans indexes, user can see what indexes they are. Also can change execution path if it operates with unintended way.