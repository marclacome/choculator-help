<link rel="stylesheet" href="./stylesheet.css" /> 

<a href="intro.md">home</a>


<h3>Overview</h3>

The Shopify method of issuing Gift Vouchers doesn't meet CE's requirements.  
The solution is ridiculously complicated. (It might be possible to acheive what we're doing here without all the complexity, but I couldn't make sense of Shopify's documentation. so ...)  

What we're trying to achieve is -  
Shopper can enter a message and voucher recipient, and a delivery date. An email containing the message will be sent to the recipient on the delivery date.

1) The Shopify site contains a single gift card product - E-Gift-Voucher, available in several variants - £10, £20 etc.  
(The product the shopper buys must be a gift card product because CE require that shopper cannot apply a discount to the voucher. Shopify handles this restriction automatically for gift card products).  
This product is in the e-gift-voucher collection.

2) CE create a number of gift voucher codes and store the voucher codes in the gift_vouchers table database on shopify_features, with status field set to unallocated.


3) Shopper buys a variant of E-Gift-Voucher (but we don't send the gift card code).  


4) When fulfilling an order which contains a gift voucher, CE allocate a voucher code from the store to the gift voucher line item, and send the email to the recipient on the requested date. 


<h4>Shopify</h4>
Front end loads product from the e-gift-voucher collection (ie - E-Gift-Voucher product)
  

When the shopper adds the variant to the basket there is a prompt for message, recipient email, delivery date etc. These fields are stored as line item metadata in the basket.  
Since this item is a gift card, an email is automatically sent by Shopify to the shopper on checkout. The template for the email has been modified  - <a href='#email_template'>see here</a> and the voucher code is not sent.

<h4>Choculator</h4>
<h4>Processing gift voucher order items</h4>
When the order is processed in Choculator/packing, gift cards are handled by sending discounts/allocate-gift-card. 
Handled by ShopifyStore/DiscountsController and DiscountsRepository->allocateGiftCard.  
This copies the data stored in the line item meta to the gift_vouchers table, and updates the status field to allocated.

<h4>Sending gift voucher emails</h4>
In Choculator/shopify-gift-vouchers CE selects a date range.  
The table displays all emails in the data range where status != unallocated.  
CE sends emails from there.



<a id="email_template">email template</a>  

This is complicated. It has to handle 3 instances 
+ CE send a gift code to a customer (the normal scenario)
+ CE create a gift code to be stored in the database 
+ shopper buys a gift voucher 

These 3 are handled within the email template by checking gift_card.handle and customer.email, and sending either
+ a standard Shopify gift code email
+ an email to CE containing gift code and value to add to the database
+ an email to the shopper which confirms purchase of the gift voucher and date email will be sent to recipient


