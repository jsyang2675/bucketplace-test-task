### 사용언어 & 툴
> 1~2. sql쿼리문작성 : mysql<br>
> 3. db테이블설계 사용 툴 : exerd<br>
> 4. 알고리즘 : java, 1.5버젼이상<br>

## 1-A
#### 전체 사진 리스트를 가져올때, 특정 사용자가 리스팅된 사진을 스크랩 했는지를 함께 알 수 있는 SQL을 작성해주세요
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
#### 사진 리스트를 스크랩한 사용자 수에 대하여 내림차순으로 정렬해주세요
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
#### 가장 처음 스크랩한 사진
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
#### 가장 마지막에 스크랩한 사진
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
#### 가장 마지막에 업로드된 사진
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
#### 브랜드 별 상품 매출데이터(구매횟수, 구매금액)를 구해주세요
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
#### 사용자 리스트를 구매 실적(누적구매횟수, 누적구매금액) 및 다음 등급을 보여주는 쿼리를 작성해주세요
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
#### 특정 카테고리 Input이 주어질때 해당 카테고리의 매출데이터(누적구매횟수, 누적구매금액)을 구해주세요
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
#### 메인 카테고리 Input이 주어질때 해당 카테고리의 서브 카테고리별 매출데이터(누적구매횟수, 누적구매금액)를 구해주세요
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
#### 요구사항 및 추가요구사항에 대한 데이터베이스 테이블 설계
![오늘의집_DB설계](https://user-images.githubusercontent.com/44989888/98084187-cb65d480-1ebe-11eb-8908-24ab8e243f7d.png)
## 4-A-1
#### 같은 길이의 A,B배열에 대해 각각 한 개의 숫자를 뽑아 두 수를 곱한 후 누적해서 최소 값 반환하는 함수
``` java
import java.util.Arrays;
import java.util.Scanner;

public class AlgorithmTest01 {
    public static void main(String[] args) {
        try {
            Scanner scanner = new Scanner(System.in);

            //입력 값 제한 (최대 32,767까지 변경 가능)
            short maxInputNum = 10;

            System.out.println("배열의 사이즈를 입력하세요");
            int length = getInputNumber(scanner, maxInputNum);

            Integer[] numberA = new Integer[length];
            Integer[] numberB = new Integer[length];

            String textArrayA = "";
            for(int i=0; i<length; i++) {
                System.out.println("A배열의 " + (i+1) + "번 째 값을 입력해 주세요.");
                numberA[i] = getInputNumber(scanner, maxInputNum);
                textArrayA += numberA[i];
                if(i<length-1) textArrayA+=",";
            }

            String textArrayB = "";
            for(int i=0; i<length; i++) {
                System.out.println("B배열의 " + (i+1) + "번 째 값을 입력해 주세요.");
                numberB[i] = getInputNumber(scanner, maxInputNum);
                textArrayB += numberB[i];
                if(i<length-1) textArrayB+=",";
            }

            System.out.println("A = [" + textArrayA + "] B = [" + textArrayB + "]");

            //입력 값 순차적으로 정렬
            Arrays.sort(numberA);
            Arrays.sort(numberB);

            /**
             * A제일작은수 * B제일큰수 + A두번째작은수 * B두번째큰수 + ...
             */
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

    public static int getInputNumber(Scanner scanner, short maxNum) {
        if(scanner.hasNextInt()) {
            int number = scanner.nextInt();
            if(number <= 0 || number > maxNum) {
                throw new IllegalArgumentException("1~"+maxNum+" 사이의 숫자만 입력 가능합니다. 종료합니다.");
            }
            return number;
        }
        else {
            throw new IllegalArgumentException("1~"+maxNum+" 사이의 숫자만 입력 가능합니다. 종료합니다.");
        }
    }
}
```
## 4-A-2
#### A팀 순서가 고정된 개발자 실력 싸움에서, B팀이 최대 승리를 가져갈 수 있는 함수
``` java
import java.util.Scanner;

public class AlgorithmTest02 {
    public static void main(String[] args) {
        try {
            Scanner scanner = new Scanner(System.in);

            //입력 값 제한 (최대 32,767까지 변경 가능)
            short maxInputNum = 100;

            System.out.println("배열의 사이즈를 입력하세요");
            int length = getInputNumber(scanner, maxInputNum);

            Integer[] teamA = new Integer[length];
            Integer[] teamB = new Integer[length];

            for (int i = 0; i < length; i++) {
                System.out.println("A팀의 " + (i + 1) + "번 째 개발자 게임실력을 입력해주세요.");
                teamA[i] = getInputNumber(scanner, maxInputNum);
            }

            for (int i = 0; i < length; i++) {
                System.out.println("B팀의 " + (i + 1) + "번 째 개발자 게임실력을 입력해주세요.");
                teamB[i] = getInputNumber(scanner, maxInputNum);
            }

            /**
             * A팀의 실력 순서대로 승리 할 수 있는 B팀의 최소 값으로 비교
             * ex) A : 10,11,2,14
             *     B : 1,11,1,12
             *     인 경우, A팀 10을 이길 수 있는 11,12 중 11로 배치해서 2승을 챙긴다.
             *     12로 배치할 경우 1승 밖에 못 챙긴다.
             */
            int winCount = 0;
            Integer[] compare = new Integer[length];
            for (int a = 0; a < length; a++) {

                //B팀이 이길 수 있는 개발자 담기 (B팀개발자 - A팀 a번째 개발자 = 0보다 클 때 승리 가능)
                for (int b = 0; b < length; b++) {
                    compare[b] = Math.max(teamB[b] - teamA[a], 0);
                }

                //승리 할 수 있는 개발자 중 한 명 담아두기
                int winner = 0;
                int position = -1;
                for (int b = 0; b < length; b++) {
                    if (compare[b] > 0) {
                        winner = compare[b];
                        position = b;
                        break;
                    }
                }

                //승리 할 수 있는 개발자가 2명 이상 있는 경우, 격차가 더 적은 개발자로 승리하기 위해 비교
                if (winner > 0) {
                    for (int b = 0; b < length; b++) {
                        if (compare[b] > 0 && winner > compare[b]) {
                            winner = compare[b];
                            position = b;
                        }
                    }
                    if (position >= 0) {
                        //승리 한 개발자는 중복 출전이 불가능 하므로 제외
                        teamB[position] = 0;
                        winCount++;
                    }
                }
            }

            System.out.println("백앤드 개발팀(B)은 최대 " + winCount + "번의 게임에서 승리가 가능합니다.");

        }
        catch (IllegalArgumentException e) {
            System.out.println(e.getMessage());
        }
    }

    public static int getInputNumber(Scanner scanner, short maxNum) {
        if(scanner.hasNextInt()) {
            int number = scanner.nextInt();
            if(number <= 0 || number > maxNum) {
                throw new IllegalArgumentException("1~"+maxNum+" 사이의 숫자만 입력 가능합니다. 종료합니다.");
            }
            return number;
        }
        else {
            throw new IllegalArgumentException("1~"+maxNum+" 사이의 숫자만 입력 가능합니다. 종료합니다.");
        }
    }
}
```
