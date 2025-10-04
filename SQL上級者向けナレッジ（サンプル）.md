# SQL上級者向けナレッジ（サンプル）
## SQLの基礎
ここはA側の変更
ここもA側の変更
## 1. パフォーマンスチューニング

SQLは書き方によって速度が大きく変わる。

### 実行計画の確認

``` sql
EXPLAIN SELECT * FROM employees WHERE dept_id = 10;
```

-   テーブルスキャン（全件走査）が発生していないか確認\
-   インデックス利用状況をチェック

### チューニングのポイント

-   適切なインデックス設計（複合インデックス、カバリングインデックス）\
-   不要なSELECT \* を避ける\
-   サブクエリよりJOINを検討\
-   LIMITやTOPで必要な件数だけ取得

------------------------------------------------------------------------

## 2. 正規化と非正規化

### 正規化

-   データの重複を排除し、一貫性を保つ設計手法\
-   第1正規形（重複行や複数値を排除）\
-   第2正規形（部分関数従属を排除）\
-   第3正規形（推移的関数従属を排除）

### 非正規化

-   パフォーマンスのために意図的に冗長性を持たせる\
-   例：検索を速くするために集計列を持たせる

------------------------------------------------------------------------

## 3. ストアドプロシージャ

SQL文をまとめてプログラム化し、再利用可能にする。

``` sql
CREATE PROCEDURE transfer_money(
  IN from_id INT,
  IN to_id INT,
  IN amount DECIMAL(10,2)
)
BEGIN
  START TRANSACTION;
  UPDATE accounts SET balance = balance - amount WHERE id = from_id;
  UPDATE accounts SET balance = balance + amount WHERE id = to_id;
  COMMIT;
END;
```

呼び出し例：

``` sql
CALL transfer_money(1, 2, 500.00);
```

------------------------------------------------------------------------

## 4. トリガー（Trigger）

特定のイベントに応じて自動実行される処理。

``` sql
CREATE TRIGGER trg_update_timestamp
BEFORE UPDATE ON employees
FOR EACH ROW
SET NEW.updated_at = NOW();
```

用途：更新履歴の自動管理、監査ログの記録など

------------------------------------------------------------------------

## 5. セキュリティと権限管理

ユーザごとに操作権限を付与・制限できる。

``` sql
GRANT SELECT, INSERT ON employees TO 'appuser'@'localhost';
REVOKE DELETE ON employees FROM 'appuser'@'localhost';
```

-   最小権限の原則（不要な権限は与えない）\
-   ビューを利用して機微データを隠す

------------------------------------------------------------------------

## 6. CTE（共通テーブル式）

WITH句を使ってSQLを整理する。

``` sql
WITH dept_avg AS (
  SELECT dept_id, AVG(salary) AS avg_sal
  FROM employees
  GROUP BY dept_id
)
SELECT e.name, e.salary, d.avg_sal
FROM employees e
JOIN dept_avg d ON e.dept_id = d.dept_id
WHERE e.salary > d.avg_sal;
```

------------------------------------------------------------------------

## 7. ウィンドウ関数

行間で比較やランキングを行う。

``` sql
SELECT name, dept_id, salary,
       RANK() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rank_in_dept
FROM employees;
```

-   RANK(): 同順位を許すランキング\
-   ROW_NUMBER(): 行番号を連番で振る\
-   SUM() OVER(): 累積合計

------------------------------------------------------------------------

## 8. 上級者向け課題

-   大規模データでのパフォーマンスチューニング（パーティション分割、シャーディング）\
-   高可用性設計（レプリケーション、フェイルオーバー）\
-   セキュアなSQL設計（SQLインジェクション対策、権限管理）\
-   ETL/BIとの統合利用

------------------------------------------------------------------------

## 9. 学習の進め方

1.  実行計画を読む習慣をつける\
2.  正規化と非正規化を状況に応じて使い分ける\
3.  ストアドプロシージャやトリガーでDBロジックを管理する\
4.  CTEやウィンドウ関数で高度な集計を習得する\
5.  実案件のパフォーマンス改善に取り組むことで理解を深める
6. test