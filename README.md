# Netflix Movies and TV Shows Dataset Analysis Using Apache Spark

**Apache Spark** is used to perform **Exploratory Data Analysis (EDA)** on the Netflix Movies and TV Shows dataset.

## **How to run the code**

### **1. Install Dependencies**
We need the following dependancies:
- **Java** for running Apache Spark.
- **Apache Spark** for distributed data processing.
- **findspark** to locate and initialize Spark in Python.

```python
# Install Java
!apt-get install openjdk-8-jdk-headless -qq > /dev/null

# Download Apache Spark
!wget -q https://dlcdn.apache.org/spark/spark-3.5.3/spark-3.5.3-bin-hadoop3.tgz

# Extract the Spark tarball
!tar xf spark-3.5.3-bin-hadoop3.tgz

# Install findspark
!pip install -q findspark
```

---

### **2. Environment Variables**

Set up Java and Spark paths.

```python
import os

# Set environment variables
os.environ["JAVA_HOME"] = "/usr/lib/jvm/java-8-openjdk-amd64"
os.environ["SPARK_HOME"] = "/content/spark-3.5.3-bin-hadoop3"
```

---

### **3. Initialize Spark Session**

Use `findspark` to initialize Spark and create a session:

```python
import findspark
findspark.init()

from pyspark.sql import SparkSession

# Create a Spark session
spark = SparkSession.builder \
    .appName("Netflix EDA") \
    .getOrCreate()
```

---

### **4. Load the Dataset**

Specify the file path to the Netflix dataset (CSV format) and load it into a Spark DataFrame.


### **5. Perform EDA**

#### **Check Dataset Shape**
```python
print(f"Rows: {netflix_df.count()}, Columns: {len(netflix_df.columns)}")
```

#### **Summary Statistics**
```python
netflix_df.describe().show()
```

#### **Check the number of missing values**
```python
from pyspark.sql.functions import col, sum

missing_values = netflix_df.select(
    [(sum(col(c).isNull().cast("int")).alias(c)) for c in netflix_df.columns]
)
missing_values.show()
```

#### **The number of movies vs tv shows**
```python
netflix_df.groupBy("type").count().show()
```

#### **The number of content added each year**
```python
from pyspark.sql.functions import year, to_date

netflix_df = netflix_df.withColumn("year_added", year(to_date(col("date_added"), "MMMM d, yyyy")))
content_by_year = netflix_df.groupBy("year_added").count().orderBy("year_added")
content_by_year.show()
```

#### **Number of content in each genre**
```python
from pyspark.sql.functions import explode, split

genres_df = netflix_df.withColumn("genre", explode(split(col("listed_in"), ", ")))
popular_genres = genres_df.groupBy("genre").count().orderBy(col("count").desc())
popular_genres.show(10)
```

#### **countries with most content on netflix**
```python
content_by_country = netflix_df.groupBy("country").count().orderBy(col("count").desc())
content_by_country.show(10)
```

#### **Distribution of ratings**
```python
ratings_distribution = netflix_df.groupBy("rating").count().orderBy(col("count").desc())
ratings_distribution.show()
```
