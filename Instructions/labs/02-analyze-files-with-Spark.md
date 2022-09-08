---
lab:
  title: Spark를 사용하여 데이터 레이크에서 데이터 분석
  module: 'Model, query, and explore data in Azure Synapse'
---

# <a name="analyze-data-in-a-data-lake-with-spark"></a>Spark를 사용하여 데이터 레이크에서 데이터 분석

Apache Spark is an open source engine for distributed data processing, and is widely used to explore, process, and analyze huge volumes of data in data lake storage. Spark is available as a processing option in many data platform products, including Azure HDInsight, Azure Databricks, and Azure Synapse Analytics on the Microsoft Azure cloud platform. One of the benefits of Spark is support for a wide range of programming languages, including Java, Scala, Python, and SQL; making Spark a very flexible solution for data processing workloads including data cleansing and manipulation, statistical analysis and machine learning, and data analytics and visualization.

이 랩을 완료하는 데 약 **45**분이 걸립니다.

## <a name="before-you-start"></a>시작하기 전에

관리 수준 액세스 권한이 있는 [Azure 구독](https://azure.microsoft.com/free)이 필요합니다.

## <a name="provision-an-azure-synapse-analytics-workspace"></a>Azure Synapse Analytics 작업 영역 프로비저닝

데이터 레이크 스토리지에 액세스할 수 있는 Azure Synapse Analytics 작업 영역과 데이터 레이크에서 파일을 쿼리하고 처리하는 데 사용할 수 있는 Apache Spark 풀이 필요합니다.

이 연습에서는 PowerShell 스크립트와 ARM 템플릿의 조합을 사용하여 Azure Synapse Analytics 작업 영역을 프로비저닝합니다.

1. `https://portal.azure.com`에서 [Azure Portal](https://portal.azure.com)에 로그인합니다.
2. Use the <bpt id="p1">**</bpt>[<ph id="ph1">\&gt;</ph>_]<ept id="p1">**</ept> button to the right of the search bar at the top of the page to create a new Cloud Shell in the Azure portal, selecting a <bpt id="p2">***</bpt>PowerShell<ept id="p2">***</ept> environment and creating storage if prompted. The cloud shell provides a command line interface in a pane at the bottom of the Azure portal, as shown here:

    ![Cloud Shell 창이 있는 Azure Portal](../images/cloud-shell.png)

    > **참고**: 이전에 *Bash* 환경을 사용하는 클라우드 셸을 만들었다면 클라우드 셸 창의 왼쪽 위에 있는 드롭다운 메뉴를 사용하여 ***PowerShell***로 변경합니다.

3. Note that you can resize the cloud shell by dragging the separator bar at the top of the pane, or by using the <bpt id="p1">**</bpt>&amp;#8212;<ept id="p1">**</ept>, <bpt id="p2">**</bpt>&amp;#9723;<ept id="p2">**</ept>, and <bpt id="p3">**</bpt>X<ept id="p3">**</ept> icons at the top right of the pane to minimize, maximize, and close the pane. For more information about using the Azure Cloud Shell, see the <bpt id="p1">[</bpt>Azure Cloud Shell documentation<ept id="p1">](https://docs.microsoft.com/azure/cloud-shell/overview)</ept>.

4. PowerShell 창에서 다음 명령을 입력하여 이 리포지토리를 복제합니다.

    ```
    rm -r dp500 -f
    git clone https://github.com/MicrosoftLearning/DP-500-Azure-Data-Analyst dp500
    ```

5. 리포지토리가 복제된 후에는 다음 명령을 입력하여 이 랩의 폴더로 변경하고 포함된 **setup.ps1** 스크립트를 실행합니다.

    ```
    cd dp500/Allfiles/02
    ./setup.ps1
    ```

6. 메시지가 표시되면 사용할 구독을 선택합니다(여러 Azure 구독에 액세스할 수 있는 경우에만 발생).
7. 메시지가 표시되면 Azure Synapse SQL 풀에 설정할 적절한 암호를 입력합니다.

    > **참고**: 이 암호를 기억하세요.

8. Apache Spark는 분산 데이터 처리를 위한 오픈 소스 엔진으로, Data Lake Storage에서 대량의 데이터를 탐색, 처리, 분석하는 데 널리 사용됩니다.

## <a name="query-data-in-files"></a>파일의 데이터 쿼리

스크립트는 Azure Synapse Analytics 작업 영역 및 Azure Storage 계정을 프로비저닝하여 데이터 레이크를 호스트한 다음, 일부 데이터 파일을 데이터 레이크에 업로드합니다.

### <a name="view-files-in-the-data-lake"></a>데이터 레이크에서 파일 보기

1. 스크립트가 완료되면 Azure Portal에서 만든 **dp500-*xxxxxxx*** 리소스 그룹으로 이동하여 Synapse 작업 영역을 선택합니다.
2. Synapse 작업 영역에 있는 **개요** 페이지의 **Synapse Studio 열기** 카드에서 **열기**를 선택하여 새 브라우저 탭에서 Synapse Studio 엽니다. 메시지가 표시되면 로그인합니다.
3. Synapse Studio 왼쪽에 있는 **&rsaquo;&rsaquo;** 아이콘을 사용하여 메뉴를 확장합니다. 이렇게 하면 Synapse Studio에서 리소스를 관리하고 데이터 분석 작업을 수행하는 데 사용할 여러 페이지가 표시됩니다.
4. Spark는 Microsoft Azure 클라우드 플랫폼의 Azure HDInsight, Azure Databricks, Azure Synapse Analytics를 비롯한 많은 데이터 플랫폼 제품에서 처리 옵션으로 사용할 수 있습니다.
5. **데이터** 페이지에서 **연결된** 탭을 보고 작업 영역에 **synapse*xxxxxxx*(Primary - datalake*xxxxxxx*)**와 유사한 이름을 가진 Azure Data Lake Storage Gen2 스토리지 계정에 대한 링크가 포함되어 있는지 확인합니다.
6. 스토리지 계정을 확장하고 **파일**이라는 파일 시스템 컨테이너가 포함되어 있는지 확인합니다.
7. Spark의 이점 중 하나는 Java, Scala, Python, SQL을 포함한 다양한 프로그래밍 언어를 지원한다는 점입니다. Spark는 데이터 정리 및 조작, 통계 분석 및 기계 학습, 데이터 분석 및 시각화를 포함한 데이터 처리 워크로드를 위한 매우 유연한 솔루션입니다.
8. **sales** 폴더와 해당 폴더에 포함된 **orders** 폴더를 열고 **orders** 폴더에 3년간의 판매 데이터에 대한 .csv 파일이 포함되어 있는지 확인합니다.
9. Right-click any of the files and select <bpt id="p1">**</bpt>Preview<ept id="p1">**</ept> to see the data it contains. Note that the files do not contain a header row, so you can unselect the option to display column headers.

### <a name="use-spark-to-explore-data"></a>Spark를 사용하여 데이터 탐색

1. Select any of the files in the <bpt id="p1">**</bpt>orders<ept id="p1">**</ept> folder, and then in the <bpt id="p2">**</bpt>New notebook<ept id="p2">**</ept> list on the toolbar, select <bpt id="p3">**</bpt>Load to DataFrame<ept id="p3">**</ept>. A dataframe is a structure in Spark that represents a tabular dataset.
2. In the new <bpt id="p1">**</bpt>Notebook 1<ept id="p1">**</ept> tab that opens, in the <bpt id="p2">**</bpt>Attach to<ept id="p2">**</ept> list, select your Spark pool (*<bpt id="p3">*</bpt>spark<ept id="p3">*</ept>xxxxxxx***). Then use the <bpt id="p1">**</bpt>&amp;#9655; Run all<ept id="p1">**</ept> button to run all of the cells in the notebook (there's currently only one!).

    Since this is the first time you've run any Spark code in this session, the Spark pool must be started. This means that the first run in the session can take a few minutes. Subsequent runs will be quicker.

3. Spark 세션이 초기화되기를 기다리는 동안 생성된 코드를 검토합니다. 이 코드는 다음과 유사하게 표시됩니다.

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/sales/orders/2019.csv', format='csv'
    ## If header exists uncomment line below
    ##, header=True
    )
    display(df.limit(10))
    ```

4. When the code has finished running, review the output beneath the cell in the notebook. It shows the first ten rows in the file you selected, with automatic column names in the form <bpt id="p1">**</bpt>_c0<ept id="p1">**</ept>, <bpt id="p2">**</bpt>_c1<ept id="p2">**</ept>, <bpt id="p3">**</bpt>_c2<ept id="p3">**</ept>, and so on.
5. Modify the code so that the <bpt id="p1">**</bpt>spark.read.load<ept id="p1">**</ept> function reads data from <bpt id="p2">&lt;u&gt;</bpt>all<ept id="p2">&lt;/u&gt;</ept> of the CSV files in the folder, and the <bpt id="p3">**</bpt>display<ept id="p3">**</ept> function shows the first 100 rows. Your code should look like this (with <bpt id="p1">*</bpt>datalakexxxxxxx<ept id="p1">*</ept> matching the name of your data lake store):

    ```Python
    %%pyspark
    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/sales/orders/*.csv', format='csv'
    )
    display(df.limit(100))
    ```

6. 코드 셀의 왼쪽에 있는 **&#9655;** 단추를 사용하여 해당 셀만 실행하고 결과를 검토합니다.

    The dataframe now includes data from all of the files, but the column names are not useful. Spark uses a "schema-on-read" approach to try to determine appropriate data types for the columns based on the data they contain, and if a header row is present in a text file it can be used to identify the column names (by specifying a <bpt id="p1">**</bpt>header=True<ept id="p1">**</ept> parameter in the <bpt id="p2">**</bpt>load<ept id="p2">**</ept> function). Alternatively, you can define an explicit schema for the dataframe.

7. Modify the code as follows (replacing <bpt id="p1">*</bpt>datalakexxxxxxx<ept id="p1">*</ept>), to define an explicit schema for the dataframe that includes the column names and data types. Rerun the code in the cell.

    ```Python
    %%pyspark
    from pyspark.sql.types import *
    from pyspark.sql.functions import *

    orderSchema = StructType([
        StructField("SalesOrderNumber", StringType()),
        StructField("SalesOrderLineNumber", IntegerType()),
        StructField("OrderDate", DateType()),
        StructField("CustomerName", StringType()),
        StructField("Email", StringType()),
        StructField("Item", StringType()),
        StructField("Quantity", IntegerType()),
        StructField("UnitPrice", FloatType()),
        StructField("Tax", FloatType())
        ])

    df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/sales/orders/*.csv', format='csv', schema=orderSchema)
    display(df.limit(100))
    ```

8. 페이지 위쪽의 검색 창 오른쪽에 있는 **[\>_]** 단추를 사용하여 Azure Portal에서 새 Cloud Shell을 만들고 ***PowerShell*** 환경을 선택하고 메시지가 표시되면 스토리지를 만듭니다.

    ```Python
    df.printSchema()
    ```

9. Cloud Shell은 다음과 같이 Azure Portal 아래쪽 창에 명령줄 인터페이스를 제공합니다.

## <a name="analyze-data-in-a-dataframe"></a>데이터 프레임에서 데이터 분석

Spark의 **dataframe** 개체는 Python의 Pandas 데이터 프레임과 유사하며 포함된 데이터를 조작, 필터링, 그룹화, 분석하는 데 사용할 수 있는 다양한 함수를 포함합니다.

### <a name="filter-a-dataframe"></a>데이터 프레임 필터링

1. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```Python
    customers = df['CustomerName', 'Email']
    print(customers.count())
    print(customers.distinct().count())
    display(customers.distinct())
    ```

2. Run the new code cell, and review the results. Observe the following details:
    - 데이터 프레임에서 작업을 수행하면 그 결과는 새 데이터 프레임이 됩니다(이 경우 **df** 데이터 프레임에서 열의 특정 하위 집합을 선택하여 새 **customers** 데이터 프레임이 생성됨).
    - 데이터 프레임은 포함된 데이터를 요약하고 필터링하는 데 사용할 수 있는 **count** 및 **distinct** 함수를 제공합니다.
    - The <ph id="ph1">`dataframe['Field1', 'Field2', ...]`</ph> syntax is a shorthand way of defining a subset of column. You can also use <bpt id="p1">**</bpt>select<ept id="p1">**</ept> method, so the first line of the code above could be written as <ph id="ph1">`customers = df.select("CustomerName", "Email")`</ph>

3. 다음과 같이 코드를 수정합니다.

    ```Python
    customers = df.select("CustomerName", "Email").where(df['Item']=='Road-250 Red, 52')
    print(customers.count())
    print(customers.distinct().count())
    display(customers.distinct())
    ```

4. Run the modified code to view the customers who have purchased the <bpt id="p1">*</bpt>Road-250 Red, 52<ept id="p1">*</ept> product. Note that you can "chain" multiple functions together so that the output of one function becomes the input for the next - in this case, the dataframe created by the <bpt id="p1">**</bpt>select<ept id="p1">**</ept> method is the source dataframe for the <bpt id="p2">**</bpt>where<ept id="p2">**</ept> method that is used to apply filtering criteria.

### <a name="aggregate-and-group-data-in-a-dataframe"></a>데이터 프레임에서 데이터 집계 및 그룹화

1. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```Python
    productSales = df.select("Item", "Quantity").groupBy("Item").sum()
    display(productSales)
    ```

2. 창 맨 위에 있는 구분 기호 막대를 끌거나 창 오른쪽 위에 있는 **&#8212;** , **&#9723;** 및 **X** 아이콘을 사용하여 Cloud Shell 크기를 조정하여 창을 최소화, 최대화하고 닫을 수 있습니다.

3. Notebook에 다른 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```Python
    yearlySales = df.select(year("OrderDate").alias("Year")).groupBy("Year").count().orderBy("Year")
    display(yearlySales)
    ```

4. Azure Cloud Shell 사용에 관한 자세한 내용은 [Azure Cloud Shell 설명서](https://docs.microsoft.com/azure/cloud-shell/overview)를 참조하세요.

## <a name="query-data-using-spark-sql"></a>Spark SQL을 사용하여 데이터 쿼리

As you've seen, the native methods of the dataframe object enable you to query and analyze data quite effectively. However, many data analysts are more comfortable working with SQL syntax. Spark SQL is a SQL language API in Spark that you can use to run SQL statements, or even persist data in relational tables.

### <a name="use-spark-sql-in-pyspark-code"></a>PySpark 코드에서 Spark SQL 사용

The default language in Azure Synapse Studio notebooks is PySpark, which is a Spark-based Python runtime. Within this runtime, you can use the <bpt id="p1">**</bpt>spark.sql<ept id="p1">**</ept> library to embed Spark SQL syntax within your Python code, and work with SQL constructs such as tables and views.

1. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```Python
    df.createOrReplaceTempView("salesorders")

    spark_df = spark.sql("SELECT * FROM salesorders")
    display(spark_df)
    ```

2. Run the cell and review the results. Observe that:
    - The code persists the data in the <bpt id="p1">**</bpt>df<ept id="p1">**</ept> dataframe as a temporary view named <bpt id="p2">**</bpt>salesorders<ept id="p2">**</ept>. Spark SQL supports the use of temporary views or persisted tables as sources for SQL queries.
    - 그런 다음 **spark.sql** 메서드를 사용하여 **salesorders** 뷰에 대해 SQL 쿼리를 실행합니다.
    - 쿼리 결과는 데이터 프레임에 저장됩니다.

### <a name="run-sql-code-in-a-cell"></a>셀에서 SQL 코드를 실행합니다.

PySpark 코드가 포함된 셀에 SQL 문을 포함시키는 것이 유용하겠지만 데이터 분석가는 SQL에서 직접 작업하려는 경우가 많습니다.

1. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```sql
    %%sql
    SELECT YEAR(OrderDate) AS OrderYear,
           SUM((UnitPrice * Quantity) + Tax) AS GrossRevenue
    FROM salesorders
    GROUP BY YEAR(OrderDate)
    ORDER BY OrderYear;
    ```

2. Run the cell and review the results. Observe that:
    - 셀의 시작 부분에 있는 `%%sql` 줄(*magic*이라고 함)은 Spark SQL 언어 런타임을 사용하여 PySpark 대신 이 셀에서 코드를 실행해야 함을 나타냅니다.
    - SQL 코드는 이전에 PySpark를 사용하여 만든 **salesorder** 뷰를 참조합니다.
    - SQL 쿼리의 출력은 셀 아래에 결과로 자동으로 표시됩니다.

> **참고**: Spark SQL 및 데이터 프레임에 대한 자세한 내용은 [Spark SQL 설명서](https://spark.apache.org/docs/2.2.0/sql-programming-guide.html)를 참조하세요.

## <a name="visualize-data-with-spark"></a>Spark를 사용하여 데이터 시각화

A picture is proverbially worth a thousand words, and a chart is often better than a thousand rows of data. While notebooks in Azure Synapse Analytics include a built in chart view for data that is displayed from a dataframe or Spark SQL query, it is not designed for comprehensive charting. However, you can use Python graphics libraries like <bpt id="p1">**</bpt>matplotlib<ept id="p1">**</ept> and <bpt id="p2">**</bpt>seaborn<ept id="p2">**</ept> to create charts from data in dataframes.

### <a name="view-results-as-a-chart"></a>결과를 차트로 보기

1. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```sql
    %%sql
    SELECT * FROM salesorders
    ```

2. 코드를 실행하고 이전에 만든 **salesorders** 뷰에서 데이터를 반환하는지 살펴봅니다.
3. 셀 아래의 결과 섹션에서 **보기** 옵션을 **테이블**에서 **차트**로 변경합니다.
4. 스크립트가 완료될 때까지 기다리세요. 일반적으로 약 10분이 걸리지만 경우에 따라 더 오래 걸릴 수 있습니다.
    - **차트 종류**: 가로 막대형 차트
    - **키**: Item
    - **값**: Quantity
    - **계열 그룹**: *비워 둠*
    - **집계**: Sum
    - **누적**: *선택되지 않음*

5. 차트가 다음과 유사한지 확인합니다.

    ![총 주문 수량별 제품의 가로 막대형 차트](../images/notebook-chart.png)

### <a name="get-started-with-matplotlib"></a>**matplotlib** 시작

1. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```Python
    sqlQuery = "SELECT CAST(YEAR(OrderDate) AS CHAR(4)) AS OrderYear, \
                    SUM((UnitPrice * Quantity) + Tax) AS GrossRevenue \
                FROM salesorders \
                GROUP BY CAST(YEAR(OrderDate) AS CHAR(4)) \
                ORDER BY OrderYear"
    df_spark = spark.sql(sqlQuery)
    df_spark.show()
    ```

2. 코드를 실행하고 연간 수익을 포함하는 Spark 데이터 프레임을 반환하는지 살펴봅니다.

    기다리는 동안 Azure Synapse Analytics 설명서에서 [Azure Synapse Analytics의 Apache Spark](https://docs.microsoft.com/azure/synapse-analytics/spark/apache-spark-overview) 문서를 검토합니다.

3. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 추가합니다.

    ```Python
    from matplotlib import pyplot as plt

    # matplotlib requires a Pandas dataframe, not a Spark one
    df_sales = df_spark.toPandas()

    # Create a bar plot of revenue by year
    plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'])

    # Display the plot
    plt.show()
    ```

4. Run the cell and review the results, which consist of a column chart with the total gross revenue for each year. Note the following features of the code used to produce this chart:
    - **matplotlib** 라이브러리에는 *Pandas* 데이터 프레임이 필요하므로 Spark SQL 쿼리에서 반환되는 *Spark* 데이터 프레임을 이 형식으로 변환해야 합니다.
    - At the core of the <bpt id="p1">**</bpt>matplotlib<ept id="p1">**</ept> library is the <bpt id="p2">**</bpt>pyplot<ept id="p2">**</ept> object. This is the foundation for most plotting functionality.
    - 기본 설정을 사용하면 사용자 지정할 수 있는 상당한 범위가 있는 사용 가능한 차트가 생성됩니다.

5. 다음과 같이 차트를 그리도록 코드를 수정합니다.

    ```Python
    # Clear the plot area
    plt.clf()

    # Create a bar plot of revenue by year
    plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')

    # Customize the chart
    plt.title('Revenue by Year')
    plt.xlabel('Year')
    plt.ylabel('Revenue')
    plt.grid(color='#95a5a6', linestyle='--', linewidth=2, axis='y', alpha=0.7)
    plt.xticks(rotation=45)

    # Show the figure
    plt.show()
    ```

6. Re-run the code cell and view the results. The chart now includes a little more information.

    A plot is technically contained with a <bpt id="p1">**</bpt>Figure<ept id="p1">**</ept>. In the previous examples, the figure was created implicitly for you; but you can create it explicitly.

7. 다음과 같이 차트를 그리도록 코드를 수정합니다.

    ```Python
    # Clear the plot area
    plt.clf()

    # Create a Figure
    fig = plt.figure(figsize=(8,3))

    # Create a bar plot of revenue by year
    plt.bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')

    # Customize the chart
    plt.title('Revenue by Year')
    plt.xlabel('Year')
    plt.ylabel('Revenue')
    plt.grid(color='#95a5a6', linestyle='--', linewidth=2, axis='y', alpha=0.7)
    plt.xticks(rotation=45)

    # Show the figure
    plt.show()
    ```

8. Re-run the code cell and view the results. The figure determines the shape and size of the plot.

    그림에는 각각 고유의 축에 여러 개의 하위 플롯이 포함될 수 있습니다.

9. 다음과 같이 차트를 그리도록 코드를 수정합니다.

    ```Python
    # Clear the plot area
    plt.clf()

    # Create a figure for 2 subplots (1 row, 2 columns)
    fig, ax = plt.subplots(1, 2, figsize = (10,4))

    # Create a bar plot of revenue by year on the first axis
    ax[0].bar(x=df_sales['OrderYear'], height=df_sales['GrossRevenue'], color='orange')
    ax[0].set_title('Revenue by Year')

    # Create a pie chart of yearly order counts on the second axis
    yearly_counts = df_sales['OrderYear'].value_counts()
    ax[1].pie(yearly_counts)
    ax[1].set_title('Orders per Year')
    ax[1].legend(yearly_counts.keys().tolist())

    # Add a title to the Figure
    fig.suptitle('Sales Data')

    # Show the figure
    plt.show()
    ```

10. Re-run the code cell and view the results. The figure contains the subplots specified in the code.

> **참고**: matplotlib를 사용하여 그리는 방법에 대한 자세한 내용은 [matplotlib 설명서](https://matplotlib.org/)를 참조하세요.

### <a name="use-the-seaborn-library"></a>**seaborn** 라이브러리 사용

While <bpt id="p1">**</bpt>matplotlib<ept id="p1">**</ept> enables you to create complex charts of multiple types, it can require some complex code to achieve the best results. For this reason, over the years, many new libraries have been built on the base of matplotlib to abstract its complexity and enhance its capabilities. One such library is <bpt id="p1">**</bpt>seaborn<ept id="p1">**</ept>.

1. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```Python
    import seaborn as sns

    # Clear the plot area
    plt.clf()

    # Create a bar chart
    ax = sns.barplot(x="OrderYear", y="GrossRevenue", data=df_sales)
    plt.show()
    ```

2. 코드를 실행하고 seaborn 라이브러리를 사용하여 가로 막대형 차트를 표시하는지 살펴봅니다.
3. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```Python
    # Clear the plot area
    plt.clf()

    # Set the visual theme for seaborn
    sns.set_theme(style="whitegrid")

    # Create a bar chart
    ax = sns.barplot(x="OrderYear", y="GrossRevenue", data=df_sales)
    plt.show()
    ```

4. 코드를 실행하고 seaborn을 사용하면 플롯에 일관된 색 테마를 설정할 수 있습니다.

5. Notebook에 새 코드 셀을 추가하고 Notebook에 다음 코드를 입력합니다.

    ```Python
    # Clear the plot area
    plt.clf()

    # Create a bar chart
    ax = sns.lineplot(x="OrderYear", y="GrossRevenue", data=df_sales)
    plt.show()
    ```

6. 코드를 실행하여 연간 수익을 꺾은선형 차트로 확인합니다.

> **참고**: seaborn을 사용한 그리기에 대한 자세한 내용은 [ 설명서](https://seaborn.pydata.org/index.html)를 참조하세요.

## <a name="delete-azure-resources"></a>Azure 리소스 삭제

Azure Synapse Analytics 탐색을 완료했으므로, 지금까지 만든 리소스를 삭제하여 불필요한 Azure 비용을 방지해야 합니다.

1. Synapse Studio 브라우저 탭을 닫고 Azure Portal로 돌아갑니다.
2. Azure Portal의 **홈** 페이지에서 **리소스 그룹**을 선택합니다.
3. (관리되는 리소스 그룹이 아닌) Synapse Analytics 작업 영역에 대한 **dp500-*xxxxxxx*** 리소스 그룹을 선택하고 Synapse 작업 영역, 스토리지 계정 및 작업 영역용 Spark 풀이 포함되어 있는지 확인합니다.
4. 리소스 그룹의 **개요** 페이지에서 **리소스 그룹 삭제**를 선택합니다.
5. **dp500-*xxxxxxx*** 리소스 그룹 이름을 입력하여 삭제 의사를 확인한 다음, **삭제**를 선택합니다.

    몇 분이 지나면 Azure Synapse 작업 영역 리소스 그룹과 여기에 연결된 관리 작업 영역 리소스 그룹이 삭제됩니다.
