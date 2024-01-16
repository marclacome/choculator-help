<link rel="stylesheet" href="./stylesheet.css" />  

<a href="intro.md">Home</a>

The url for all features api requests is https://features.chocolateemporium.com/public/api

<h3>Product search</h3>

<h3>Shopper</h3>  
Shopper search sends  
product-search with search params.  
Handled by ProductController->productSearch  (which calls productSearchWithPamCollections).  
This creates a mysql query and returns results from products table.  

<h3>CE</h3>
products table must be updated when products are added to Shopify.  
This is done in Choculator/  shopify-store () - truncates collections and products tables and sends graphql queries to Shopify to re-fetch data.  

<h3>Stock Notifications</h3>
<h3>Shopper</h3>  
If a product is out of stock the "Add to Basket" button is replaced with "Email when back in stock".  
This sends add-stock-notification.  
Handled by StockNotificationsController->addStockNotification which adds entry to stock_notifications table.  

<h3>CE</h3>
On page shopify-stock-notifications, component ShopifyStore/stock_notifications.vue mounted sends stock_notifications/get_all_notifications. This returns all in stock_notifications table.  
CE can select products which are back in stock and send emails.  
On sending email the stock notification entry is deleted from the table.  


<h3>Customer favourites</h3>
<h3>Shopper</h3>
Customer can add to favourites. This sends add-to-favourites.  
Handled by StockNotificationsController->addStockNotification.  
This adds a record to the  stock-notifications table.  

<h3>CE</h3>
CE has no functionality for this.










