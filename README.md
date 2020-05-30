# Recommender

<h1>Neighborood recommender<h1>
<h2>Business problem</h2>
When you rent a place for vacations, or for an internship, in another city that you don't know, it can be difficult to choose a neighborood. Some neighborood are quiet with parks, others have a lot of restaurants...
  
The aim of this recommender is to provide a list of neighborood similar to a neighborood that you like, in targeted cities.
You enter a neighborood in a city that you know and like, the name of the targeted city, and the recommender will provide to you neighborood similar to the one that you enjoy in the targeted cities. It could help people to choose a place for vacations or living when they don't have the possibility to visit the targeted city before (for example, interns, students in international exchanges...).

<h2>Data</h2>
For the scope of this projects, I will start with three cities: Paris, New York and Singapore. We are limited to 950 requests per day in the foursquare API, so I didn't wanted to add too much cities.

For Paris, I will generate the districts numbers. For Singapore and New York, I will use data from wikipedia: Singapore:https://en.wikipedia.org/wiki/Postal_codes_in_Singapore New York: https://fr.wikipedia.org/wiki/Liste_des_quartiers_de_New_York.
The result is a list of cities and neighboroods.
['Singapore,Upper Thomson',
 'Singapore, Springleaf',
 'Singapore,Yishun',
 'Singapore, Sembawang',
 'Singapore,Seletar']…

I will then use these names to localize each neighborood using geolocator. The result is a dataframe with neighboroods localized.

	City	Latitude	Longitude	Neighborood	full_adress
275	New York	43.521737	-75.965196	Lighthouse Hill	New York,Lighthouse Hill
276	New York	40.634548	-74.112087	West New Brighton	New York,West New Brighton
277	New York	40.642326	-74.092919	New Brighton	New York,New Brighton
278	New York	40.643963	-74.073442	St. George	New York,St. George
279	New York	40.621215	-74.131809	Westerleigh	New York,Westerleigh

Geolocator provided to me some wrong locations: I have dropped them.
 
In order to assign features to each neighborood, I will use the foursquare API (https://developer.foursquare.com/docs/places-api/) in order to retrieve the places and their features for each neighborood. Foursquare provides venues (restaurants, parks, yoga studios etc... within a determined radius around a point.
Foursquares provides data in json format. 

We can easily retrieve the venues from this result:
results['response']['groups'][0]['items']

After grouped venues by neighborhoods, I obtained a dataframe with 264 neighborhoods, and 389 venues.
 

<h2>Methodology</h2>
I have worked on data retrieved from Foursquare (one-hot encoding then average by neighboroods to sort top venues by neighborood.
 
I have then tried to cluster neighboroods, using a k-means and DBscan algorithms.
I first tried Kmeans:
 
 
I was not happy with the results. No real Elbow, and very unbalanced clusters if I wanted more than three clusters.

I tried DBscan: it returns three clusters with 82 outliers with the best epsilon. It was not usable.

I assumed that the problem was that my data were very sparsed: I have  264 rows  and  389  features.

So I did some feature engineering and reduce the number of venues categories: I decrease them from 389 to 16, grouping them manually (restaurants, shops, nature etc…).
 

I then ran again kmeans and dbscan, and the results where much better.
 
 
For DBscan, I obtained 4 clusters, and I only had 23 outliers left.

So I decided to go for kmeans (with optimized K=9 using Elbow and Silhouette method).
The result was better!
 
I have decided to work with kmeans, with the meta categories. Two reasons for this choice:
- I have more clusters, meaning my recommender will be more specialized
- I don't have outliers. Outliers will be a problem with my recommender. What if the initial place is an outlier?
- Looking at the values in my cluster, dbscan has in fact one big cluster and the other ones are very small. Kmeans have more balanced clusters.
Because I lost some informations using meta categories, I have decided to offer to the user the possibility to choose one precise feature that I want to have in the chosen neighborhood.
Here are the steps:
- First, the user choose a feature he absolutly wants.
- Then he enters the neighborhood he likes
- And the targeted city
- The answer is a neighborhood in the targeted city, within the same cluster than the initial target, with the feature he absolutly wants (if exists in the cluster).

Here the result:

Please choose a feature than you like.Pizza Place
Please choose a neighborood that you like (format city, Neighborood)Singapore, Bukit Panjang
Please choose the city where you want to moveNew York

Here the places similar to the neighborood that you love, in New York and with a Pizza Place !

66                  New York,Annadale
72                   New York,Astoria
78                   New York,Bayside
79             New York,Bayside Hills
81              New York,Bedford Park
92                 New York,Bronxdale
93                  New York,Bruckner
108             New York,Dongan Hills
122             New York,Far Rockaway
125                 New York,Flushing
126         New York,Flushing Heights
129             New York,Forest Hills
139              New York,Great Kills
145                New York,Hillcrest
149             New York,Howard Beach
153          New York,Jackson Heights
158        New York,Kew Gardens Hills
163                New York,Laurelton
183              New York,Murray Hill
186                 New York,New Dorp
187            New York,New Hyde Park
 

 
 

<h2>Results</h2>
Due to data sparsity, I obtained better results with feature engineering. However, my recommender is less precise. I tried to overcome this problem adding the choice of one favorite precise feature.

<h2>Conclusion</h2>
This recommender can be improved a lot.
-	For meta-categories creation: I did it manually. Use of NLP will be more efficient.
-	With more cities and more data, the problem of sparsity will decrease. 
-	I haven’t tried hierarchical clustering.

