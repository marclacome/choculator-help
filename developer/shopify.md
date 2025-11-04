<link rel="stylesheet" href="./stylesheet.css" />  

<a href="intro.md">Home</a>

[display products in the grid](#product_grid)  
[buy a product](#buying_products)  
[cart metadata](#cart_metadata)  
[giftwrap metadata](#giftwrap_metadata)  
[vue components](#vue_components)  
[quantity checking](#quantity_checking)   
[product search](#product_search)


<a id="product_grid">
</a>
## DISPLAY PRODUCTS IN PRODUCT GRID


There are 5 classes of products 
+ non-pam 
+ pam 
+ personalised gift boxes
+ gift vouchers 
+ subscriptions.

Products are displayed in an infinite scroll grid.
There are multiple grid templates with functionality for different product classes.
On navigating to a collection Shopify will load templates/collection.liquid.
This loads the required grid template (dependant on collection.handle)

### non pam collections
npProductGridVue created in np_collection_grid_j  
includes np-collection-grid-mixin

### pam collection
npProductGridVue created in np_collection_pam_grid_j  
includes np-collection-grid-mixin

### personalised gift boxes
There are 2 product grids for pgb. 
np_collection_pgb_main_grid (for the box which products are added to)
Once a box is selected np_collection_grid is loaded
np_collection_pgb_main_grid is simple and doesn't use infinite scroll (just uses liquid to list box products - there won't be many)
Once a box is purchased re-navigate to the collection for the box (the url will contain "-personalised-gift-box")
npProductGridVue created in np_collection_pgb_grid_v
includes np-collection-grid-mixin

### gift vouchers collection
npProductGridVue created in np_collection_gift_cards_grid_j  
includes np-collection-grid-mixin

### subscriptions 
npProductGridVue created in np_collection_subscriptions_grid_j  
includes np-collection-grid-mixin



### np-collection-grid-mixin handles infinite scroll 
(creates an event handler for scroll event which fetches products and adds results to the display)  
Products can fetched directly from Shopify (query products in collection)  
or an array of product id's can be fetched from Choc Emporium app Features (fetching search results or customer favourites)

np_collection_pam_grid has extra features  
display of the current pam bag, plus extra behaviour for the buy button (add line item metadata to add products to a specific pam bag)
np_collection_gift_cards_grid_j - extra behaviour for gift card metadata  
In pages which show multiple product classes (search results and customer favourites) np_collection_grid is used.  
The "buy button" display contains logic which hides the button and replaces with a link to relevant page for products which are included in pam and gift-voucher collections.  
This ensures that products cannot be added to the cart without relevant metadata (if an order is received with metatdata missing it probably means its been added to the wrong collection)  


<a id="product_search">  
<h3>non pam search  </h3>
locates to /pages/search-results?title=" + search;  
in Shopify, page search results uses template page.search_results  
which includes np_collection_grid_v  
np-collection-grid-mixin->mounted calls newSearchMounted  

### pam search
np_collection_pam_grid contains npPamCollectionVendorFilterVue and text input _pamsearch  
search is performed by calling np.js->doPamSearch    
which calls npPamCollectionVendorFilterVue->doSearch  
which calls np.js->pamSearchFunc  
which calls npProductGridVue->newPamSearch  
(which results in np-collection-grid-mixin->newTXSearchQuery)

### favourites  
locates to /pages/favourites  
in Shopify, page favourites uses template page.favourites  
which includes np_collection_grid_v  
np-collection-grid-mixin->mounted calls newSearchMounted  

<a id="buying_products"></a>
## BUYING PRODUCTS  
### NON PAM PRODUCTS  
Rules for buying a non pam product -   
If you buy multiple quantity of a product then giftwrap is not offered - products are directly added to cart with metadata for default giftwrap option, and no message.  
If you buy a single product which has no giftwrap option the product is directly added to cart with no metadata  
If you buy a single product which has a giftwrap option then a popup shows giftwrap options (giftwrap, tag and message),
once giftwrap selected adds product to the cart with giftwrap metatdata.  

Default giftwrap option means  
+ Giftwrap = default (Ribboned Cellophane Bag)  
+ No gifttag.  
+ No message.  

When a product is added to to the cart with no giftwrap, the cart is searched for a line item with that product id and no giftwrap.
If found then quantity of that line item is adjusted (Shopify mutation cartLinesUpdate), else product is added using Shopify mutation cartLinesAdd

When a product is added to to the cart with default giftwrap, the cart is searched for a line item with that product id and default giftwrap.
If found then quantity of that line item is adjusted (Shopify mutation cartLinesUpdate), else product is added using Shopify mutation cartLinesAdd

When a product is added to the cart with giftwrap, the product is added with cartLinesAdd (the line item will have message metatdata, which is assumed to be unique, therefore there cannot add multiple quantities of that line item)


The buy button calls np_collection_grid_j->buyproduct
calls assets/np_buyfuncs->buyNonPam  
This performs checks to see whether the giftwrap option needs to be selected
If no giftwrap option (no metatdata) - adds product to the basket 
else displays the popup for giftwrap option.  
Completing the giftwrap option calls np_buyfuncs/buyNonPamWithGiftwrap (adds line item to cart)


### PAM PRODUCTS
Rules for buying pam products
If the currently selected pam bag has giftwrap selected, and the giftwrap has an upper size limit, then you can't exceed that limit when adding products to the bag

### BUY PAM PRODUCT
[cart pam metadata](#cart_pam_metadata)  
[cart line item metadata](#cart_pam_line_item_metadata)
The buy button calls assets/np_buy_funcs/buyPamWithQuantity  
Creates or updates line attributes and quantity for the item   
If product already exists in cart then it is removed using Shopify mutation cartLinesRemove  
Product is added using Shopify mutation cartLinesAdd  

If the currently selected pam bag has giftwrap selected and adding products will exceed the giftwrap upper size limit, then call showPamBagFullPopup  
DIALOG npPamBagFullVue  
Dialog offers   
+ Select Default Giftwrap (no size limit) - call assets/np_bag_edit->changeGiftwrapOnBagFull. This adds the products and edits the giftwrap metatdata
+ Edit Selection - display pamBagVue (DIALOG pamBagVue). Don't add product.
+ Cancel - close popup and don't add product

### EDIT PAM PRODUCT QUANTITIES 
Quantity can be edited by the following ...  
Manually edit quantity in PamBagVue (np-pambag-v.liquid) calls np_bag_edit->savePamBagChangesAndUpdateCart  
Duplicate a bag (snippets\newpam-pambag-cart-expansion-header.liquid and PamBagVue np-pambag-v.liquid) calls np_bag_edit->copyPamBag  
Delete a bag (snippets\newpam-pambag-cart-expansion-header.liquid and PamBagVue np-pambag-v.liquid) calls np_bag_edit->deletePamBag  
Copy a bag from a previous order (snippets/np_order_history_j.liquid) calls assets/np_bag_edit->copyPreviousOrderPamBag  
All edit functions edit the cart using Shopify mutation cartLinesUpdate and cartLinesRemove, then query the cart.  
On response to cart query the internal representation of PAM data is updated.  

### BUY PGB PRODUCTS
You buy an empty box - a normal purchase transaction. Adding products to the box does not require a purchase transaction - products are just added to the attributes for the purchased box.
The PGB box must first be purchased
The attributes structure for the cart line item is created in createBoxEmptyAttributes
The box is then purchased with these attributes.
Purchasing an item calls buyPGBProductsWithQuantity
This simply updates the attributes "contents" field
Editing product quantities is also handled by buyPGBProductwWithQuantity


<div style="page-break-after: always"></div>

### CART ATTRIBUTES METADATA
(Used to store data for giftwrap for pam bags, and gift voucher data)  
<a id="cart_metadata">cart metadata</a>
```json
cart metadata   
{  
    cart: not used
    pam : array of pam bag - see below
    voucher: voucher data - see below
    version 1
}
```

<a id="cart_pam_line_item_metadata">CART PAM LINE ITEM METADATA</a>
An array of json objects - 1 for each bag.  
The object struct for a single bag is -   
```json
    {'giftwrap':{
        'giftwrap_data':{
            'id':'42926566834337',
            'title':'Brown Gift Box ',
            'image':'//bdidev.myshopify.com/cdn/shop/products/Brown_Gift_Box_small.jpg?v=1678360860',
            'price':'£3.25',
            'handle':'brown-gift-box',
            'size':'15'
        },
        'gifttag_data':{
            'title':'No gift tag',
            'id':-1,
            'price':'',
            'handle':-1,
            'image':''},
            'message':'',
            'set':true
        },
        'bag_name':1
    },
```

VOUCHER METADATA  
An array of json objects - 1 for each voucher.  
The object struct for a single voucher is - 
```json
        {key:'message', value:clean_message_str_non_pam(message)}, 
        {key:'email', value:email}, 
        {key:'sender',value:sender}, 
        {key:'delivery date', value:date}, 
        {key:'recipient', value:recipient}
```        

### CART LINE ITEM METADATA  
(used for giftwrap data for non pam items, and quantities per pam bag for pam items)

GIFTWRAP METADATA
```json
    [
        {
            "key": "_giftwrap",
            "value": "{'giftwrap_id':'42926566932641','gifttag_id':-1}"
        }
    ]
```  

<a id="cart_pam_metadata">CART PAM BAG METADATA</a>

```json
    [
        {
            "key": "_product",
            "value": "{'id':'gid://shopify/ProductVariant/42926586298529','handle':'lindt-lindor-caramel'}"
        },
        {
            "key": "_bags",
            "value": "[{'bag_name':1,'quantity':1},{'bag_name':2,'quantity':1}]"
        }
    ],    
```  
### INTERNAL REPRESENTATION OF CART & METADATA  
Cart is fetched by calling assets/np_cart->getVueCart (calls graphql query cart))  
On response to graphql query  
+ calls setNewPamCart (which sets newPamCart - the internal represenation of the cart)
+ calls updateVueCartAndEditableAttributes
+ creates json objects for 3 cart attributes - editablePamAttributes, voucherAttributes and cart_attributes

<a id="giftwrap_metadata"></a>
### GIFTWRAP METADATA
giftwrap metadata is loaded from Shopify as text strings hidden fields.
np.js initialisation calls the following functions which parse and create json objects for -
+ gwCollectionMeta = parseCollectionMeta();  in #gift_wrap_collection_meta
+ pamGiftwrapCollectionMetaObj = parsePamGiftwrapCollectionMeta(); in #pam_gift_wrap_collection_meta
+ giftwrapProductsObj = parseGiftwrapProductsObj('giftwrap') in #gift_wrap_collection_products or #pam_gift_wrap_collection_products
+ giftTags = parseGiftTags($('#gift_tags_str').val()); in  #gift_tags_str

  ### collectionMeta = giftwrap collection meta.
  2 fields - collection_enabled (not used - always true).  
  default (shopify id for the giftwrap product which is used if no specific giftwrap selected by the user - Ribbon Cellophane bag)

### giftwrapProductsObj
```json
  [
  { 
    "meta": {
      "productId": "gid://shopify/Product/7668163707041",
      "enabled": "1",
      "variants": [
        {
          "id": "42926566932641",
          "enabled": true,
          "pam_enabled": true
        }
      ],
      "pam_enabled": "1"
    },
    "products": [
      {
        "title": "Ribboned Cellophane Bag",
        "id": "42926566932641",
        "barcode": "FREE-GIFTWRAP",
        "price": "£0.00",
        "image": "//bdidev.myshopify.com/cdn/shop/products/cello-bag-view_small.png?v=1678360863",
        "handle": "ribboned-cellophane-bag-for-pick-and-mix",
        "variant": "Default Title"
      }
    ]
  },
  {
    "products": [
      {
        "title": "Brown Gift Box 125g",
        "id": "42926566703265",
        "barcode": "B125",
        "price": "£1.99",
        "image": "//bdidev.myshopify.com/cdn/shop/products/Brown_Gift_Box_small.jpg?v=1678360860",
        "handle": "brown-gift-box",
        "variant": "125g"
      },
      {
        "title": "Brown Gift Box 250g",
        "id": "42926566736033",
        "barcode": "B250",
        "price": "£2.25",
        "image": "//bdidev.myshopify.com/cdn/shop/products/Brown_Gift_Box_small.jpg?v=1678360860",
        "handle": "brown-gift-box",
        "variant": "250g"
      } 
      etc
    ],
    "meta": {
      "productId": "gid://shopify/Product/7668163674273",
      "enabled": "1",
      "variants": [
        {
          "id": "42926566703265",
          "enabled": true,
          "pam_enabled": true
        },
        {
          "id": "42926566736033",
          "enabled": true,
          "pam_enabled": true
        } 
        etc
      ],
      "pam_enabled": "1"
    }
  },
  {
    "products": [
      {
        "title": "Cream Gift Box 125g",
        "id": "42926587019425",
        "barcode": "",
        "price": "£1.99",
        "image": "//bdidev.myshopify.com/cdn/shopifycloud/shopify/assets/no-image-100-c91dd4bdb56513f2cbf4fc15436ca35e9d4ecd014546c8d421b1aece861dfecf_small.gif",
        "handle": "cream-gift-box",
        "variant": "125g"
      },
      {
        "title": "Cream Gift Box 250g",
        "id": "42926587052193",
        "barcode": "",
        "price": "£2.25",
        "image": "//bdidev.myshopify.com/cdn/shopifycloud/shopify/assets/no-image-100-c91dd4bdb56513f2cbf4fc15436ca35e9d4ecd014546c8d421b1aece861dfecf_small.gif",
        "handle": "cream-gift-box",
        "variant": "250g"
      },
      etc
    ],
    "meta": {
      "productId": "gid://shopify/Product/7668168622241",
      "enabled": "1",
      "variants": [
        {
          "id": "42926587019425",
          "enabled": true,
          "pam_enabled": true
        },
        {
          "id": "42926587052193",
          "enabled": true,
          "pam_enabled": true
        }
        etc
      ],
      "pam_enabled": "1"
    }
  }
]
```

### gift tags
[
  {
    "title": "No gift tag",
    "id": -1,
    "price": "",
    "handle": -1,
    "image": ""
  },
  {
    "title": "Swing Tag with Printed Gift Message",
    "id": "42926567030945",
    "price": "£0.35",
    "handle": "free-gift-tag",
    "image": "//bdidev.myshopify.com/cdn/shop/files/swing-tag_small.jpg?v=1687192268"
  }
]

## VUE COMPONENTS
### HEADER BAR 
[npSearch](#npSearch)
+ pamCustomerLoginHeaderVue
+ pamCartHeaderCountVue

### MAIN CONTENT  
### COLLECTIONS
Normal (non pam) collection
+ [npProductGridVue](#npProductGridVue) (np_collection_grid)
+ [productPopupVue](#productPopupVue)
+ [informNoStockPopupVue](#informNoStockPopupVue)
+ [pamGiftwrapVue](#pamGiftwrapVue)  

if collection.handle = pick-and-mix
+    [npProductGridVue](#npProductGridVue) (np_collection_pam_grid)
+    [pamProductPopupVue](#pamProductPopupVue)
+    [npPamBagFullVue](#npPamBagFullVue)
+    [npPamCollectionVendorFilterVue](#npPamCollectionVendorFilterVue)
+    [pamBagViewHeaderVue](#pamBagViewHeaderVue)
+    [pamBagCompleteHeaderVue](#pamBagCompleteHeaderVue)
+    [pamBagVue](#pamBagVue)
+    [informNoStockPopupVue](#informNoStockPopupVue)
+    [pamGiftwrapVue](#pamGiftwrapVue)  

if collection.handle = personalised-gift-boxes 
  no vue components - just a simple liquid grid

if collection.handle contains personalised-gift-box (ie - christmas-personalised-gift-box)
 + [npProductGridVue](#npProductGridVue)
 + [pgbBoxViewHeaderVue](#pgb_box_view_header_vue)
 + [PGBBoxVue](#pgb_box_vue)


if collection.handle = gift-cards
+ [npProductGridVue](#npProductGridVue) (np_collection_gift_cards_grid)  

if collection.handle = gift-vouchers  
+ [npProductGridVue](#npProductGridVue)  (np_collection_gift_cards_grid)

if collection.handle = e-gift-voucher  
+ [npProductGridVue](#npProductGridVue)  (np_collection_gift_cards_grid)

if collection.handle = subscriptions  
+ [npProductGridVue](#npProductGridVue)  (np_collection_subscriptions_grid)
+ [subscriptionsPopupVue](#subscriptionsPopupVue)

### CART 
+ [pamBagVue](#pamBagVue)
+ [full_cart](#full_cart)

### CUSTOMER 
+ [npAddressesVue](#npAddressesVue)
+ [pamCustomerLoginPopupVue](#pamCustomerLoginPopupVue)
+ [npOrderHistoryVue](#npOrderHistoryVue)

### VUE COMPONENTS SOURCE FILES
<a id="vue_components"></a>

### PAM COMPONENTS

<a id="pamCartHeaderCountVue">pamCartHeaderCountVue</a> (and pamCartHeaderCountVue_Mobile)
+ np_pam_cart_header_count.js
+ #count_header_cart_vue in theme.liquid
+ theme.liquid

                        
<a id="pamBagSizePopupVue">pamBagSizePopupVue</a>
+ np_bag_size_popup_vj.liquid
+ #pam_bag_size_popup in np_bag_size_popup_vj.liquid
+ cart.liquid

<a id="npPamBagFullVue">npPamBagFullVue</a>
+ np_pam_bag_full_dialog.liquid
+ #pam_bag_full_dialog_vue
+ collection.liquid and page.pam-favorites.liquid

<a id="npPamCollectionVendorFilterVue">npPamCollectionVendorFilterVue</a>
+ np_pam_collection_vendor_filter-vj
+ #pam_collection_vendor_filter in np_pam_collection_vendor_filter 
+ np_collection_pam_grid_v.liquid 

<a id="pamProductPopupVue">pamProductPopupVue</a>
+ np_pam_product_popup_j.liquid
+ #pam_product_popup_vue  in np_pam_produdct_popup_v.liquid
+ collection.liquid, page.order-history.liquid, page.pam-favourites.liquid

<a id="productPopupVue">productPopupVue</a>
+ np_product_popup_j.liquid
+ #product_popup_vue in np_product_popup_v.liquid
+ collection.liquid, page.order-history.liquid, page.favourites.liquid, page.search_results.liquid

<a id="subscriptionsPopupVue">subscriptionsPopupVue</a>
+ np_subscriptions_popup_j.liquid
+ #product_popup_vue in np_subscriptions_popup_v.liquid
+ np_collection_subscriptions_grid_v.liquid

<a id="pamGiftwrapVue">pamGiftwrapVue</a>
+ np-pam-gift-wrap-popup-j.liquid
+ #pam_giftwrap_vue in np-pam-gift-wrap-popup-v.liquid
+ theme.liquid

<a id="pamBagVue">pamBagVue</a>
+ np_pambag-j.liquid
+ #pam_bag_vue in np-pambag-v.liquid
+ np_collection_pam_grid_v.liquid

<a id="pamBagCompleteHeaderVue">pamBagCompleteHeaderVue</a>
+ np-pambag-complete-header-j.liquid
+ #pam_bag_complete_header_vue in np-pambag-complete-header-v.liquid
+ np_collection_pam_grid_v.liquid

<a id="pamBagViewHeaderVue">pamBagViewHeaderVue</a>
+ np-pambag-view-header-j.liquid
+ #pam_bag_view_header_vue in np-pambag-view-header-v.liquid
+ np_collection_pam_grid_v.liquid


### PGB COMPONENTS
+ <a id="npProductGridVue">npProductGridVue
 + [pgbBoxViewHeaderVue](#pgb_box_view_header_vue)
 + [PGBBoxVue](#pgb_box_vue)



### PRODUCTS IN GRID
<a id="npProductGridVue">npProductGridVue</a> (many variations of this component in different files)
+ np_collection_grid_j.liquid, np_collection_pam_grid_j.liquid, np_collection_gift_cards_grid_j.liquid, np_collection_subscriptions_grid_j.liquid
+ #productgrid, #pamgrid
+ np_collection_gift_cards_grid_v.liquid, np_collection_grid_v.liquid, np_collection_subscriptions_grid_v.liquid, np_collection_pam_grid_v.liquid

### CART
<a id="full_cart">full_cart</a>
+ np-vue-cart-j.liquid
+ #full_cart in cart.liquid
+ templates/cart.liquid

### CUSTOMER
<a id="npAddressesVue">npAddressesVue</a>
+ np_customer_address_j.liquid
+ #customer_addresses in page.addresses.liquid
+ page.addresses.liquid

<a id="pamCustomerLoginHeaderVue">pamCustomerLoginHeaderVue</a> USED???
np-customer-login-header-vj.liquid
#pam_customer_login_header_vue in np-customer-login-header-vj.liquid
theme.liquid

<a id="pamCustomerLoginPopupVue">pamCustomerLoginPopupVue</a>
+ np-customer-login.liquid
+ #pam_customer_login_vue in np-customer-login.liquid
+ theme.liquid

<a id="npOrderHistoryVue">npOrderHistoryVue</a>
+ np_order_history_j.liquid
+ #order_history in page.order-history.liquid
+ page.order-history.liquid


### SEARCH
<a id="npSearch">npSearch</a> (incl mobile version)
+ np_search_vj.liquid and np_search_mobile_vj.liquid
+ #header_search_vue in np_search_vj.liquid and #header_search_mobile_vue  in np_search_mobile_vj.liquid
+ theme.liquid

<a id="pamSearchVue">pamSearchVue</a> (used? as a popup?)
+ np-search-popup-j.liquid 
+ #pam_search_vue  in np-search-popup-v.liquid
+ theme.liquid


### PRODUCT OUT OF STOCK
<a id="informNoStockPopupVue">informNoStockPopupVue</a>
+ np_inform_no_stock_vj.liquid
+ #inform_no_stock_popup_vue in np_inform_no_stock_vj.liquid
+ theme.liquid

<a id="notifyStockEmailPopup">notifyStockEmailPopup</a>
+ np_notify_out_of_stock_email_popup_vj.liquid
+ #notify_no_stock_email_popup_vue in np_notify_out_of_stock_email_popup_vj.liquid
+ theme.liquid

### MISC
<a id="cookiesPopupVue">cookiesPopupVue</a>
+ np_cookies_popup_vj.liquid
+ #cookies_popup_vue in np_cookies_popup_vj.liquid
+ theme.liquid

<a id="newPamLoadingDialog">newPamLoadingDialog</a>
+ np-loading-dialog-vj.liquid
+ #pam_loading_vue in np-loading-dialog-vj.liquid
+ theme.liquid

<a id="npPamHowToPopupVue">npPamHowToPopupVue</a>
+ np_pam_howto_popup_vj.liquid
+ #pam_howto_popup_vue in np_pam_howto_popup_vj.liquid 
+ theme.liquid

<a id="cartDebugVue">cartDebugVue</a>
+ np_cart_debug.liquid
+ #cart_debug in np_cart_debug.liquid
+ theme.liquid

<a id="quantity_checking"></a>
### CHECKING QUANTITIES  
### PRE PURCHASE
When a product is fetched for display in the grid the available quantity is included in the graphql request.
When shopper adds a product to basket, the requested quantity is checked against available quantity.
If available quantity is 0 then the product is displayed but the buy button is replaced with a "request email when re-stocked" button.
If shopper attempts to buy a quantity greater than available, then 
a) all available stock is added to the basket
b) on return from the purchase transaction, a popup is displayed to inform that all available have been purchased, and a button to request email when re-stocked.

Checking available stock is complicated.
Shopify doesn't update stock levels until shopper checks out, so quantities already in the basket have to be subtracted from the Shopify reported stock levels.

It can happen that shopper A adds a low stock item to the basket but shopper B exhausts available stock and checks out first.
In that case the product will still show in the basket for shopper A, but as part of Shopify's checkout procedure it will be removed.
There is nothing we can do to avoid this situation. It will cause problems where metadata has been created which relates to quantities in the basket (ie. Shopify will edit quantities, but we already have created the metadata). Chocolate  Emporium have been informed that this is a limitation of the system.


### POST PURCHASE
The cart contents are monitored whenever the cart is updated to ensure that contents match expectations.  
This is achieved by the checksObject in attributes/np_checks.js  

The procedure is -   
In whichever function adjusts cart quantity (either by adding an item or editing quantity of an existing item)
+ getChecksObject().reset()
+ add new item and quantity to the checks Object (calling either checksObject.addPam or checksObject.addNonPam)
+ update the cart 
+ fetch the cart (np_cart.fetchCart)
+ call checksObject.checkCart  
### checkCart checks
+ pam line item meta (line item quantity must match total count of product allocated to pam bags in line item meta)
+ gitftwrap meta (total count of each giftwrap product listed in line item meta must match the line item quantity for that giftwrap product)
+ line items with 0 quantity (I think this is due to a bug in Shopify - a cart line item can exists with a quantity of 0. It just looks messy for the shopper so these items are removed)
+ quantity purchased (the quantity of each line item added or edited on the most recent graphql call must match the quantity requested in that graphql call. These quantities are added using checksObject.addPam or checksObject.addNonPam as listed above)





