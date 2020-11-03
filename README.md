## 1-A
``` sql
SELECT
	U.id,
	C.image_url,
	U.nickname,
	(CASE WHEN S.id IS NULL THEN 'FALSE' ELSE 'TRUE' END) AS is_scrap
FROM users U INNER JOIN cards C ON U.id = C.user_id
LEFT OUTER JOIN scraps S ON C.id = S.card_id;
```
## 1-B
``` sql
SELECT * FROM (
	SELECT
		C.id,
		C.image_url,
		U.nickname,
		(SELECT COUNT(*) FROM scraps WHERE card_id = C.id) AS scrapper_count
	FROM users U INNER JOIN cards C ON U.id = C.user_id
) T
ORDER BY T.scrapper_count DESC;
```
## 1-C-1
``` sql
SELECT 
	T.id,
	T.title,
	T.image_url,
	T.scrap_count
FROM (
	SELECT 
		SB.id,
		SB.title,
		C.image_url,
		(SELECT COUNT(*) FROM scraps WHERE card_id = C.id) AS scrap_count,
		ROW_NUMBER () OVER (PARTITION BY SB.id ORDER BY S.created_at) AS rn
	FROM scrapbooks SB INNER JOIN scraps S ON S.scrapbook_id = SB.id
	INNER JOIN cards C ON C.id = S.card_id
	WHERE SB.user_id = 1
	ORDER BY SB.created_at DESC
) T
WHERE T.rn = 1;
```
## 1-C-2
``` sql 
SELECT 
	T.id,
	T.title,
	T.image_url,
	T.scrap_count
FROM (
	SELECT 
		SB.id,
		SB.title,
		C.image_url,
		(SELECT COUNT(*) FROM scraps WHERE card_id = C.id) AS scrap_count,
		ROW_NUMBER () OVER (PARTITION BY SB.id ORDER BY S.created_at DESC) AS rn
	FROM scrapbooks SB INNER JOIN scraps S ON S.scrapbook_id = SB.id
	INNER JOIN cards C ON C.id = S.card_id
	WHERE SB.user_id = 1
	ORDER BY SB.created_at DESC
) T
WHERE T.rn = 1;
```
## 1-C-3
``` sql
SELECT 
	T.id,
	T.title,
	T.image_url,
	T.scrap_count
FROM (
	SELECT 
		SB.id,
		SB.title,
		C.image_url,
		(SELECT COUNT(*) FROM scraps WHERE card_id = C.id) AS scrap_count,
		ROW_NUMBER () OVER (PARTITION BY SB.id ORDER BY C.created_at DESC) AS rn
	FROM scrapbooks SB INNER JOIN scraps S ON S.scrapbook_id = SB.id
	INNER JOIN cards C ON C.id = S.card_id
	WHERE SB.user_id = 1
	ORDER BY SB.created_at DESC
) T
WHERE T.rn = 1;
```
## 2-A
``` sql
SELECT 
	P.brand_name,
	SUM(O.count) AS buy_count,
	SUM(P.cost * O.count) AS buy_amount
FROM products P INNER JOIN orders O ON P.id = O.product_id
GROUP BY P.brand_name;
```
