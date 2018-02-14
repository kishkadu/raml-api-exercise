# RAML API Models As An Exercise Around Customer API
This repository contains the RAMLs for Customer APIs. These APIs are designed using APILed Connectivity Pattern to meet objectives as follows.

## Objective
* API Should provide a CRED resource for Customer using RESTFul principals
* A Consumer may call every 5 min to get copy of the Customer records.
* Mobile App may call the API to retrieve / update customer details
* Extending the API to support Order and Products

## Applying API-led Connectivity Design
The API-led Connectivity is layered API Pattern where APIs are designed with one of three objectives/purpose
- Encapsulate the Connectivity to end System and provide anti corruption layer for the data model changes ( System APIs )
- Encapsulate the process/orchestration logic about particular function which fulfils specific business needs ( Process APIs )
- Simplify data model / operations so that these can be easily adopted by the Consumer. ( Experience APIs )

Applying API-led with purity always comes with tread-off of complexity and inefficient inter process communication. Another alternative to apply this pattern is to evaluate if each layer and evaluate if it provides the necessary benefits without worrying about cross layer communications.

With this consideration we will model the APIs in System and Experience layer for stated objective. This will allow good enough flexibility while still having a flexible design.
![Solution Design](images/Solution.jpg?raw=true "Solution Design")

Diagram about depicts the 2 APIs and the communication model with respective to API consumer.

## Anatomy Of The Designed APIs
### Customer System API
This is a System API component in our solution. All the interaction with the System Of Records where Customer data is resides happens through this layer.
The API provides the generic RESTful resources for managing Customer information in the System Of Records. The System API interfaces are designed with focus on capability of the end system rather than the requirement of the consumer. This way System API can help having the capability of end system offered in self-serving manner.

The API interfaces provide following operations on the Customer resource.
```yaml
/customers:
  get:
  post:
  /{id}:
    get:
    put:
    delete:  
```
The operation provides capability to retrieve list of customer, create new customer, retrieve specific customer using unique id, updating and able to delete such customer. These operation are generic enough to support most of the specific use cases.

The data model of the API is based on the data model of end systems (System Of Records) rather then what is required for particular use case. Such modelling supports wider range of data usages.

### Customer Experience API
The Experience layer component is focus more on making it easy for consumer and api developers leverages API  functions. Most of the times uses cases required specific data and specific operations. Using generic API add complexity to the use case construction. Experience layer will provide the simplified data modal and api resource particular to specific use cases.

The API interfaces provides following resource

```yaml
/customers:
  get:
  /updatedInlast5min:
    get:
  /{id}:
    get:
    put:
    /{fieldName}:
      patch:
    /orders:
      get:
      post:
    /products:
      /wishList:
        get:
      /recommendations:
        get:
/products:
  get:
```
While proxying the generic capability of the System API with simplified data model, Experience API offers resource for specific use case (like getting the list of customers updated in last 5 min). It also provides interface to unify data retrieved from different Process / System APIs (like Product/Order System APIs, Recommendation/Order Fulfilment Process APIs ) making it easy for consumers to leverages APIs effectively.

## Fulfilment Of The Objectives
#### Allow Create, Read, Edit, Delete on Customer records in System Of Records
While system API provides the basic Create, Read, Edit and Delete capability on Customer records as RESTFul resource. The System API hides the connectivity complexity from the consumers. However for most use case a Process or Experience API will be needed to add use case specific data or orchestration logic context.

#### Allow consumer to call APIs every 5 min and get copy of the Customer records
The System API provide the underline necessary GET resource which retrieves the list of customer from System Of Records. However retrieving all the records will be very expensive operation. A time period filter expressed using from and to timestamp query parameter has been added to that only the records updating in this time period will be retrieved.
The the network communication for this use case is shown in below diagram.
![Use Case 1](images/UC1.jpg?raw=true "Use Case 1")
The API consumer will use Experience API GET "/customers/updatedInlast5min" resource to get the customer records modified in last 5 min. While consumer could have call the System API directly with time period filter doing so will means consumer will have to adapt to the System API model and it's complexity. Experience API provdes the simplified data model as required by consumer with direct resource "/customers/updatedInlast5min" which cloud fulfils this requirement.

#### Allow consumer (Mobile App) to call APIs and retrieve, update Customer records
While System API provides RESTFul resource for retrieving and updating customers, it will add complexity on consumer side to
adapt the system API model as well have having to deal with the generic resources provided by System API.
The API interaction for this use case is as per the diagram below.
![Use Case 2](images/UC2.jpg?raw=true "Use Case 2")
The Mobile app will use the GET "/customer/{id}" resource to retrieve details of particular Customer.
The mobile app will use PUT "/customer/{id}" on Experience API to update multiple fields of a customer records. For both the functions, Experience API will use the underline System API resources with necessary transformation of data model.
The Experience API also offer a PATCH "/customer/{id}/{fieldName}" for Mobile app to quick update single field, The Experience API take cares of translating into System API PUT "/customer/{id}/" call.

#### Allow extension of the APIs to support Orders and Products
![Use Case 3](images/UC3.jpg?raw=true "Use Case 3")
Orders and Products resource will be incorporated into the Experience API with two main functional context
* Create/Get Order history for a particular customer, Get Product recommendations/wishList for particular customer.
The Experience API cloud have resource like "/customers/{id}/orders" with necessary filters to retrieve Order of particular Customer. Such capability need to be supported by underline Order System API which could interface with the Order Management System.
For Products "/customers/{id}/products/wishList" and "/customers/{id}/products/recommendations" can retrieve Product. These capability will come from the Recommendations process API.

* Get Product
Experience API can also have GET "/products" resource to get available Products

## Further Improvements
### Add pagination
The GET "customers" even with period cloud return large list of Customer records. Adding pagination to the underline System API GET method could help improve the performance of the system. Adding paginations is however depends on the end system capabilities.

### Single Experience API
For all the above use cases we have assume that is a same API consumer and thus designed single Experience API. But separate Experience API for each consumer can enable faster rate of change than having a common Experience API between multiple API consumer.