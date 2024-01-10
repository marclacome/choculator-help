<link rel="stylesheet" href="./stylesheet.css" /> 

## Overview

The database contains a record of every product which is available on any e-commerce platform.
The Product table has 2 linked tables - 
Brand - for internal use only when filtering products.  
Batches - for tracking product quantities. 

### Product types 

Products are classed to enable calculations within Choculator.
Classes are 
+ giftwrap (for calculating order giftwrap total)
+ gifttag (for calculating order giftwrap total)
+ pam (for grouping display of loose v pam products in the packing page)
+ variety (this is a link to the id of an item in the VarietyBag table )

### Variety Bags
### Batches
### Active/inactive products
### SKU's
### Shopify search

On installation of the software, EComm/QueuedShopifyAPIRepository->getProducts is called.  
(This can only run once - if the Product table is not empty then the function will return).

This loops a query to Shopify fetching all products.
Each product is then processed by EComm/QueuedShopifyAPIRepository->addProduct
+ Add the product to the Product table
+ Add a batch to the ProductBatch table

Once initialised, products are added manually to the database when they are made available on any platform.

### Adding a product manually
On the add-product page entering a product sends products/store-product,  handled by Choc\ProductsController and Choc\ProductsRepository->store

### Editing an existing product
On the products-list page a product can be edited.  
This sends products/edit-product,  handled in Choc\ProductsController and Choc\ProductsRepository->storeProduct  
This creates both a Product and a ProductBatch

### Delete an existing product
Products cannot be deleted (in order to maintain data integrity for old orders).  
If a product is no longer stocked it can be marked as inactive in the active-products page. User can choose to hide inactive products in the front end display when listing/searching products






