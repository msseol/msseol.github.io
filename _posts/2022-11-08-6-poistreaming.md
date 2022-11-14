---
layout: post
title:  "Apache Poi Excel 스트리밍 업로드"
date:   2022-11-08 23:00:08
categories: BACKEND
tags: excel poi streaming
description: 대용량 엑셀 스트리밍 업로드 하기(with Poi)
---

<i class="fa-solid fa-check"></i> *대용량 엑셀 업로드 시 분할 업로드 처리하기 메모*

---

내부 툴을 개발하면서 대용량(30만 로우이상) 데이터의 엑셀을 처리해야 할 일이 있었는데, 통으로 파일을 받을 경우 메모리가 부족한 현상이 있었다.
(서비스가 대부분 Xmx Xms 64mb 수준으로 돌아가서....) 관련해서 http 스트리밍 업로드 하는 라이브러리가 있어 적용하였다.

**Maven dependency**

poi와 xlsx-streamer 라는 라이브러리를 사용한다.
```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>4.1.0</version>
</dependency>

<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>4.1.0</version>
</dependency>

<dependency>
    <groupId>com.monitorjbl</groupId>
    <artifactId>xlsx-streamer</artifactId>
    <version>2.0.0</version>
</dependency>
```

**ExcelReader.java**

아래는 MultipartFile로 엑셀을 받아 특정 행의 값을 10000개 행씩 처리하는 예시다.   
StreamingReader으로 Workbook을 읽어오는 부분만 자세히 보면 된다.

```java
import java.util.Arrays;
import java.util.Iterator;
import java.util.function.Function;

import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartFile;

import com.monitorjbl.xlsx.StreamingReader;

@Component
public class ExcelReader {
    protected Logger logger = LoggerFactory.getLogger(this.getClass());

    @SuppressWarnings("unchecked")
    public <T> void readExcel(MultipartFile file, Function<List<T>, Integer> func, Class<T> type) throws Exception {

        List<T> datas = new LinkedList<>();
        try (Workbook workbook = StreamingReader.builder().rowCacheSize(100).bufferSize(4096)
                .open(file.getInputStream())) {
            Sheet sheet = workbook.getSheetAt(0);

            boolean firstRow = true;
            int targetCellIndex = -1;

            int loopNo = 1;
            int bulkSize = 10000; // 몇 개 행씩 처리할 지


            Iterator<Row> rows = sheet.iterator();
            while (rows.hasNext()) {
                Row row = rows.next();

                if (firstRow) { // 첫 행에 대한 처리, 헤더값을 통해 특정 객체 지정하거나 특정 컬럼 읽기 시도 등등 가능
                    Iterator<Cell> cells = row.cellIterator();
                    while (firstRow && cells.hasNext()) {
                        Cell cell = cells.next();
                        String cellName = cell.getStringCellValue();
                        if ("특정셀헤더".equals(cellName)) { // ex) Id
                            targetCellIndex = cell.getColumnIndex();
                        }
                    }

                    firstRow = false;
                } else {
                    boolean endOfKey = false; // 엑셀 밑라인 공백 방지

                    Cell cell = row.getCell(targetCellIndex);
                    String cellValue = (cell == null) ? "" : cell.getStringCellValue();

                    if ("".equals(cellValue)) endOfKey = true;

                    if (!endOfKey) {

                        --- Model로 변환 // ex) T => Model...  new Model(cellValue)

                        datas.add((T) m);
                    }

                    if (!rows.hasNext() || endOfKey || loopNo == bulkSize) {
                        loopNo = 0;

                        if (datas.isEmpty()) break;

                        int ret = func.apply(datas);

                        datas.clear();

                        --- ret 에 따른 오류나 응답 처리

                        if (endOfKey) break;
                    }

                    loopNo++;
                }

            }
        }
    }
}
```

**사용하기**

함수는 편한대로 수정해서 사용하면 된다.

```java

// DI
private final ExcelReader excelReader;
private final Repository repository;

...

excelReader.readExcel(multipartFile, Repository::saveAll, Model.class); // Function 부분엔 row datas에 대한 처리 및 응답(status) 리턴해서 사용

```