# **Document Similarity Using Hadoop MapReduce**

## **Objective**  
The goal of this assignment is to compute the **Jaccard Similarity** between pairs of documents using **MapReduce in Hadoop**. This involves:
1. Extracting words from multiple text documents.
2. Identifying which words appear in multiple documents.
3. Computing the **Jaccard Similarity** between document pairs.
4. Outputting document pairs with similarity **above 50%**.

---

## **📏 Jaccard Similarity Calculation**

### **Formula**
The Jaccard Similarity between two sets A and B is calculated as:
```
Jaccard Similarity = |A ∩ B| / |A ∪ B|
```
Where:
- `|A ∩ B|` is the number of words common to both documents
- `|A ∪ B|` is the total number of unique words in both documents

### **Example Calculation**
Consider two documents:

**doc1.txt words**: `{hadoop, is, a, distributed, system}`  
**doc2.txt words**: `{hadoop, is, used, for, big, data, processing}`  

- Common words: `{hadoop, is}`  
- Total unique words: `{hadoop, is, a, distributed, system, used, for, big, data, processing}`  

Jaccard Similarity calculation:
```
|A ∩ B| = 2 (common words)
|A ∪ B| = 10 (total unique words)

Jaccard Similarity = 2/10 = 0.2 or 20%
```

---

## **🛠 Approach and Implementation**

### **Mapper**
- The **Mapper** processes each document and extracts words.
- It maps each word to the document it appears in.
- The output is in the format:
  ```
  word -> document
  ```
  Example:
  ```
  hadoop -> doc1
  hadoop -> doc2
  ```

### **Reducer**
- The **Reducer** receives all documents associated with a word.
- It constructs sets of words for each document and computes the Jaccard Similarity between all possible pairs.
- The output is:
  ```
  (doc1, doc2) -> Similarity Score
  ```
- Only pairs with similarity **above 50%** are included in the final output.

---

## **🚀 Running the Project**

### **📌 Step 1: Setup Hadoop Environment in Docker**

1. **Start the Hadoop cluster:**  
   ```sh
   docker-compose up -d
   ```

2. **Verify that Hadoop is running:**  
   ```sh
   docker ps
   ```

---

### **📦 Step 2: Build and Deploy the MapReduce Job**

1. **Compile the project using Maven:**  
   ```sh
   mvn install
   ```

2. **Move the compiled JAR to the shared folder:**  
   ```sh
   mv target/*.jar shared-folder/input/code/
   ```

3. **Copy the JAR file to the Hadoop container:**  
   ```sh
   docker cp shared-folder/input/code/DocumentSimilarity-0.0.1-SNAPSHOT.jar resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
   ```

4. **Copy the input document to the Hadoop container:**  
   ```sh
   docker cp shared-folder/input/data/document.txt resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/
   ```

---

### **📂 Step 3: Upload Data to HDFS**

1. **Access the Hadoop container:**  
   ```sh
   docker exec -it resourcemanager /bin/bash
   ```

2. **Create an input directory in HDFS:**  
   ```sh
   hadoop fs -mkdir -p /input/dataset
   ```

3. **Upload the dataset to HDFS:**  
   ```sh
   hadoop fs -put ./document.txt /input/dataset/
   ```

4. **Verify the uploaded files:**  
   ```sh
   hadoop fs -ls /input/dataset/
   ```

---

### **🚀 Step 4: Run the MapReduce Job**

Execute the MapReduce job using the JAR file:
```sh
hadoop jar /opt/hadoop-3.2.1/share/hadoop/mapreduce/DocumentSimilarity-0.0.1-SNAPSHOT.jar com.example.controller.DocumentSimilarityDriver /input/dataset /output
```

Check if the job ran successfully:
```sh
hadoop fs -ls /output
```

View the output results:
```sh
hadoop fs -cat /output/part-r-00000
```

Retrieve the output to the local machine:
```sh
hdfs dfs -get /output /opt/hadoop-3.2.1/share/hadoop/mapreduce/
```

Exit the container:
```sh
exit
```

Copy the output from the container to the shared folder:
```sh
docker cp resourcemanager:/opt/hadoop-3.2.1/share/hadoop/mapreduce/output/ shared-folder/output/
```

---

## **📊 Expected Output Format**

The output should show the Jaccard Similarity between document pairs in the following format:
```
(doc1, doc2) -> 60%
(doc2, doc3) -> 50%
```

---

## **⚠ Challenges and Solutions**

### **Challenge 1: Handling Large Datasets**
- Processing large text files can be slow.
- **Solution:** Used Hadoop's **combiner** feature to pre-aggregate intermediate results.

### **Challenge 2: Docker Container File Transfers**
- Moving files between local and Hadoop container was initially difficult.
- **Solution:** Used `docker cp` to copy files into and out of the Hadoop container.

### **Challenge 3: Ensuring Jaccard Similarity is Above 50%**
- Some document pairs had very low similarity scores.
- **Solution:** Applied a threshold check before writing output to HDFS.

---

## **📌 Conclusion**
This project demonstrated the use of **Hadoop MapReduce** for computing **Jaccard Similarity** between documents. The implemented approach successfully processed multiple documents, extracted unique words, computed similarities, and filtered results efficiently.

