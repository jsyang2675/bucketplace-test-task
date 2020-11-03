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
