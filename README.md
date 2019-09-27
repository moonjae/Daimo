# DAIMŌ Intro 
DAIMŌ is a web service that provides analytics on residential properties located in Seoul.
We have been harnessing modern machine learning technologies to figure out which property is undervalued and by how much.
The prediction showed reliable outcome with mean absolute error of ₩30,000,000 which is considerably low when compared to the average price of property in Seoul in approximately ₩100,000,000. 

### Disclaimer
 - Please do not use DAIMŌ solely for investment purposes
 - Our software serves to help you find the best option for you but it won't tell you when to purchase the property 
    - Our algorithm does not do time series forecasting 
    - Our algorithm does not take account of macroeconomic or political events. It simply analyzes features of a residential property to estimate a resonable price for it 
  - Our software collects data from [Naver]
  - The website may not display all the properties listed on Naver for the following reasons
    - It may take time for our crawler backend to crawl recently added properties on Naver 
    - Properties with missing information are being omitted on purpose since it can negatively affect the prediction accuracy 
- MAE of 0 would mean that every property is valued at the right price which means that there is no price gap between the expected price and the market price 
- Our software keeps updating and discarding the training dataset so, the accuracy of the prediction might fluctuate depending on the traning dataset being used at the moment 


### How it works 
#### Design 
##### Backend (Django, EC2, Nginx, Postgresql)
- Crawler keeps retrieving data from naver endpoint 
- Gradient Boosting Regressor predicts price values 
- Processes user requests and returned json data (REST API)
    - Query requests ex) property list, favorites list 
    - Add to favorites request 
    - Retrieves Google OAuth token from the frontend and process user login (The backend simply uses the token to retrieve user information so it can be stored in the database and reused when the same user attempts to login again)

##### Frontend (React)
- Google Oauth Authentication (OAuth is done at frontend level)
- Displays information requested by the users 

#### Logic 
```python
    def task_daimo(self):
        while(True):
            # records when the crawling cycle started
            time = datetime.now(tz=timezone.utc)
            
            # crawler 
            crawler = Crawler()
            crawler.crawl()
            
            # gradient boosting regressor 
            GBR = GradBoost()
            # get rid of data that is older than 3 months 
            GBR.delete_old_data()
            GBR.import_data()
            # use data older than time variable as traning set 
            GBR.train(time)
            # predict values for data collected during this crawling cycle 
            GBR.predict()
```
### Gradient Boosting Regressor 
Regression methods that came to my mind at first were Random Forest, Gradient Boosting and Neural Nets. Since my features or explanatory variables were either numerical or categorical (non-image), I thought neural nets would not really help much so, I narrowed my choices down to Random Forest and Gradient Boosting Algorithm. I tested both of them out and gradient boosting regressor outperformed random forest in most cases. 

#### List of features 
- Address (categorical)
- Common Area Space (numerical)
- Exclusive Use Space (numerical)
- Direction (numerical)
- Floor (numerical)
- Highest Floor (numerical)
- Time needed to get to the closest subway station (numerical)
- Type of Entrance (categorical)
- No of parking permits per unit (numerical)
- No of family (numerical)
- No of family per area (numerical)
- No of rooms (numerical)
- No of bathrooms (numerical)
- Avg utility fee (numerical)
- Built Date (numerical)
- Is the building built by a well known company? (categorical)
    -  The following companies are regarded as top construction companies 
    -  '삼성물산', '삼성건설', '대림산업', '현대건설', 'GS건설','지에스건설', '대우건설', '포스코건설', '현대산업개발', '롯데건설', '호반건설', '에스케이건설','SK건설' ,'한화', '두산건설'
- Distance to closest bus stop (numerical)
- Distance to closest subway station (numerical)
- Distance to closest childcare (numerical)
- Distance to closest preschool (numerical)
- Distance to closet school (numerical)
- Distance to closest hospital (numerical)
- Distance to closest parking lot (numerical)
- Distance to closest mart (numerical)
- Distance to closest convenience store (numerical)
- Distance to closest laundry room (numerical)
- Distance to closest bank (numerical)
- No of buses available (numerical)
- No of subway lines available (numerical)
- Distance to Yeouido (numerical)
- Distance to Hongdae (numerical)
- Distance to Euljiro (numerical)
- Distance to Gasan (numerical)
- Distance to Jamsil (numerical)
- Max price of property at the same address (numerical)
- Min price of property at the same address (numerical)


#### General Steps
1. Drop Instances with missing feature values that are key to explaining the correlation
2. Drop outliers so they don't distort the training model 
3. Hot encode the categorical variables 
4. Use GridsearchCV to tune the hyperparamets for better prediction accuracy
5. Train
6. Predict 


[Naver]: <https://land.naver.com/>

