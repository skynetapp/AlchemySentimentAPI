#### Date: 26-10-2016
#### Description: This document aims to define the Alchemy Sentiment API coding 


#### The Folder Structure is as follows:
   
   
   Root Directory | Sub Directory | Sub Directory 
------------ | ------------- | -------------
index.php | | |
Global | DBmongo(Mongo Connection),DBmysql(MySQL Connection), AlchemyAPI(Alchemy API Connection)  | 
Lib | Smarty,Common functions,AlchemyAPI | |
Modules | AlchemySentiment | Alchemy Sentiment Controller, Alchemy Sentiment Action, Alchemy Sentiment MySql, Alchemy Sentiment View, Alchemy Sentiment DB Mongo|
Views | AlchemySentiment | header.tpl, footer.tpl(Common files), masterList.tpl,detailList.tpl|

#### Architecture

1. Read data from MySQL Table where SentimentExtractionStatus='' and SentimentExtraction=0 (Table name - BlueMixAlmEntityExtractReq)
2. Invoke Watson by calling Alchemy API (sentiment) which calls the **api key** and gets response from API for multiple records.
3. The response is processed and updated in Parent Table (master_sentiment_request) and the corresponding children are stored in child table name (children_sentiment_request). 
4. Updating the processed records status after getting the response. 
5. The raw response from Watson is also stored in MongoDB (lytepole) as raw JSON file.

#### Code Flows as follows:
   * To insert or get data from DB code flows.. index.php -> Controller -> Action -> MySql.
   * To view the data code flows.. index.php -> Controller -> View.
   
 
#### Step 1:
  Add the created Url and Alchemy Key in the config.php under bluemix2.0 folder.we can get the Alchemy API Key by logging into IBM Bluemix. 
	
**_Code:_**
	
```
	
$GLOBALS['alchemy_apiKey']='xxxxxxxxxxxxxxxxxxxxxxxxx';
$GLOBALS['alchemy_url']='https://gateway-a.watsonplatform.net/calls';
	
```
	
  
#### Step 2:
  Create a module name as 'alchemySentiment' in index.php and respective actions will be performed accordingly.
  
#### Step 3:
   In the server normally we will be able to see existing Master list data. When we click on the update button **GetList** action will be performed from index page and **getDataFromAlchemyDB()** will be called.
   
#### Step 4:
   From index.php, **AlchemyExtractController** class will be called which controlles all the operations of Alchemy Sentiment module. Here function **getDataFromAlchemyDB()** will be executed.
   
#### Step 5:
   This **getSentimentTextData()** function gets the multiple records data from Request table and sends request to Alchemy API response function **sentiment('text', $text, null)** one by one using for loop. The Alchemy API will be in Lib folder.
   
#### Step 6:
   On receiving response from API, the JSON response will be stored in Mongo DB by calling function  **insertAlchemyJSONResponseIntoMongo(json_encode($data))**.

#### Step 7:
   The response array will be inserted into Master data by function **insertSentimentMasterDataIntoMySQL($data,$id);** from Controller -> Action -> MySql.
   

#### Step 8:
   Here based on the master request id, the response will be stored in Child table by function **getParsedDataFromJSONResponse($masterReqId)**.


#### Step 9:
   On inserting the JSON response into master and child tables, the status and Request date will be updated for that record in the Request table by function **updateSentimentTextData($id)** in controller.


#### Step 10:
   To get the Master list function **getALLMasterDataFromMySQL** will be called from controller.
   
#### Step 11:
   To view the Master list function **showSentimentMasterListView()** will be called from controller to View.
   
**_View page Code:_**

```
  function showSentimentMasterListView($data_arr){
        $smarty = new Smarty();
        $smarty->assign('base_path',$GLOBALS['base_path']);
	$smarty->assign('cursor',$data_arr);
		
	$smarty->display(''.$GLOBALS['root_path'].'/Views/AlchemySentiment/allMasterList.tpl');
    }
    
``` 

#### Step 12:
   To view the Child data based on the master id, function **getSentimentChildDataFromMySQL($post_data)** will be called from controller.
   Function **getAllChildDataFromMySQL($post_data)** will get the records based on the master id using MySql query. Function **showChildDetailListView($alchemy_list_vo)** will be called in view. 
 
 
 #### Assumptions

- DBMongo - Inserts the Alchemy JSON response into mongo. It is included in Global -> DBMongo.


#### Errors

If any erros found, the following may be the reasons.

- Url might not be correct. It should be as below.

**_Url:_**

```
http://159.203.239.91/bluemix2.0/index.php?module=alchemySentiment&action=GetMasterList

```
- The root path, base path and the database name should be correct in config.php.
- The alchemy API key should be a valid key from IBM Bluemix. If the key is expired, the results may not be appeared.
- If the day limit is exceeded for the API key, then also the child results will not be inserted. 


#### MySQL Database Details

  
 Database Name: bluemix
 
 Tables | Description | Fields 
------------ | ------------- | ------------
BlueMixAlmEntityExtractReq | Request table to get records where SentimentExtractionStatus='' and SentimentExtraction=0 | |
master_sentiment_request | Stores the extracted data based on the request id | alm_id, alm_request_date, alm_external_id, alm_sentiment_response_text |
child_sentiment_request | Stores the child records based on master id | child_sentiment_id, master_sentiment_id, sentiment_type, sentiment_score, alc_date |
 
 
#### Mongo Database details
 
- Database Name: lytepole
- Table or collection name : BlueMixEntityExtract
- Description: Mongo stores the JSON response given by the Alchemy API for all the records.

- To start the mongoDB, open putty.
- To start the MongoDB service type command - **mongo**.
- To display the database, type command - **db**. It will return **test** as default database. To use our database type command **use dbname**.
- Type command **exit** to quit the mongo shell.



