
### 跨table 處理字串 兼比對內容  堆壘回傳可能的內容
```SQL
SELECT 
    M.MB001,
    M.MB002,

    STUFF((
        SELECT DISTINCT ', ' + E.MODEL_NO
        FROM TRx_SPEC_EEPROM E
        WHERE LEFT(E.MODEL_NO COLLATE Chinese_Taiwan_Stroke_CI_AS, 13) = 
              LEFT(
                    CASE 
                        WHEN LEFT(M.MB002, 4) = 'TRX ' THEN STUFF(M.MB002, 1, 4, '')
                        ELSE M.MB002 
                    END 
				COLLATE Chinese_Taiwan_Stroke_CI_AS, 13)
        FOR XML PATH(''), TYPE).value('.', 'NVARCHAR(MAX)'), 1, 2, '') AS matched_TRx_SPEC_EEPROM,

	M.MB003

FROM [FORMERICA].[dbo].[INVMB] M
WHERE M.MB001 LIKE '93%' AND M.MB002 NOT LIKE '%失效%' AND M.MB002 NOT LIKE '%作廢%'
ORDER BY M.MB002

```  

---
### 外層主查詢
主要是應付  `[FORMERICA].[dbo].[INVMB]` 且 改名為 `M`
```SQL
SELECT 
    M.MB001,
    M.MB002,
    M.MB003,
    ...
FROM [FORMERICA].[dbo].[INVMB] M
WHERE M.MB001 LIKE '93%' 
  AND M.MB002 NOT LIKE '%失效%' 
  AND M.MB002 NOT LIKE '%作廢%'
```

--- 
### SELECT DISTINCT ...外包語法全合併一排（子查詢）
```SQL
STUFF((
    SELECT DISTINCT ', ' + E.MODEL_NO
    FROM TRx_SPEC_EEPROM E
    WHERE LEFT(E.MODEL_NO COLLATE Chinese_Taiwan_Stroke_CI_AS, 13) = 
          LEFT(
              CASE 
                  WHEN LEFT(M.MB002, 4) = 'TRX ' THEN STUFF(M.MB002, 1, 4, '')
                  ELSE M.MB002 
              END COLLATE Chinese_Taiwan_Stroke_CI_AS, 
              13
          )
    FOR XML PATH(''), TYPE).value('.', 'NVARCHAR(MAX)'), 1, 2, '')
```

---
這是取出不重複的 且每個row前面都加 `, `
```SQL
SELECT DISTINCT ', ' + E.MODEL_NO
    FROM TRx_SPEC_EEPROM E
```
後續再追加一個`where`
```SQL
SELECT DISTINCT ', ' + E.MODEL_NO
    FROM TRx_SPEC_EEPROM E
    WHERE ......這邊有成立才會形成一行  不成立就沒有
```
加上`FOR XML PATH('')` `TYPE` 
```SQL
SELECT DISTINCT ', ' + E.MODEL_NO
    FROM TRx_SPEC_EEPROM E
    where E.MODEL_NO like '%T1%'

    FOR XML PATH(''), TYPE
```
為了不能裸露文字  要再外面包個`select`
```SQL
select(SELECT DISTINCT ', ' + E.MODEL_NO
        FROM TRx_SPEC_EEPROM E
		where E.MODEL_NO like '%T1%'

		FOR XML PATH(''), TYPE).value('.', 'NVARCHAR(MAX)')
```

---
其中 `CASE..WHEN..ELSE..END`  
就是 一般程式語言的 `if else`
```SQL
CASE
    WHEN 條件1 THEN 結果1
    WHEN 條件2 THEN 結果2
    ELSE 預設結果
END
```

---
| 語法                   | 用途            |
| -------------------- | ------------- |
| `FOR XML PATH('')`   | 將多筆資料合併為一列    |
| `TYPE`               | 防止 XML 編碼轉義   |
| `.value('.', '...')` | 從 XML 中抽出純文字  |
| `STUFF(...)`         | 移除開頭多餘字元（如逗號） |

---
### STUFF(M.MB002, 1, 4, '')
STUFF 是 SQL Server 中的字串處理函數，用來替換字串的一部分。  
這裡的意思是：從 M.MB002 的第 1 個字元開始，刪除 4 個字元，並用空字串 ('') 取代。  
效果就是：把 'TRX ' 去掉。

---
### `STUFF(為何這邊還可以插入一個select, 1, 4, '')`
因為 SQL Server 中的 STUFF() 函數接受的是一個字串類型的參數，  
而只要你傳給它的是一個可以回傳字串的東西  
例如 `SELECT ... FOR XML PATH('')`
它就能處理。

---
### <表達式> AS <別名>
SQL 中的 欄位別名（alias）用法，用來給查詢結果中的欄位取一個易讀的名稱