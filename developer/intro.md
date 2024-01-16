<link rel="stylesheet" href="./stylesheet.css" /> 

<a href="intro.md">home</a>

The CE software consists of 3 applications  
+ The shopify store 
+ Choculator
+ Features

<h3>Shopify Overview</h3>
The code is very customized, and virtually all data access and manipulation uses storefront-api graphql calls.  

<a href='shopify.md'>shopify docs</a>  

<a href='gift_vouchers.md'>gift vouchers docs</a>


<a id="choculator">
<h3>Choculator Overview</h3>
</a>

+ Stores an inventory of products
+ Downloads orders from Shopify, EBay and Amazon
+ Converts orders to internal format and caluclates stores in db
+ Calculates costs, vat, profit etc for each order
+ Produces a list of ingredients for an order
+ Tracks stock as orders are fulfilled
+ Calculates statistics for orders over a time period
+ Controls giftwrap options for products on the Shopify store
+ Manages the product search database for the Shopify store
+ Manages gift vouchers for the Shopify store
+ Manages customer requests for notification of out of stock items on the Shopify store

<a href='choculator.md'>choculator docs</a>  
<a href='choculator_products.md'>choculator products docs</a>  
<a href='gift_vouchers.md'>gift vouchers docs</a>


<a id="features">
<h3>Features  Overview</h3>
</a>
Adds auxiliary functionality to the Shopify store  
+ Stores shopper favourites
+ Stores request for notification of out of stock items
+ Stores details of gift voucher purchases
+ Handles shopper product search


<a href='features.md'>features docs</a>

<a href='gift_vouchers.md'>gift vouchers docs</a>


<h3>Shopify Store Source</h3>
+ github repo chocemporium_shopify
+ hosted on shopify 

<h3>Choculator Source</h3>
+ github repo choculator
+ hosted on Kualo in public_html/admin/choculator

<h3>Features Source</h3>
+ github repo choc_features
+ hosted on Kualo in public_html/features

<h3>Databases</h3>
Features - table is chocolat_shopify      
  
Choculator - uses table chocolat_choc1  for Choculator data, but also accesses choc_features.  
The Laravel models directory contains a subdirectoy - ShopifyStore.  
All models within ShopifyStore are declared with 
protected $connection='mysql_chocolat_shopify', which connects to choc_features instead of default chocolat_choc1.
