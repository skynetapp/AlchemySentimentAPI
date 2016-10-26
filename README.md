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

#### Code Flows as follows:
   * To insert or get data from DB code flows.. index.php -> Controller -> Action -> MySql.
   * To view the data code flows.. index.php -> Controller -> View.
   
 
#### Step 1:
  Add the created Url and Alchemy Key in the config.php under bluemix2.0 folder.
	
**_Code:_**
	
```
	
$GLOBALS['alchemy_apiKey']='75c6700778eda02ffce454aed26b51f1faad5f54';
$GLOBALS['alchemy_url']='https://gateway-a.watsonplatform.net/calls';
	
```
	
  
#### Step 2:
  Create a module name as 'alchemySentiment' in index.php and respective actions will be performed accordingly.
  
**_Code:_**

```
<?php
if($_REQUEST['module']=='alchemySentiment'){
    switch ($_REQUEST['action']){
        case 'GetList':
        {
			//echo "1";exit; 
            $alchemyController = AlchemySentimentController::getInstance();
            $alchemyController->getDataFromAlchemyDB();
            break;
        }
		 case 'GetMasterList':
        {
			//echo "1";exit; 
            $alchemyController = AlchemySentimentController::getInstance();
            $alchemyController->getMasterListData();
            break;
        }
        case 'DetailList':
        {
            $alchemyController = AlchemySentimentController::getInstance();
            $alchemyController->getSentimentChildDataFromMySQL($_REQUEST);
            break;
        }
      
    }
}
?>

```

#### Step 3:
   In the server normally we will be able to see existing Master list data. When we click on the update button **GetList** action will be performed from index page and **getDataFromAlchemyDB()** will be called.
   
#### Step 4:
   From index.php, **AlchemyExtractController** class will be called which controlles all the operations of Alchemy Sentiment module. Here function **getDataFromAlchemyDB()** will be executed.
   
#### Step 5:
   This **getSentimentTextData()** function gets the multiple records data from Request table and sends request to Alchemy API response function **sentiment('text', $text, null)** one by one using for loop.
   
#### Step 6:
   On receiving response from API, the JSON response will be stored in Mongo DB by calling function  **insertAlchemyJSONResponseIntoMongo(json_encode($data))**.

#### Step 7:
   The response array will be inserted into Master data by function **insertSentimentMasterDataIntoMySQL($data,$id);** from Controller -> Action -> MySql.
   
**_ MySql Code:_**

```  
 public function insertAllSentimentMasterDataIntoMySQL($data,$id){
		//print_r($data); exit;
		$this->response_array =$data;
		$masterId=$this->insertIntoMasterData(json_encode($data),$id);
        if($masterId>0)
		return $this->getParsedDataFromJSONResponse($masterId);
	}
	
	
	public function insertIntoMasterData($json_response,$id){
    	$sql = "INSERT INTO master_sentiment_request
               (
                alm_sugar_id,
                alm_sentiment_response_text,
                alm_request_date,alm_external_id
                ) VALUES (
                
                '',
                '$json_response',
                NOW(),
				'$id'
                )";
		mysqli_query($this->con,$sql);
        return $this->con->insert_id;
    }

```

#### Step 8:
   Here based on the master request id, the response will be stored in Child table by function **getParsedDataFromJSONResponse($masterReqId)**.

**_Code:_**

```
   public function getParsedDataFromJSONResponse($masterId){
            $row['master_sentiment_id']=$masterId;
			$row['mixed']=$this->response_array['docSentiment']['mixed'];
            $row['sentiment_score']=$this->response_array['docSentiment']['score']; 
		    $row['sentiment_type']=$this->response_array['docSentiment']['type'];
			 $this->insertIntoRowMysqlTable($row);
    }

     public function insertIntoRowMysqlTable($rowData=array()){
    	$sql = "INSERT INTO child_sentiment_request(
                    master_sentiment_id,
					mixed,
                    sentiment_type,
                    sentiment_score,
                    alc_date
                ) VALUES (
                    '".$rowData['master_sentiment_id']."',
					'".$rowData['mixed']."',
                    '".$rowData['sentiment_type']."',
                    '".$rowData['sentiment_score']."',
                    NOW()
                )";
        mysqli_query($this->con,$sql);
        return $this->con->insert_id;
    }
    
```

#### Step 9:
   On inserting the JSON response into master and child tables, the status and Request date will be updated for that record in the Request table by function **updateSentimentTextData($id)** in controller.


#### Step 10:
   To get the Master list function **getALLMasterDataFromMySQL** will be called from controller.
   
#### Step 11:
   To view the Master list function **showSentimentMasterListView()** will be called from controller to View.
   
**_Code:_**

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
   
**_Code:_**

```
public function getSentimentChildDataFromMySQL($post_data){
        $alchemy_action = AlchemySentimentAction::getInstance();
        $alchemy_list_vo = $alchemy_action->getAllChildDataFromMySQL($post_data);
		$alchemy_view = AlchemySentimentView::getInstance();
    	$alchemy_view->showChildDetailListView($alchemy_list_vo);
	}
```
