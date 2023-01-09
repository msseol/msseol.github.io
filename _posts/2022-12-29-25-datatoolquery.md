---
layout: post
title:  "게임 기획 데이터 관리 프로세스 정리"
date:   2022-12-29 00:01:09
categories: DATABASE
tags: game excel data
cover: 25_1.png
side: true
---

<i class="fa-regular fa-circle-check" style="margin-right:0.7rem"></i>*게임 엑셀 데이터를 디비 적용 시 필요한 것들에 대한 정리*

---

게임서비스에서 기획자들이 작성한 게임 기획 데이터들을 적용하는 건 생각보다 복잡한 프로세스로 동작하고 있다. 단순히 엑셀을 DB에 밀어넣는 작업만 하면 단순한데, 
안정성 있는 데이터를 게임에 제공하기 위해서는 작업 로그나 무결성 체크 등등 복잡한 데이터 검증과 처리 프로세스들이 들어가야 한다.   
   
   
아래 프로세스는 모 게임사에 재직할 때의 처리 프로세스를 기억에 의존하여 도식화 한 그림이다. 몇 가지 누락되었을 수도 있지만, 대략 어떤 과정들이 들어가기도 하는구나 정도로 이해하기엔 충분한 그림이다.

<a href="/assets/images/25_1.png" data-lightbox="falcon9-large" data-title="플로우">
  <img src="/assets/images/25_1.png" title="플로우">
</a>
<em>그림1. A게임 데이터 처리 플로우 ([PlantUml][umllink])</em>

관련해서 처리에 필요한 쿼리를 살펴보자. 쿼리는 mssql 기준으로 작성되었다.

#### 테이블 스키마 조회

해당 쿼리를 통해 **테이블 존재 여부**, **컬럼명**이나 **타입**을 비교하여 데이터를 검증할 수 있다. 이런 류의 스키마 조회 쿼리는 검색해보면 많이 나오므로 간단해 보이는 것으로 사용해도 된다.

```
SELECT T.[NAME] AS TABLENAME,  
SUBSTRING(COLUMN_NAMES, 1, LEN(COLUMN_NAMES)-1) AS [COLUMNS] 
FROM SYS.OBJECTS T WITH(NOLOCK) 
LEFT OUTER JOIN SYS.INDEXES I WITH(NOLOCK) 
ON T.OBJECT_ID = I.OBJECT_ID 
CROSS APPLY (
    SELECT COL.[NAME] + ',' 
    FROM SYS.INDEX_COLUMNS IC WITH(NOLOCK) 
    INNER JOIN SYS.COLUMNS COL WITH(NOLOCK) 
        ON IC.OBJECT_ID = COL.OBJECT_ID 
        AND IC.COLUMN_ID = COL.COLUMN_ID 
    WHERE IC.OBJECT_ID = T.OBJECT_ID 
        AND IC.INDEX_ID = I.INDEX_ID 
    ORDER BY COL.COLUMN_ID 
    FOR XML PATH ('') 
) D (COLUMN_NAMES) 
WHERE IS_UNIQUE = 1 
AND T.IS_MS_SHIPPED <> 1 
AND I.[TYPE] = 2
```

> mysql도 INFORMATION_SCHEMA를 참조하면 된다.

#### 모든 테이블 목록 및 상세 스키마

```sql
SELECT  
    AA.TABLE_NAME AS TableName, 
    (SELECT 
            A.TABLE_NAME AS TableName 
        , A.COLUMN_NAME AS ColumnName 
        , A.DATA_TYPE AS DataType 
        , ISNULL(ISNULL(CAST(A.CHARACTER_MAXIMUM_LENGTH AS VARCHAR(10)), CAST(A.NUMERIC_PRECISION AS VARCHAR(10))), '-2') AS ColumnLength 
        , ISNULL(A.COLUMN_DEFAULT, '') AS ColumnDefault 
        , A.IS_NULLABLE AS IsNullable 
        , ISNULL(B.VALUE, '') AS ColumnComment 
        , CASE WHEN D.COLUMN_NAME IS NULL THEN 'N' ELSE 'Y' END AS PrimaryKey 
        , CASE WHEN E.Colum IS NULL THEN 'N' ELSE 'Y' END AS [Identity] 
    FROM 
        INFORMATION_SCHEMA.COLUMNS A WITH(NOLOCK) 
    LEFT OUTER JOIN 
        SYS.EXTENDED_PROPERTIES B WITH(NOLOCK)  
        ON (B.major_id = object_id(A.TABLE_NAME) AND A.ORDINAL_POSITION = B.minor_id) 
    LEFT OUTER JOIN 
        INFORMATION_SCHEMA.KEY_COLUMN_USAGE D WITH(NOLOCK) 
        ON (D.TABLE_NAME = AA.TABLE_NAME AND D.COLUMN_NAME = A.COLUMN_NAME) 
    LEFT OUTER JOIN 
        (SELECT B.[name] AS [Table], A.[name] AS [Colum] 
        FROM  syscolumns A WITH(NOLOCK) JOIN  sysobjects B WITH(NOLOCK) ON (B.id = A.id) 
        WHERE A.[status] = 128 -- Identity 컬럼은 0x80 값을 갖는다. 
        AND   B.[name] = AA.TABLE_NAME) E  
        ON (A.TABLE_NAME = E.[Table] AND A.COLUMN_NAME = E.Colum) 
    WHERE  
        A.TABLE_NAME = AA.TABLE_NAME 
    ORDER BY  
        A.TABLE_NAME, A.ORDINAL_POSITION 
    FOR JSON AUTO) AS TableSchema 
FROM INFORMATION_SCHEMA.TABLES AA WITH(NOLOCK) 
WHERE  
    AA.TABLE_TYPE = 'BASE TABLE' 
    AND AA.TABLE_NAME NOT IN (   'sysdiagrams' 
                        , 'tmpattend' 
                        , 'TemplateNOPS' 
                        , 'Validation') 
ORDER BY AA.TABLE_NAME
```

#### 단일 테이블 상세 스키마

```sql
SELECT 
        A.TABLE_NAME AS TableName 
    , A.COLUMN_NAME AS ColumnName 
    , A.DATA_TYPE AS DataType 
    , ISNULL(ISNULL(CAST(A.CHARACTER_MAXIMUM_LENGTH AS VARCHAR(10)), CAST(A.NUMERIC_PRECISION AS VARCHAR(10))), '-2') AS ColumnLength 
    , ISNULL(A.COLUMN_DEFAULT, '') AS ColumnDefault 
    , A.IS_NULLABLE AS IsNullable 
    , ISNULL(B.VALUE, '') AS ColumnComment 
    , CASE WHEN D.COLUMN_NAME IS NULL THEN 'N' ELSE 'Y' END AS PrimaryKey 
    , CASE WHEN E.Colum IS NULL THEN 'N' ELSE 'Y' END AS [Identity] 
FROM 
    INFORMATION_SCHEMA.COLUMNS A WITH(NOLOCK) 
LEFT OUTER JOIN 
    SYS.EXTENDED_PROPERTIES B WITH(NOLOCK)  
    ON (B.major_id = object_id(A.TABLE_NAME) AND A.ORDINAL_POSITION = B.minor_id) 
LEFT OUTER JOIN 
    INFORMATION_SCHEMA.KEY_COLUMN_USAGE D WITH(NOLOCK) 
    ON (D.TABLE_NAME = @TableName AND D.COLUMN_NAME = A.COLUMN_NAME) 
LEFT OUTER JOIN 
    (SELECT B.[name] AS [Table], A.[name] AS [Colum] 
    FROM  syscolumns A WITH(NOLOCK) JOIN  sysobjects B WITH(NOLOCK) ON (B.id = A.id) 
    WHERE A.[status] = 128 -- Identity 컬럼은 0x80 값을 갖는다. 
    AND   B.[name] = @TableName) E  
    ON (A.TABLE_NAME = E.[Table] AND A.COLUMN_NAME = E.Colum) 
WHERE  
    A.TABLE_NAME = @TableName 
ORDER BY  
    A.TABLE_NAME, A.ORDINAL_POSITION
```

---

#### 임시 테이블과 대상 테이블을 통한 데이터 변경 확인

작업자가 여럿일 때 어떤 사람이 어떤 데이터를 등록/수정/삭제 했는지 알 필요가 있을 때가 있다. 이럴 땐 아래와 같이 별도 쿼리로 **임시테이블 + 대상테이블**을 조회하여 로그로 적재하여 관리툴 등에서 확인할 수 있도록 할 수 있다.

1. A는 임시 테이블*(ex - GameUser_temp)*, B는 대상 테이블 *(ex - GameUser)*이다.
2. 데이터 저장 프로세스 과정에서 업로드 한 엑셀 데이터는 A에 저장되고, A를 기준으로 기존 테이블 데이터 B와 검증이 이루어진다.
3. 각각 이전, 이후 데이터를 가진 A, B테이블을 키로 Join 하여 <span class="text-success">**신규**</span>, <span class="text-danger">**삭제**</span>를 찾을 수 있다.
4. 키로 조인한 모든 컬럼을 비교해서 변경점이 있는 항목에 대하여 이전 이후 데이터를 볼 수 있으니 <span class="text-warning">**변경점**</span>을 찾을 수 있다.

```sql
A LEFT JOIN B , B IS NULL -> A에서 신규로 생긴 것들 찾기
B LEFT JOIN A , A IS NULL -> A에서 삭제된 것들 찾기
A LEFT JOIN B + [Check All column]  -> 모든 컬럼 비교 후 변화 있는 것에 대하여 before, after 를 찾을 수 있다.
```

[umllink]: https://www.plantuml.com/plantuml/uml/bLHVQzDm57tFfxZaQHDMS1z09xL2SNHWjmSPUnWkmy6iicNuchfV8JIJjRXOLvBGGN4AUSXgAode5yhl_hw3DxyiVqeiw4i9pRddddFkQJTEzVqJ_UpDiRBoO1NMrXw1YeC-TWpW7mDqfu85Aw9pGKyyl5e577JuZSC_EQhoZpYGbCpX4rP_2ZouX0m6w8p4y1koEy39LBHzLL6E3a5DlXQs0-dSOnNqrrfB0MBRRMpjvPWlU6i0sbrqZGojAjhngHQ32Vsorjm3OatbeZUAGXFml2tE7H3NDhfXnYxvTngRMzNdoH2VC_myHS_atQlSAOtGsl8mWDm9ktnE6v9zueBS_YB5mfvoQrvmltrordgrhsjgCXjhXaMQ1Gh06nBY4rEcWypqAUcKe7LmT4YFz-OJ-z9AURcCAs5FE73SHw_3BPDRuuggf6p9fr9UAS_QrBLt3oltkUEW1mNOg8t3x-WoG8V9Kte-MWvDqrwV_pIoarFCYqHBeNa7v5NJMaM1KHnPD2B3o6kJuGX6IW5jngrUDFcDmm1Y5UiG1tHvJLNy3p2i7RnztqeZMQyKEzuTYmzZVckM32-1bSq-UrBVZ6zO1ilC_H17JXA2YTtPNwmtTwhLMbCcbgs99WP-TG7PNVHZHcwNligzeSSeZuBeUgDPUx4jVzHH650_yrTv4w9PZoofoWPrYVvd_W00