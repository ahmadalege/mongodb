# **MongoDB Querying and Aggregation Project**


In this project, I will not be doing any major CRUD operation as I will use mongimport to create the database, collection, and to import the documents which are in JSON format.

> [!NOTE] 
> I will be using the mongo shell (mongosh) to carry out all the operations and not Mongo Atlas or any other third-party application.


First, we need to login to the mongoDB server using 'mongosh' command and check our existing databases to ensure I'm not just working with a pre-existing database.

![1  checking existing databases](https://github.com/ahmadalege/mongodb/assets/131969880/3fb3509f-d517-43eb-805d-3dc115be4662)

We can see we have four databases after running the 'show dbs' command. they are:

 ```
 1. admin
 2. config
 3. family
 4. local
```

Now we can logout and use the mongoimport from our terminal. Note that you have to be in the location of the file to import i.e. the json file. The commands are
 ```
mongoimport --db newdb --collection books --type json books.json
mongoimport --db newdb --collection restaurant --type json restaurant.json 

```
The above commands do three things:
1. creates a new database newdb
2. creates collections books and restaurant
3. lastly, it loads the books.json and restaurant.json files into the books and restaurant collections respectively.

![2  import and check collections](https://github.com/ahmadalege/mongodb/assets/131969880/81ff40eb-d285-48d8-83b9-46bf637de13b)

After logging back into the mongoDB server, we can see that we now have five databases with newdb now present and the collections as well.
now to get into querying.
To get an idea of the fields and content of the documents, we query only the first documents in both collections using:
 ```
db.restaurant.find().limit(1)
db.books.find().limit(1)
```
This is what we get

![3  comfirm documents have imported](https://github.com/ahmadalege/mongodb/assets/131969880/32c497af-65ce-4d40-aa9f-5fe2f1886623)

In the restaurant collection, we have the following fields:
* _id
* URL
* address
* name
* outcode
* postcode
* rating
* type of food
  
And in the books collection, we have:
* _id
* title
* isbn
* pageCount
* PublishedDate
* thumbnailURL
* shortDescription
* LongDescription
* status
* authors
* categories

>To get the restaurants with rating greater than 5
we use the
 ```
db.restaurant.find({$rating:{$gt:5}});
```
![4  find restaurants with rating gt 5](https://github.com/ahmadalege/mongodb/assets/131969880/b81d4d22-3604-4455-88ff-e8bc685d46ae)


>To get restaurants serving Pizza, Kebab, and Afhgan dishes
 ```
db.restaurant.find({$type_of_food:{$in: ['Pizza', 'Kebab', 'Afghan']}})
```
![5  $In operator food in](https://github.com/ahmadalege/mongodb/assets/131969880/d7bafd3d-d753-484a-970a-49d17491f48c)


>To get restaurants selling Kebabs **and** rating greater than 5.5
 ```
db.restaurant.find({$and:[{$type_of_food:'Kebab'},{$rating:{$gt:5.5}}]})
```
![6  $and operator](https://github.com/ahmadalege/mongodb/assets/131969880/88ea85d2-3dda-4bc5-89a5-499d61512167)


All the querying we've done have been with the find command, now we want to into aggregations such as Match, Group, Project, Count and Sort.

# $Match is used to filter like find but it is an aggregation operation.
>To find a book titled 'MongoDB in Action'
 ```
db.book.aggregate([
{$match: {title: 'MongoDB in Action'}}
])
```
![6  $match](https://github.com/ahmadalege/mongodb/assets/131969880/7c6b3dce-6121-4015-b66e-f02c08cdd76c)

# $Project is used to include, exclude, or add a new field. 
It can also be used together with other aggregates to get even better results.

>To find the book MongoDB in action and display the title, author, and categories fields
```
db.books.aggregate([
{$match:{title:'MongoDB in Action'}}, {$project:{_id:0, title:1, authors:1, categories:1}}])
```
> To include the _id and pageCount fields in the above result
```
db.books.aggregate([
{$match:{title:'MongoDB in Action'}}, {$project:{ title:1, authors:1, categories:1}}])
```
![7  match and project](https://github.com/ahmadalege/mongodb/assets/131969880/1f17a74c-90e6-4fc0-ad96-2a1687c68e3a)


# $Count is used to count documents
>To count the total number of documents in both collections
 ```
db.books.aggregate([{$count:"total_count"}])
db.restaurant.aggregate([{$count:"total_count"}])
```
![8  count total documents](https://github.com/ahmadalege/mongodb/assets/131969880/87b6214b-ccb3-48c2-a654-6fadd562452d)

We can see have 431 documents in books and 2548 documents in the restaurant collection.

# $group is used just as in SQL, it is used to group documents, and then other operations can be performed on the groups. Let's see a basic grouping operation:

>To see the books categories and food types in the restaurants
 ```
db.books.aggregate([{$group:{_id: "$categories"}}])
db.restaurant.aggregate([{$group:{_id: "$type_of_food"}}])
```

>To group by multiple fields
 ```
db.books.aggregate([{$group:{_id:{category:"$categories", book_status:"$status"}}}, {$limit:10}])
```
![12  group multiple fields](https://github.com/ahmadalege/mongodb/assets/131969880/cacaa0af-4332-4154-8ed6-02ecbee4ce03)

We grouped by categories and book status then limited the output to only 10 documents

>To see count the number of book categories and types of food in the restaurants
 ```
db.books.aggregate([{$group: {_id: "$categories}}, {$count: "Total_no_of_categories"}])
db.restaurant.aggregate([{$group:{_id: "$type_of_food}}, {$count: "Total_no_of_food_count}])
```
![9  group and count](https://github.com/ahmadalege/mongodb/assets/131969880/81b717ab-d906-46ff-8418-5a253be5b7d4)

As we can see in the output, we have 58 book categories in the books collection and 52 food types in the restaurant collection

>To group by categories and food type, count of documents in each groups then sort by the count in descending order
 ```
db.books.aggregate([{$group:{_id:"$categories", count: {$sum: 1}}}, {$sort: {count:-1}}])
db.restaurant.aggregate([{$group:{_id:"$type_of_food", count: {$sum: 1}}}, {$sort: {count:-1}}])
```
![10  group, count and sort](https://github.com/ahmadalege/mongodb/assets/131969880/3cb6eb82-2f55-4402-aaca-85aad1df9230)

What the above query does is, it groups by categories and takes 1 as the value for all the items in the group, sums it all together then returns it as the variable count then sorts based the count in descneding order


>To match and group on first n documents, then count
 ```
db.restaurant.aggregate([ {$limit: 100}, {$match:{rating:{$gt:5.5}}}, {$group:{_id:"$type_of_food", count:{$sum:1}}} ])
```
![11  comp'ex query](https://github.com/ahmadalege/mongodb/assets/131969880/5c08254f-ac9c-408c-a2a3-6b5da688bd0f)

The first query filters for only food type (group) with a rating greater than 5.5 in the 50 documents then counts the items in the group i.e number of restaurants that meet the filter condition, we only got curry and pizza with only one restaurant both.

Then we checked the first 100 documents and we got the same result. 
Then we adjusted the filter conditions and only searched for restaurants with a rating of 5.0 and above. Then we got:
* Chinese: 7
* Curry: 3
* Thai: 1
* Pizza: 13

