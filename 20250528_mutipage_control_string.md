```SQL
SELECT 
    M.MB001,
    M.MB002,
    --M.MB003,
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
