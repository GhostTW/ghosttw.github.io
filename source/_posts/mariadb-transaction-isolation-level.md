---
title: MariaDb 在不同 isolation level 下的行為及解決的問題
date: 2019-12-08 20:54:26
tags: ['mariadb', 'database']
categories:
---

此練習使用 Sql 與 MySqlWorkBench 練習實作不同的 Isolation Level 對資料造成的影響，分別有 Dirty Data, Non-Repeatable, Phantom Data 的問題．

## 實驗資料庫的建立

相關程式碼皆在 [demo repo](https://github.com/GhostTW/demos) 下的 `demo-storage-scenarios` 資料夾
使用範例程式資料夾 `database` 下的 `docker-compose` 啟動設定好的實驗資料庫

`docker-compose up TestDB`

## 00 Transaction Commit

基本 Transaction 使用並 Commit．

```sql
# step 1
SELECT * FROM User;
```

原始資料

{% asset_img default-users.png default-users %}

```sql
# step 2
START TRANSACTION;
UPDATE User SET IsActive = 0 WHERE Code = 'Admin';
COMMIT;
SELECT * FROM User;
```

結果

{% asset_img commit-result.png commit-result %}

## 01 Transaction Rollback

基本 Transaction 使用並 Rollback 放棄變更．

```sql
# step 1
SELECT * FROM User;

# step 2
START TRANSACTION;
UPDATE User SET IsActive = 1 WHERE Code = 'Admin';
SELECT * FROM User;

# step 3
ROLLBACK;
SELECT * FROM User;
```

{% asset_img rollback-result.png rollback-result %}

## 02 Check current session status

檢查目前連線 session 的 isolation level 與有沒有執行中的 transaction．

```sql
# step 1 get current session isolation level setting.
SELECT @@TX_ISOLATION;

# step 2
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SELECT @@TX_ISOLATION;
```

{% asset_img check-isolation-level.png check-isolation-level %}

```sql
# step 3
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;

# step 4 get current session working transaction.
SELECT @@IN_TRANSACTION;
```

{% asset_img check-transaction.png check-transaction %}

## 03 DirtyRead

在 MySqlWorkBench 開啟兩個 Session 連線至資料庫，使用不同的 isolation level 同時操作一樣的資料．
Session B 更改資料，但在未送出 commit 前被 SessionA 使用 ReadUncommitted 讀到髒資料．

* Step 1 Session A

```sql
# step 1
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SELECT @@tx_isolation;
SELECT * FROM User;
```

{% asset_img read-uncommitted.png read-uncommitted %}
{% asset_img default-users.png default-users %}

* Step 2 SessionB
建立一個 transaction 但不結束，觀察另一個 session 的使用狀況．

```sql
START TRANSACTION;
UPDATE User SET IsActive = 0 WHERE Code = 'Admin';
```

* Step 3 SessionA
讀取到尚未 commit 的資料.

```sql
SELECT * FROM User WHERE Code = 'Admin;
```

`IsActive: 0`

* Step 4 SessionA
改變 SessionA 的 isolation level 驗證該交易只能拿到 committed 過的資料．

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT @@tx_isolation;
SELECT * FROM User WHERE Code = 'Admin;
```

`IsActive: 1`

* Step 5 SessionB
將 transaction commit 送出

```sql
COMMIT;
```

* Step 6 SessionA get committed data.
SessionA 會取得 commit 後的資料．

```sql
SELECT * FROM User WHERE Code = 'Admin;
```

`IsActive: 0`

## 04 NonRepeatable Read

當兩個交易同時發生，交易 A 讀取完值後該值被交易 B 更改掉，但交易 A 仍未結束，會導致第二次讀取同一個值是已被交易 B 更改的．當資料前後不一致時，可能會造成邏輯判斷的錯誤．

* Step 1 SessionA

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT @@tx_isolation;
START TRANSACTION;
SELECT * FROM User;
```

`Password: 1E867FA1A3A64AB5E1EE21BD76F05912`

* Step 2 SessionB

```sql
START TRANSACTION;
UPDATE User SET Password = '0' WHERE Code = 'Test001';
COMMIT;
```

`Password: 0`

* Step 3 SessionA
更新時與第一次不一樣的值，結果仍是 SessionB 改的 0

```sql
SELECT * FROM User;
COMMIT;
SELECT * FROM User;
```

commit 前第二次讀取會取到被更改的值．
`Password: 0`
`Password: 0`

* Step 4 SessionA
使用 REPEATABLE READ 防止此情況發生

```sql
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT @@tx_isolation;
UPDATE User SET Password = '1E867FA1A3A64AB5E1EE21BD76F05912' WHERE Code = 'Test001';
START TRANSACTION;
SELECT * FROM User;
```

`Password: 1E867FA1A3A64AB5E1EE21BD76F05912`

* Step 5 SessionB

```sql
START TRANSACTION;
UPDATE User SET Password = '1' WHERE Code = 'Test001';
COMMIT;
SELECT * FROM User;
```

`Password: 1`

* Step 6 SessionA

```sql
SELECT * FROM User;、
COMMIT;
SELECT * FROM User;
```

commit 前第二次讀取會取到跟第一次一樣的值．

`Password: 1E867FA1A3A64AB5E1EE21BD76F05912`
`Password: 1`

* Step 7 SessionA
使用 `lock in share mode` 讓 SessionB 等待

```sql
UPDATE User SET Password = '1E867FA1A3A64AB5E1EE21BD76F05912' WHERE Code = 'Test001';
START TRANSACTION;
SELECT * FROM User LOCK IN SHARE MODE;
```

* Step 8 SessionB

```sql
START TRANSACTION;
UPDATE User SET Password = '1' WHERE Code = 'Test001';
COMMIT;
```

* Step 9 SessionA

```sql
COMMIT;
```

## 05 Phantom Read

當兩個交易進行時，Ｂ交易對資料做新增或刪除時，Ａ交易不會知道有關新增刪除的資料．

* Step 1 SessionA
原始資料有三筆

```sql
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT @@tx_isolation;
UPDATE User SET IsActive = 1 WHERE Code = 'Admin';

START TRANSACTION;
SELECT * FROM User;
```

* Step 2 SessionB
新增一筆資料

```sql
START TRANSACTION;
INSERT INTO User VALUES(4,'testP0', '1E867FA1A3A64AB5E1EE21BD76F05912', 1);
COMMIT;
```

* Step 3 SessionA
仍是取到三筆資料

```sql
SELECT * FROM User;
COMMIT;
```

* Step 4 SessionA
使用 Serialiazable

```sql
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT @@tx_isolation;
START TRANSACTION;
SELECT * FROM User;
```

* Step 5 SessionB
新增一筆資料，但因為是 Serializable 進入等待．

```sql
START TRANSACTION;
INSERT INTO User VALUES(5, 'testP1', '1E867FA1A3A64AB5E1EE21BD76F05912', 1);
COMMIT;
SELECT * FROM User;
```

* Step 6 SessionA
結束 transaction 後，交易 B 才將資料新增進去．

```sql
COMMIT;
SELECT * FROM User;
```
