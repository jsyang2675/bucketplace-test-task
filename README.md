## 1-A
``` sql
SELECT
	U.id,
	C.image_url,
	U.nickname,
	(CASE WHEN S.id IS NULL THEN 'FALSE' ELSE 'TRUE' END) AS is_scrap
FROM users U 
INNER JOIN cards C ON C.user_id = U.id
LEFT OUTER JOIN scraps S ON S.card_id = C.id;
```
## 1-B
``` sql
SELECT * FROM (
	SELECT
		C.id,
		C.image_url,
		U.nickname,
		(SELECT COUNT(*) FROM scraps WHERE card_id = C.id) AS scrapper_count
	FROM users U 
	INNER JOIN cards C ON C.user_id = U.id
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
	FROM scrapbooks SB
	INNER JOIN scraps S ON S.scrapbook_id = SB.id
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
	FROM scrapbooks SB 
	INNER JOIN scraps S ON S.scrapbook_id = SB.id
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
	FROM scrapbooks SB 
	INNER JOIN scraps S ON S.scrapbook_id = SB.id
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
FROM products P 
INNER JOIN orders O ON O.product_id = P.id
GROUP BY P.brand_name;
```
## 2-B
``` sql
SELECT 
	U.id,
	U.nickname,
	NVL(PO.buy_count,0) AS buy_count,
	NVL(PO.buy_amount,0) AS buy_amount,
	(CASE WHEN NVL(buy_count,0) >= 4 AND NVL(buy_amount,0) >= 1000000 THEN 'Platinum'
	 WHEN NVL(buy_count,0) >= 3 AND NVL(buy_amount,0) >= 500000 THEN 'VIP'
	 WHEN NVL(buy_count,0) >= 2 AND NVL(buy_amount,0) >= 300000 THEN 'Friend' ELSE 'Normal' END) AS rating
FROM users U 
LEFT OUTER JOIN
(
	SELECT 
		O.user_id,
		SUM(O.count) AS buy_count,
		SUM(P.cost * O.count) AS buy_amount
	FROM products P
	INNER JOIN orders O ON O.product_id = P.id
	WHERE O.created_at >= ADDDATE(NOW(), INTERVAL -6 MONTH)
	GROUP BY O.user_id
) PO ON PO.user_id = U.id;
```
## 2-C
``` sql
SELECT 
	SUM(O.count) AS buy_count,
	SUM(P.cost * O.count) AS buy_amount
FROM products P 
INNER JOIN orders O ON O.product_id = P.id 
INNER JOIN product_categories PC ON PC.product_id = P.id 
INNER JOIN categories C ON C.id = PC.category_id
WHERE C.`first` = '가구'
AND C.`second` = '의자';
```
## 2-D
``` sql
SELECT 
	T.name,
	SUM(T.count) AS buy_count,
	SUM(T.cost * T.count) AS buy_amount
FROM (
	SELECT
		C.`second` as name,
		P.cost,
		O.count,
		ROW_NUMBER () OVER (PARTITION BY PC.product_id, PC.category_id ORDER BY PC.`position`) AS rn
	FROM products P 
	INNER JOIN orders O ON O.product_id = P.id 
	INNER JOIN product_categories PC ON PC.product_id = P.id
	INNER JOIN categories C ON C.id = PC.category_id
	WHERE C.`first` = '가구'
) T
WHERE T.rn = 1
GROUP BY T.name;
```
## 3-A
![오늘의집_DB설계](https://user-images.githubusercontent.com/44989888/98084187-cb65d480-1ebe-11eb-8908-24ab8e243f7d.png)
## 4-A
``` java
import java.util.Arrays;
import java.util.Scanner;

public class AlgorithmTest01 {
    public static void main(String[] args) {
        try {
            Scanner scanner = new Scanner(System.in);

            System.out.println("배열의 사이즈를 입력하세요");
            int length = getInputNumber(scanner);

            Integer[] numberA = new Integer[length];
            Integer[] numberB = new Integer[length];

            String textArrayA = "";
            for(int i=0; i<length; i++) {
                System.out.println("A배열의 " + (i+1) + "번 째 값을 입력해 주세요.");
                numberA[i] = getInputNumber(scanner);
                textArrayA += numberA[i];
                if(i<length-1) textArrayA+=",";
            }

            String textArrayB = "";
            for(int i=0; i<length; i++) {
                System.out.println("B배열의 " + (i+1) + "번 째 값을 입력해 주세요.");
                numberB[i] = getInputNumber(scanner);
                textArrayB += numberB[i];
                if(i<length-1) textArrayB+=",";
            }

            System.out.println("A = [" + textArrayA + "] B = [" + textArrayB + "]");

            //입력 값 순차적으로 정렬
            Arrays.sort(numberA);
            Arrays.sort(numberB);

            int totalMin = 0;
            int indexA = 0;
            int indexB = length-1;
            while (indexA <= length-1) {
                totalMin += numberA[indexA] * numberB[indexB];
                indexA++;
                indexB--;
            }

            System.out.println("결과 값은 " + totalMin + "입니다.");

        }
        catch (IllegalArgumentException e) {
            System.out.println(e.getMessage());
        }
    }

    public static int getInputNumber(Scanner scanner) {
        if(scanner.hasNextInt()) {
            int number = scanner.nextInt();
            if(number <= 0 || number > 10) {
                throw new IllegalArgumentException("1~10 사이의 숫자만 입력 가능합니다. 종료합니다.");
            }
            return number;
        }
        else {
            throw new IllegalArgumentException("1~10 사이의 숫자만 입력 가능합니다. 종료합니다.");
        }
    }
}
```

