## Aggregating Emails Using Logic Apps.
The organization I work for is a vehicle claims manager, which does vehicle assessment for insuracne companies.
The organization has had a booking system since 2006 which was written in asp.

Recently a power app has been deployed in our organization, which allows users to view legacy bookings.

By legacy bookings, I mean that these booking were created by an application which has now been decommissioned.

However, for legal reasons, the data has to be readily available to our users. The data about the bookings are now stores in an azure sql server database and the documents and photos are stored in blob storage.

In the power app, Any time a search is done, an entry is written into a storage table as a log with the following payload. This allows us to track the usage of the app.
```
{
    "email":"david.o@vehicle_pty_ltd.com.au",
    "search":"Burna",
    "created":"2021-03-09"
}

```

The created field is used as the partition key for the storage table.

### Logic App
To help improve efficency, the business will like to have some form of reporting on the usage.
One simple metrics will be to have a summary of usage count by users. As follows
```
[
    {
        "email":"david.o@vehicle_pty_ltd.com.au"
        "count":23,
        "date":2021-03-09"
    },
    {
        "email":"kizz.daniel@vehicle_pty_ltd.com.au"
        "count":2,
        "date":2021-03-09"
    }
]
```

In any traditional programming, this is quite easy, see pseudocode,

```
    init email_log = get_log_from_storage_table(date_var)

    def metricObject {
        email,
        count,
        date
    }

    init hashmap as  <string, metricObject>

    foreach item in email_log
        if(item.email in hashmap)
            oldObject = hashmap.get(item.email)
            hashmap.put(item.email, new metricObject(item.email, oldObject.count + 1, date_var))

        else
            hashmap.put(item.email, new metricObject(item.email, 1, date_var))

```
## Logic App Implementation

- Create an array to store all the email addresses.
![alt text](images\01.png "Initialize an array to store all email addresses")


- Create another array which will store only the unique email addresses.
![alt text](images\02.png "Initialize an array to store all unique email addresses")


- Get all the entities , use a schema to have access to all the elements of the json result.
![alt text](images\03.png "Get all the entities by partition key.")

- Loop thru the results and append the email addresses to the all_email_addresses array.
![alt text](images\04.png "Loop thru results")

- Create an array of unique email addresses, using the Union data operation.
![alt text](images\05.png "Get unique email address only.")


- Next Loop thru the unique email addresses and create a filtered array using the current email address as the filter value.

A foreach loop could be used to itterate thru the original results and count the number of emails which match.

However this approach requires the concurrency flag to be set and the loop has to be done sequentially. 

This can result in an O(n*m) complexity. The original implementation using this approach took 1m to loop thru 9 unique email address from 26 elements in the result set.
![alt text](images\06.png "Loop thru unique email addresses and create filtered array")

- Compose a result element , with the count value equal to the lenght of the body from the previous step.
![alt text](images\08.png "Compose result elment")


- Append the result element into a final result array
![alt text](images\09.png "Compose result elment")


**Below are the input and output json files used in this implementation**

[Input json](images\input.json)

[Output json](images\output.json)