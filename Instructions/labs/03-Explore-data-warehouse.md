---
lab:
  title: 랩 – 관계형 데이터 웨어하우스 탐색
  module: 'Model, query, and explore data in Azure Synapse'
---

# <a name="explore-a-relational-data-warehouse"></a>랩 – 관계형 데이터 웨어하우스 탐색

Azure Synapse Analytics is built on a scalable set capabilities to support enterprise data warehousing; including file-based data analytics in a data lake as well as large-scale relational data warehouses and the data transfer and transformation pipelines used to load them. In this lab, you'll explore how to use a dedicated SQL pool in Azure Synapse Analytics to store and query data in a relational data warehouse.

이 랩을 완료하는 데 약 **45**분이 걸립니다.

## <a name="before-you-start"></a>시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free)이 필요합니다.

## <a name="provision-an-azure-synapse-analytics-workspace"></a>Azure Synapse Analytics 작업 영역 프로비저닝

An Azure Synapse Analytics <bpt id="p1">*</bpt>workspace<ept id="p1">*</ept> provides a central point for managing data and data processing runtimes. You can provision a workspace using the interactive interface in the Azure portal, or you can deploy a workspace and resources within it by using a script or template. In most production scenarios, it's best to automate provisioning with scripts and templates so that you can incorporate resource deployment into a repeatable development and operations (<bpt id="p1">*</bpt>DevOps<ept id="p1">*</ept>) process.

이 연습에서는 PowerShell 스크립트와 ARM 템플릿의 조합을 사용하여 Azure Synapse Analytics 작업 영역을 프로비저닝합니다.

1. `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)에 로그인합니다.
2. Use the <bpt id="p1">**</bpt>[<ph id="ph1">\&gt;</ph>_]<ept id="p1">**</ept> button to the right of the search bar at the top of the page to create a new Cloud Shell in the Azure portal, selecting a <bpt id="p2">***</bpt>PowerShell<ept id="p2">***</ept> environment and creating storage if prompted. The cloud shell provides a command line interface in a pane at the bottom of the Azure portal, as shown here:

    ![Cloud Shell 창이 있는 Azure Portal](../images/cloud-shell.png)

    > **참고**: 이전에 *Bash* 환경을 사용하는 클라우드 셸을 만들었다면 클라우드 셸 창의 왼쪽 위에 있는 드롭다운 메뉴를 사용하여 ***PowerShell***로 변경합니다.

3. Azure Synapse Analytics는 데이터 레이크의 파일 기반 데이터 분석뿐만 아니라 대규모 관계형 데이터 웨어하우스 및 이를 로드하는 데 사용되는 데이터 전송 및 변환 파이프라인을 포함하여 엔터프라이즈 데이터 웨어하우징을 지원하는 확장 가능한 집합 기능을 기반으로 합니다.

4. PowerShell 창에서 다음 명령을 입력하여 이 리포지토리를 복제합니다.

    ```
    rm -r dp500 -f
    git clone https://github.com/MicrosoftLearning/DP-500-Azure-Data-Analyst dp500
    ```

5. 리포지토리가 복제된 후에는 다음 명령을 입력하여 이 랩의 폴더로 변경하고 포함된 **setup.ps1** 스크립트를 실행합니다.

    ```
    cd dp500/Allfiles/03
    ./setup.ps1
    ```

6. 메시지가 표시되면 사용할 구독을 선택합니다(여러 Azure 구독에 액세스할 수 있는 경우에만 발생).
7. 메시지가 표시되면 Azure Synapse SQL 풀에 설정할 적절한 암호를 입력합니다.

    > **참고**: 이 암호를 기억하세요.

8. 이 랩에서는 Azure Synapse Analytics의 전용 SQL 풀을 사용하여 관계형 데이터 웨어하우스에 데이터를 저장하고 쿼리하는 방법을 살펴봅니다.

## <a name="explore-the-data-warehouse-schema"></a>데이터 웨어하우스 스키마 탐색

이 랩에서 데이터 웨어하우스는 Azure Synapse Analytics의 전용 SQL 풀에서 호스트됩니다.

### <a name="start-the-dedicated-sql-pool"></a>전용 SQL 풀 시작

1. 스크립트가 완료되면 Azure Portal에서 만든 **dp500-*xxxxxxx*** 리소스 그룹으로 이동하여 Synapse 작업 영역을 선택합니다.
2. Synapse 작업 영역에 있는 **개요** 페이지의 **Synapse Studio 열기** 카드에서 **열기**를 선택하여 새 브라우저 탭에서 Synapse Studio 엽니다. 메시지가 표시되면 로그인합니다.
3. Synapse Studio 왼쪽에 있는 **&rsaquo;&rsaquo;** 아이콘을 사용하여 메뉴를 확장합니다. 이렇게 하면 Synapse Studio에서 리소스를 관리하고 데이터 분석 작업을 수행하는 데 사용할 여러 페이지가 표시됩니다.
4. **관리** 페이지에서 **SQL 풀** 탭이 선택되어 있는지 확인하고 **sql*xxxxxxx*** 전용 SQL 풀을 선택하고 **&#9655;** 아이콘을 사용하여 시작합니다. 메시지가 표시되면 다시 시작하려는지 확인합니다.
5. Wait for the SQL pool to resume. This can take a few minutes. Use the <bpt id="p1">**</bpt>&amp;#8635; Refresh<ept id="p1">**</ept> button to check its status periodically. The status will show as <bpt id="p1">**</bpt>Online<ept id="p1">**</ept> when it is ready.

### <a name="view-the-tables-in-the-database"></a>데이터베이스의 테이블 보기

1. Synapse Studio **데이터** 페이지를 선택하고 **작업 영역** 탭이 선택되어 있고 **SQL 데이터베이스** 범주가 포함되어 있는지 확인합니다.
2. **SQL 데이터베이스**, **sql*xxxxxxx*** 풀 및 해당 **Tables** 폴더를 확장하여 데이터베이스의 테이블을 확인합니다.

    A relational data warehouse is typically based on a schema that consists of <bpt id="p1">*</bpt>fact<ept id="p1">*</ept> and <bpt id="p2">*</bpt>dimension<ept id="p2">*</ept> tables. The tables are optimized for analytical queries in which numeric metrics in the fact tables are aggregated by attributes of the entities represented by the dimension tables - for example, enabling you to aggregate Internet sales revenue by product, customer, date, and so on.
    
3. Expand the <bpt id="p1">**</bpt>dbo.FactInternetSales<ept id="p1">**</ept> table and its <bpt id="p2">**</bpt>Columns<ept id="p2">**</ept> folder to see the columns in this table. Note that many of the columns are <bpt id="p1">*</bpt>keys<ept id="p1">*</ept> that reference rows in the dimension tables. Others are numeric values (<bpt id="p1">*</bpt>measures<ept id="p1">*</ept>) for analysis.
    
    키는 팩트 테이블을 하나 이상의 차원 테이블과 연결하기 위해 사용되며, *별표 스키* 마인 경우가 많습니다. 팩트 테이블이 각 차원 테이블과 직접 관련된 경우(가운데에 팩트 테이블이 있는 다중 뾰족한 "별"을 형성).

4. View the columns for the <bpt id="p1">**</bpt>dbo.DimPromotion<ept id="p1">**</ept> table, and note that it has a unique <bpt id="p2">**</bpt>PromotionKey<ept id="p2">**</ept> that uniquely identifies each row in the table. It also has an <bpt id="p1">**</bpt>AlternateKey<ept id="p1">**</ept>.

    Usually, data in a data warehouse has been imported from one or more transactional sources. The <bpt id="p1">*</bpt>alternate<ept id="p1">*</ept> key reflects the business identifier for the instance of this entity in the source, but a unique numeric <bpt id="p2">*</bpt>surrogate<ept id="p2">*</ept> key is usually generated to uniquely identify each row in the data warehouse dimension table. One of the benefits of this approach is that it enables the data warehouse to contain multiple instances of the same entity at different points in time (for example, records for the same customer reflecting their address at the time an order was placed).

5. dbo의 열을 봅니다 **. DimProduct**에는 dbo를 참조하는 **ProductSubcategoryKey** 열이 포함되어 있습니다 **. DimProductSubcategory** 테이블- dbo를 참조하는 **ProductCategoryKey** 열이 차례로 포함됩니다  **. DimProductCategory** 테이블.

    Azure Synapse Analytics *작업 영역*은 데이터 및 데이터 처리 런타임을 관리하기 위한 중앙 지점을 제공합니다.

6. dbo의 열을 봅니다 **. DimDate** 테이블은 요일, 월, 월, 연도, 일 이름, 월 이름 등을 포함하여 날짜의 다양한 임시 특성을 반영하는 여러 열을 포함합니다.

    Azure Portal 대화형 인터페이스를 사용하여 작업 영역을 프로비전하거나 스크립트 또는 템플릿을 사용하여 작업 영역 및 리소스를 배포할 수 있습니다.

## <a name="query-the-data-warehouse-tables"></a>데이터 웨어하우스 테이블 쿼리

이제 데이터 웨어하우스 스키마의 더 중요한 측면 중 일부를 살펴보셨으므로 테이블을 쿼리하고 일부 데이터를 검색할 준비가 되었습니다.

### <a name="query-fact-and-dimension-tables"></a>팩트 테이블과 차원 테이블 비교

대부분의 프로덕션 시나리오에서는 리소스 배포를 *DevOps*(반복 가능한 개발 및 작업) 프로세스에 통합할 수 있도록 스크립트 및 템플릿을 사용하여 프로비저닝을 자동화하는 것이 가장 좋습니다.

1. **데이터** 페이지에서 **sql*xxxxxxx*** SQL 풀을 선택하고 **...** 메뉴에서 **새 SQL 스크립트** > **빈 스크립트**를 선택합니다.
2. When a new <bpt id="p1">**</bpt>SQL Script 1<ept id="p1">**</ept> tab opens, in its <bpt id="p2">**</bpt>Properties<ept id="p2">**</ept> pane, change the name of the script to <bpt id="p3">**</bpt>Analyze Internet Sales<ept id="p3">**</ept> and change the <bpt id="p4">**</bpt>Result settings per query<ept id="p4">**</ept> to return all rows. Then use the <bpt id="p1">**</bpt>Publish<ept id="p1">**</ept> button on the toolbar to save the script, and use the <bpt id="p2">**</bpt>Properties<ept id="p2">**</ept> button (which looks similar to <bpt id="p3">**</bpt>&amp;#128463;.<ept id="p3">**</ept>) on the right end of the toolbar to close the <bpt id="p4">**</bpt>Properties<ept id="p4">**</ept> pane so you can see the script pane.
3. 빈 새 코드 셀에 다음 코드를 추가합니다.

    ```sql
    SELECT  d.CalendarYear AS Year,
            SUM(i.SalesAmount) AS InternetSalesAmount
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    GROUP BY d.CalendarYear
    ORDER BY Year;
    ```

4. Use the <bpt id="p1">**</bpt>&amp;#9655; Run<ept id="p1">**</ept> button to run the script, and review the results, which should show the Internet sales totals for each year. This query joins the fact table for Internet sales to a time dimension table based on the order date, and aggregates the sales amount measure in the fact table by the calendar month attribute of the dimension table.

5. 다음과 같이 쿼리를 수정하여 시간 차원의 월 특성을 추가한 다음 수정된 쿼리를 실행합니다.

    ```sql
    SELECT  d.CalendarYear AS Year,
            d.MonthNumberOfYear AS Month,
            SUM(i.SalesAmount) AS InternetSalesAmount
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    GROUP BY d.CalendarYear, d.MonthNumberOfYear
    ORDER BY Year, Month;
    ```

    Note that the attributes in the time dimension enable you to aggregate the measures in the fact table at multiple hierarchical levels - in this case, year and month. This is a common pattern in data warehouses.

6. 다음과 같이 쿼리를 수정하여 월을 제거하고 집계에 두 번째 차원을 추가한 다음 실행하여 결과를 확인합니다(각 지역의 연간 인터넷 판매 합계 표시).

    ```sql
    SELECT  d.CalendarYear AS Year,
            g.EnglishCountryRegionName AS Region,
            SUM(i.SalesAmount) AS InternetSalesAmount
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
    JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
    GROUP BY d.CalendarYear, g.EnglishCountryRegionName
    ORDER BY Year, Region;
    ```

    페이지 위쪽의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 ***PowerShell*** 환경을 선택하고 메시지가 표시되면 스토리지를 만듭니다.

7. 쿼리를 수정하고 다시 실행하여 다른 눈송이 차원을 추가하고 제품 범주별 연간 지역 매출을 집계합니다.

    ```sql
    SELECT  d.CalendarYear AS Year,
            pc.EnglishProductCategoryName AS ProductCategory,
            g.EnglishCountryRegionName AS Region,
            SUM(i.SalesAmount) AS InternetSalesAmount
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
    JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
    JOIN DimProduct AS p ON i.ProductKey = p.ProductKey
    JOIN DimProductSubcategory AS ps ON p.ProductSubcategoryKey = ps.ProductSubcategoryKey
    JOIN DimProductCategory AS pc ON ps.ProductCategoryKey = pc.ProductCategoryKey
    GROUP BY d.CalendarYear, pc.EnglishProductCategoryName, g.EnglishCountryRegionName
    ORDER BY Year, ProductCategory, Region;
    ```

    이번에는 제품 범주의 눈송이 차원에 제품, 하위 범주 및 범주 간의 계층적 관계를 반영하기 위해 세 개의 조인이 필요합니다.

8. 스크립트를 게시하여 저장합니다.

### <a name="use-ranking-functions"></a>순위 및 행 집합 함수 사용

대량의 데이터를 분석할 때 또 다른 일반적인 요구 사항은 데이터를 파티션별로 그룹화하고 특정 메트릭에 따라 파티션에서 각 엔터티의 *순위를* 결정하는 것입니다.

1. 기존 쿼리에서 다음 SQL을 추가하여 국가/지역 이름에 따라 파티션을 통해 2022년 판매액 값을 검색합니다.

    ```sql
    SELECT  g.EnglishCountryRegionName AS Region,
            ROW_NUMBER() OVER(PARTITION BY g.EnglishCountryRegionName
                              ORDER BY i.SalesAmount ASC) AS RowNumber,
            i.SalesOrderNumber AS OrderNo,
            i.SalesOrderLineNumber AS LineItem,
            i.SalesAmount AS SalesAmount,
            SUM(i.SalesAmount) OVER(PARTITION BY g.EnglishCountryRegionName) AS RegionTotal,
            AVG(i.SalesAmount) OVER(PARTITION BY g.EnglishCountryRegionName) AS RegionAverage
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
    JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
    WHERE d.CalendarYear = 2022
    ORDER BY Region;
    ```

2. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다.

    | 지역 | RowNumber | OrderNo | LineItem | SalesAmount | RegionTotal | RegionAverage |
    |--|--|--|--|--|--|--|
    |오스트레일리아|1|SO73943|2|2.2900|2172278.7900|375.8918|
    |오스트레일리아|2|SO74100|4|2.2900|2172278.7900|375.8918|
    |...|...|...|...|...|...|...|
    |오스트레일리아|577.9|SO64284|1|2443.3500|2172278.7900|375.8918|
    |Canada|1|SO66332|2|2.2900|563177.1000|157.8411|
    |캐나다|2|SO68234|2|2.2900|563177.1000|157.8411|
    |...|...|...|...|...|...|...|
    |캐나다|3568|SO70911|1|2443.3500|563177.1000|157.8411|
    |프랑스|1|SO68226|3|2.2900|816259.4300|315.4016|
    |프랑스|2|SO63460|2|2.2900|816259.4300|315.4016|
    |...|...|...|...|...|...|...|
    |프랑스|2588|SO69100|1|2443.3500|816259.4300|315.4016|
    |독일|1|SO70829|3|2.2900|922368.2100|352.4525|
    |독일|2|SO71651|2|2.2900|922368.2100|352.4525|
    |...|...|...|...|...|...|...|
    |독일|2617|SO67908|1|2443.3500|922368.2100|352.4525|
    |영국|1|SO66124|3|2.2900|1051560.1000|341.7484|
    |영국|2|SO67823|3|2.2900|1051560.1000|341.7484|
    |...|...|...|...|...|...|...|
    |영국|3077|SO71568|1|2443.3500|1051560.1000|341.7484|
    |미국|1|SO74796|2|2.2900|2905011.1600|289.0270|
    |미국|2|SO65114|2|2.2900|2905011.1600|289.0270|
    |...|...|...|...|...|...|...|
    |미국|10051|SO66863|1|2443.3500|2905011.1600|289.0270|

    이러한 결과에 대해 다음과 같은 사실을 확인합니다.

    - 각 판매 주문 품목에 대한 행이 있습니다.
    - 행은 판매가 이루어진 지역에 따라 파티션으로 구성됩니다.
    - 각 지리적 파티션 내의 행은 판매 금액 순서로 번호가 매겨집니다(최하위에서 가장 높음).
    - 각 행에 대해 품목 판매액과 지역 총 및 평균 판매액이 포함됩니다.

3. 기존 쿼리에서 다음 코드를 추가하여 GROUP BY 쿼리 내에서 창 함수를 적용하고 총 판매액에 따라 각 지역의 도시 순위를 지정합니다.

    ```sql
    SELECT  g.EnglishCountryRegionName AS Region,
            g.City,
            SUM(i.SalesAmount) AS CityTotal,
            SUM(SUM(i.SalesAmount)) OVER(PARTITION BY g.EnglishCountryRegionName) AS RegionTotal,
            RANK() OVER(PARTITION BY g.EnglishCountryRegionName
                        ORDER BY SUM(i.SalesAmount) DESC) AS RegionalRank
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    JOIN DimCustomer AS c ON i.CustomerKey = c.CustomerKey
    JOIN DimGeography AS g ON c.GeographyKey = g.GeographyKey
    GROUP BY g.EnglishCountryRegionName, g.City
    ORDER BY Region;
    ```

4. Select only the new query code, and use the <bpt id="p1">**</bpt>&amp;#9655; Run<ept id="p1">**</ept> button to run it. Then review the results, and observe the following:
    - 결과에는 지역별로 그룹화된 각 도시에 대한 행이 포함됩니다.
    - 각 도시에 대한 총 판매액(개별 판매액 합계)이 계산됩니다.
    - 지역별 총 판매액(지역의 각 도시에 대한 판매 금액 합계 합계)은 지역 파티션을 기준으로 계산됩니다.
    - 지역 파티션 내의 각 도시에 대한 순위는 도시당 총 판매액을 내림차순으로 정렬하여 계산됩니다.

5. 업데이트된 스크립트를 게시하여 변경 내용을 저장합니다.

> <bpt id="p1">**</bpt>Tip<ept id="p1">**</ept>: ROW_NUMBER and RANK are examples of ranking functions available in Transact-SQL. For more details, see the <bpt id="p1">[</bpt>Ranking Functions<ept id="p1">](https://docs.microsoft.com/sql/t-sql/functions/ranking-functions-transact-sql)</ept> reference in the Transact-SQL language documentation.

### <a name="retrieve-an-approximate-count"></a>대략적인 개수 검색

When exploring very large volumes of data, queries can take significant time and resources to run. Often, data analysis doesn't require absolutely precise values - a comparison of approximate values may be sufficient.

1. 기존 쿼리에서 다음 코드를 추가하여 각 연도의 판매 주문 수를 검색합니다.

    ```sql
    SELECT d.CalendarYear AS CalendarYear,
        COUNT(DISTINCT i.SalesOrderNumber) AS Orders
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    GROUP BY d.CalendarYear
    ORDER BY CalendarYear;
    ```

2. 창 맨 위에 있는 구분 기호 막대를 끌거나 창 오른쪽 위에 있는 **&#8212;** , **&#9723;** 및 **X** 아이콘을 사용하여 Cloud Shell 크기를 조정하여 창을 최소화, 최대화하고 닫을 수 있습니다.
    - 쿼리 아래의 **결과** 탭에서 각 연도의 주문 수를 확인합니다.
    - **메시지** 탭에서 쿼리의 총 실행 시간을 봅니다.
3. Azure Cloud Shell 사용에 관한 자세한 내용은 [Azure Cloud Shell 설명서](https://docs.microsoft.com/azure/cloud-shell/overview)를 참조하세요.

    ```sql
    SELECT d.CalendarYear AS CalendarYear,
        APPROX_COUNT_DISTINCT(i.SalesOrderNumber) AS Orders
    FROM FactInternetSales AS i
    JOIN DimDate AS d ON i.OrderDateKey = d.DateKey
    GROUP BY d.CalendarYear
    ORDER BY CalendarYear;
    ```

4. 반환되는 출력을 검토합니다.
    - On the <bpt id="p1">**</bpt>Results<ept id="p1">**</ept> tab under the query, view the order counts for each year. These should be within 2% of the actual counts retrieved by the previous query.
    - On the <bpt id="p1">**</bpt>Messages<ept id="p1">**</ept> tab, view the total execution time for the query. This should be shorter than for the previous query.

5. 스크립트를 게시하여 변경 내용을 저장합니다.

> 자세한 내용은 [APPROX_COUNT_DISTINCT](https://docs.microsoft.com/sql/t-sql/functions/approx-count-distinct-transact-sql) 함수 설명서를 참조하세요.

## <a name="challenge---analyze-reseller-sales"></a>과제 - 재판매인 판매 분석

1. **sql*xxxxxxx*** SQL 풀에 대한 빈 스크립트를 새로 만들고 **Reseller Sales 분석**이라는 이름으로 저장합니다.
2. 스크립트에서 SQL 쿼리를 만들어 **FactResellerSales** 팩트 테이블 및 관련 차원 테이블을 기반으로 다음 정보를 찾습니다.
    - 회계 연도 및 분기당 판매된 항목의 총 수량입니다.
    - 판매한 직원과 관련된 회계 연도, 분기 및 판매 지역별 판매 항목의 총 수량입니다.
    - 제품 범주별 회계 연도, 분기 및 판매 지역별 판매 항목의 총 수량입니다.
    - 연도의 총 판매액을 기준으로 회계 연도당 각 판매 지역 순위입니다.
    - 각 판매 지역의 연간 대략 판매 주문 수입니다.

    > **팁**: 쿼리를 Synapse Studio **개발** 페이지의 **솔루션** 스크립트와 비교합니다.

3. 쿼리를 실험하여 데이터 웨어하우스 스키마의 나머지 테이블을 여가로 탐색합니다.
4. 완료되면 **관리** 페이지에서 **sql*xxxxxxx*** 전용 SQL 풀을 일시 중지합니다.

## <a name="delete-azure-resources"></a>Azure 리소스 삭제

Azure Synapse Analytics 탐색을 완료했으므로, 지금까지 만든 리소스를 삭제하여 불필요한 Azure 비용을 방지해야 합니다.

1. Synapse Studio 브라우저 탭을 닫고 Azure Portal로 돌아갑니다.
2. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
3. (관리되는 리소스 그룹이 아닌) Synapse Analytics 작업 영역에 대한 **dp500-*xxxxxxx*** 리소스 그룹을 선택하고 Synapse 작업 영역, 스토리지 계정 및 작업 영역용 Spark 풀이 포함되어 있는지 확인합니다.
4. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
5. **dp500-*xxxxxxx*** 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음, **삭제**를 선택합니다.

    몇 분이 지나면 Azure Synapse 작업 영역 리소스 그룹과 여기에 연결된 관리 작업 영역 리소스 그룹이 삭제됩니다.
