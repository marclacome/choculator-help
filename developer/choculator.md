<link rel="stylesheet" href="./stylesheet.css" />  

<a href="intro.md">home</a>



## Orders/DownloadOrders
### Downloading Orders
download_orders.vue 
includes download_monitor.vue
sends ecomm/api-queue-init with array of sources to download and date range.  
Handled by EComm/OrdersQueueController
then download_monitor->doQueueLoop - repeatedly calls ecomm/api-queue  
handled by EComm/APIQueueController 
+ loads n orders from a source
+ converts orders to internal fomat and stores
+ returns cursor (where to start next download)
repeats until the source is completed then moves to next source till all are completed.

APIQueueController downloads orders by calling 
+ EComm\QueuedShopifyAPIRepository
+ EComm\QueuedAmazonAPIRepository
+ EComm\QueuedEBayDBRepository

Orders are converted to internal storage format using 
+ DB\ShopifyDBRepository
+ DB\AmazonDBRepository
+ DB\EBayDBRepository


<a id="processing_orders">
### Processing Orders

Once download is complete user selects Process Orders.  
Sends orders/process-orders-unprocessed-order-items.  
Handled by EComm/OrderProcessController and OrderProcessRepository->processOrders_UnprocessedOrderItems. This loops through each unprocessed order and    
+ adjusts stock quantity for each line item in the order
+ creates an OrderItem record for each line item

## Packing  

Sequence for fulfilling an order is 
+ open packing page - which fetches all unfulfilled orders
+ select an order to fulfill. This fetches customer and order details.
+ Edit order items if required, then confirm items
+ Manually enter shipping cost (other costs - ie vat, packaging - can also be edited at this time)
+ Mark the order as fulfilled


On loading the packing page, orders/get-unfulfilled-orders is sent.  
Handled by Choc/OrdersController and Choc/OrdersRepository  
Returns all orders where status != [fulfilled, refunded]  

User selects an order from the drop list, which sends 
packing/get_customer and packing/get_order_details  
customer details are not stored locally so are fetched by an api call to the relevant platform.  
order details are returned from the db  

Confirm Items  
sends orders/process-order 
Handled by Choc/OrdersController and EComm/OrderProcessRepository->ProcessOrderCostsAndBatches
+ updates stock levels for each order item
+ calculates and stores vat, fees, revenue and profit

Manually enter shipping cost  
Sends order/edit-order  
Handled by Choc/OrdersController and Choc/OrderRepository->editOrder  
+ Sets totals and fees for the order (order_total, items_total, giftwrap etc)
+ calls Choc/OrderProcessRepository->ProcessOrderCostsAndBatches to recalculate vat, fees etc based on updated user input  
+ calls Choc\OrderIngredientsRepository->setOrderIngredients to set the ingredients field based on ingredients for each line item product

Mark fulfilled
sends order/set-order-fulfilled  
Handled by Choc/OrdersController and Choc/OrdersRepository->setOrderFulfilled  
Sets status to fulfilled, and fills the fulfilled_date

### Problem Orders  
An order cannot be processed correctly if it contains products which are not present in the products table.  
In this case the order is processed field is set to problem.
The Problem Orders page allows the product table to be updated to include the required product. The problem order can then be deleted, downloaded again and processed.


### Manual Orders
An order can be entered manually and products from the database can be added.
This allows orders outside of e-commerce to be created. 
It can also be used as a solution if an order will not download from an e-commerce platform - the order can be replicated with a manually created order without having to wait for a software solution to the download problem. 







-------------------------------------------------------------

new order status default is new,  
processed default is pending  
ingredients processed default is pending
can_fulfill is false

after Processing Order processed is items

after ProcessOrderCostsAndBatches processed is confirmed