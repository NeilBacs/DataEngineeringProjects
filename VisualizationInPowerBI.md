### Distribution of ratings
### created histogram together with cards

![image](https://github.com/NeilBacs/DataEngineeringProjects/assets/107451251/2650905d-6c14-47b1-a84b-52b020ff0bf0)
### noticed the histogram is quite similar to normal distribution shape, it's just positively skewed
#### And I calculated the My percentile using DAX "My Rapid Percentile = COUNTROWS(FILTER('Ratings', 'Ratings'[rapid] < 'Ratings'[MyRapidRating]))/COUNTROWS(Ratings)*100"
#### And computed 77% percentile which gives me insight that I have a decent rating

### next is tactics rating 

![image](https://github.com/NeilBacs/DataEngineeringProjects/assets/107451251/f22fb469-5181-42be-a194-33af76f8e9cc)

### the shape of the histogram is quite different, most fall in the lower end and it has a very long tail
### Surprisingly I fall in the 92nd percentile, which means there are players that were higher rated in me in rating but lower in me in tactics rating.


### Now I want to look for the  correlation between tactics rating and rapid rating
### I use scatter plot and perform linear regression

![image](https://github.com/NeilBacs/DataEngineeringProjects/assets/107451251/6b5de2af-3c39-4278-85f0-37f656fa45b7)

### By eyeballing the graph, we can see that the trend line is upward which indicates a positve correlation
### But to know how strong the correlation is, I went back to python and compute it

### But first, I get the data by querying in SQL then save the result to csv
```sql
SELECT *
FROM MyChessDotComClub..Members a INNER JOIN MyChessDotComClub..Joined_Details b ON a.member_id = b.id
INNER JOIN MyChessDotComClub..Ratings c ON b.id = c.id
```
### now import it in python
```python
res = pd.read_csv(r'Portfolio_DataEngineering\result.csv')
correlation = res['tactics'].corr(res['rapid'])
print(correlation)
```
### the result of correlation is 0.4802090323388481 which means it is moderately correlated 
### We can conclude that tactics and rapid rating has moderately positive correlation
