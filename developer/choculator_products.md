<link rel="stylesheet" href="./stylesheet.css" /> 

## Overview

The database contains a record of every product which is available on any e-commerce platform (many platforms are not sole on all platforms)
The Product table has 4 linked tables - 
+ Brand - for internal use only when filtering products.  
+ <a href="#batches">Batches</a> - for tracking product stock levels.
+ <a href='#variety_bags'>Variety Bags</a> - see below

### Product types 

Products are classed to enable calculations within Choculator.
Classes are 
+ giftwrap (for calculating order giftwrap total)
+ gifttag (for calculating order giftwrap total)
+ pam (for grouping display of loose v pam products in the packing page)
+ variety (this is a link to the id of an item in the VarietyBag table )

<a href='#installation'>
<h3>Installation</h3></a>
<a href='#products'> 
<h3>Products </h3>
</a>
<a href="#sku">
<h3>SKU's</h3>  
</a>
<a href='#batches'>
<h3>Batches</h3>  
</a>
<a href='active_products'>
<h3>Active/inactive products</h3>  
</a>
<a href='variety_bags'>
<h3>Variety Bags</h3>
</a>

<a id="installation"></a>
On installation of the software, EComm/QueuedShopifyAPIRepository->getProducts is called.  
(This can only run once - if the Product table is not empty then the function will return).

This loops a query to Shopify fetching all products.
Each product is then processed by EComm/QueuedShopifyAPIRepository->addProduct
+ Add the product to the Product table
+ Add a batch to the ProductBatch table

Once initialised, products are added manually to the database when they are made available on any platform.

<a id="products"></a>
### Producs
### Adding a product manually
On the add-product page entering a product sends products/store-product,  handled by Choc\ProductsController and Choc\ProductsRepository->store

### Editing an existing product
On the products-list page a product can be edited.  
This sends products/edit-product,  handled in Choc\ProductsController and Choc\ProductsRepository->storeProduct  
This creates both a Product and a ProductBatch

<a id="active_products"></a>
### Delete an existing product
Products cannot be deleted (in order to maintain data integrity for old orders).  
If a product is no longer stocked it can be marked as inactive in the active-products page. User can choose to hide inactive products in the front end display when listing/searching products


<a id="sku"></a>
<h3>Product SKU</h3>
Every product must have an sku which uniquely identifies the product on each platform on which it is sold.  
These are copied from the admin product pages of the platform and copied to Choculator.  
There are 10 columns in the product table for platform sku's - only 3 are currently used. These are mapped in EComm/EcommOrderConverterRepository->mapSKU(shopify = sku1, amazon = sky2, ebay=sku3)

<a id="batches"></a>
<h3>Batches</h3>
Product stock levels are updated by the user when product is purchased, and automatically adjusted when orders are fulfilled in Choculator.
The db contains a batches table which contains entries for each product which stores quantity and purchase price.  
There can be multiple batch entries for a product - a new batch of product can be added to an existing batch at a different purchase price.  
When an order is processed and there are multiple batches for a product then batches are adjusted oldest first.  
Batches are adjusted when <a href='./choculator.md#processing_orders'>processing orders</a>

<a id='variety_bags'></a>
<h3>Variety bags</h3>

CE sell variety bags by putting a number of individual products in a bag and selling them as a single product (ie - 10 Lindt Lindor Milk Chocolate).  
When an order is processed Choculator needs to adjust 
+ stock level for the individual products
+ calculate the total cost of fulfilling the order  
To achieve the above, variety bags are stored in the DB as a table of product_uuid and quantity.


bags in variety_bags  
product per bag in variety_bag_products

variety bag from order - variety_bag_order_records






