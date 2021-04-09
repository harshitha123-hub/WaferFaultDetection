# WaferFaultDetection

## Problem Statement
To build a classification methodology to predict the quality of wafer sensors based on the given training data. 
## Data Description
* The client will send data in multiple sets of files in batches at a given location. Data will contain Wafer names and 590 columns of different sensor values for each wafer. The last column will have the "Good/Bad" value for each wafer.
* "Good/Bad" column will have two unique values +1 and -1.  
* "+1" represents Bad wafer.
* "-1" represents Good Wafer. 
* Apart from training files, we also require a "schema" file which contains all the relevant information about the training files such as:
  * Name of the files, Length of Date value in FileName, Length of Time value in FileName, Number of Columns, Name of the Columns, and their datatype.


## Architecture
![image](https://user-images.githubusercontent.com/60007389/114236450-3f8d3280-999f-11eb-92fa-0bc43535496c.png)
### Data Validation

we perform different sets of validation on the given set of training files.  
  **1. Name Validation**- We validate the name of the files based on the given name in the schema file. We have created a regex pattern as per the name given in the schema file
        to use for validation. After validating the pattern in the name, we check for the length of date in the file name as well as the length of time in the file name. If all         the values are as per requirement, we move such files to "Good_Data_Folder" else we move such files to "Bad_Data_Folder."
  
  **2. Number of Columns** - We validate the number of columns present in the files, if it doesn't match with the value given in the schema file then the file is moved to              "Bad_Data_Folder". 
  
  **3. Name of Columns** - The name of the columns is validated and should be same as given in the schema file. If not, then the file is moved to "Bad_Data_Folder". 
  
  **4. Datatype of columns** - The datatype of columns is given in the schema file. This is validated when we insert the files into Database. If dataype is wrong then the file           is moved to "Bad_Data_Folder". 
  
  **5. Null values in columns** - If any of the columns in a file has all the values as NULL or missing, we discard such file and move it to "Bad_Data_Folder".

### Data Insertion in Database 
  1) Database Creation and connection - Create database with the given name passed. If the database is already created, open the connection to the database. 
  2) Table creation in the database - Table with name - "Good_Data", is created in the database for inserting the files in the "Good_Data_Folder" on the basis of given column        names and datatype in the schema file. If table is already present then new table is not created, and new files are inserted the already present table as we want training        to be done on new as well old training files.     
  3) Insertion of files in the table - All the files in the "Good_Data_Folder" are inserted in the above-created table. If any file has invalid data type in any of the columns,      the file is not loaded in the table and is moved to "Bad_Data_Folder".

### Model Training 
  1) Data Export from Db - The data in a stored database is exported as a CSV file to be used for model training.
  2) Data Preprocessing   
    a) Check for null values in the columns. If present, impute the null values using the KNN imputer.
    b) Check if any column has zero standard deviation, remove such columns as they don't give any information during model training.
  3) Clustering - KMeans algorithm is used to create clusters in the preprocessed data. The optimum number of clusters is selected by plotting the elbow plot, and for the            dynamic selection of the number of clusters, we are using "KneeLocator" function. The idea behind clustering is to implement different algorithms
     To train data in different clusters. The Kmeans model is trained over preprocessed data and the model is saved for further use in prediction.
  4) Model Selection - After clusters are created, we find the best model for each cluster. We are using two algorithms, "Random Forest" and "XGBoost". For each cluster, both        the algorithms are passed with the best parameters derived from GridSearch. We calculate the AUC scores for both models and select the model with the best score. Similarly,      the model is selected for each cluster. All the models for every cluster are saved for use in prediction.


### Prediction 
 
  1) Data Export from Db - The data in the stored database is exported as a CSV file to be used for prediction.
  2) Data Preprocessing    
    a) Check for null values in the columns. If present, impute the null values using the KNN imputer.
    b) Check if any column has zero standard deviation, remove such columns as we did in training.
  3) Clustering - KMeans model created during training is loaded, and clusters for the preprocessed prediction data is predicted.
  4) Prediction - Based on the cluster number, the respective model is loaded and is used to predict the data for that cluster.
  5) Once the prediction is made for all the clusters, the predictions along with the Wafer names are saved in a CSV file at a given location and the location is returned to the        client.
 ### Final Prediction Output
 
 **1. HomePage**
   ![Wafer](https://user-images.githubusercontent.com/60007389/114239355-9d237e00-99a3-11eb-887b-766759a4124d.jpg)
 
 **2. Prediction Page**
  ![wafers](https://user-images.githubusercontent.com/60007389/114239416-b2001180-99a3-11eb-8d06-f2ffd717ba16.jpg)
