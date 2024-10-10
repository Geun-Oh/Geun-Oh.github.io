+++
title = 'Database 9'
date = 2024-10-10T23:37:24+09:00
draft = false
+++

## Database storing structure

### Tablespace

Tablespace is a space that storing real data in database object.
It is a physical part of database, allocates repositories to all DBMSs that are managed by segment.

It is for managing massive datas with stable restoring mechanism, and also for disk i/o opt.

### Page

In MySQL, there are many types of datas.

1. Row in Table
2. Index
3. Undo Log

etc..

Row: Real Data that are managed by MySQL.
Index: For efficient row searching.
Undo Log: Datas for transaction operation.

These datas are managed in On-disk sector repository.
It's called a Tablespace(already seen it!)
Row, Index, Undo Logs are stored in each Tablespace.

Page is a bunddle of datas. It also called as "Block of Data".
Page is a logical bunddle of data inside of Tablespace, a huge size of file.
Also, Page is a data unit that MySQL calls at once when Disk i/o operates.

This means that when we search for a particular row, we load all the data pages containing the row at once.

### Types of Page

#### Data Page

is for storing real data.

#### Index Page

Stores information of indexes.

Each index page has index value and data page address that corresponds to it.
Primary index page has one data page that corresponds to, and Secondary index page has two data pages that corresponds to it.

#### Undo Log Page

Undo log page writes down undo log entries.

For implementing Transaction Isolation, Undo log stores the original data for each transaction.
Specially, It is used for implementing REPEATABLE READ at REPEATABLE READ isolation level.