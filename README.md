##### Level of the data and timeframe 
* The data is imported at 'imp_path' (query), 'sku' (document), 'imp_position' level. Total clicks and Total impression are the two measures imported
* The timeframe of the trainig data is current_date-10 to current_date-3 and current_date-2 to current_date-1 will be used for testing


##### Filters applied
* All the 'imp_path' having 'clothing' or 'shoes' are filtered for from the above data. This captures the entire clothing and shoes category
* A sku must have atleast 10 impressions over a week
* A query must have atleast 10 skus in it

##### Click model
* Click model is used to get a relationship between total number of impressions across different positions and total clicks obtained
* This relationship will then be used to calculate the expected total click value (Predicted clicks)
* Ratio = (Actual clicks/Predicted clicks)
* Ratio captures how an sku performed in actual, compared to what was expected. Values > 1 indicate skus being more relevant and <1 indicates skus being less relevant to the query.
* A relevance_score is defined based on the ratio, which ranges from 0-4 with 4 indicating a strong relevance and 0 indicating no relevance

##### Features for the model
* product attributes such as:
    * Brand, category, department, gender, subcategory, color etc.
    * Time since launch and time since relaunch
    * Launched in the last 7/14/30 days
    * Discount percentage, average product availability etc.
* Historical product performance (current_date-17 to current_date-11):
    * Total clicks/impressions received by the skus
    * CTR
    * Computing the CTR at different levels: Brand,Query-Brand, Category,Query-Category, etc.
    * Computing CTR at combination of levels: Brand-Category-Gender, Brand-Subcategory-gender etc.
    
##### Training
* Trained using XGboost algorithm and 'pariwise ranking' method
* currently giving cross validation NDCG of 78%-80%
* Test NDCG of ~76%

#### Observations from the model
* Sicne the relevancy score defined relies just on the ratio, skus having smaller click count could be ranked on top of say, Top 25 items by clicks for a query
* This could lead to cases where some items get high relevancy score purely by chance and could mislead the model
* One idea to overcome this is to do stratified relevance scoring, wherein:
    * The entire skus space of a query will be divided into N parts based on the total clicks (N could 2 or 3)
    * The logic for partition could be the click contribution, where Top 50% click contributors = Partiton 1 and Next 50% click contributors = partition 2
    * The relevance score for skus can be assigned at query-partiton level instead of just query level 
