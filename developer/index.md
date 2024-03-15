To open a separate preview window, use the keyboard shortcut Ctrl+Shift+V
To open side by side, use the keyboard shortcut Ctrl+K V

# h1 header
## h2 header

[link](./index.md#linkhere)

### linkhere

# Shopify store
The store vue js components embedded in liquid code. \
The components are in snippets, using naming convention \
componentname_v.liquid = component vue code \
componentname_j.liquid = component js code \
componentname_vj.liquid = component vue and js code \

Mixins are stored in assets. \

All relevant vue source filenames in snippets and assets begin with prefix np_ \

## Initialisation
layout/theme.liquid includes assets/np.js \
on load, np.js calls doInit(), which fetches the cart and performs initialisation.

## Storefront API
The Storefront API is used exclusively for fetching product collections and editing and fetching the cart. \
Most transactions required metadata (either line item or cart). \
### A simple transaction - without metadata - (non pam product) \
### Fill a collection page with products
collection.liquid loads snippets\np_collection_grid_v.liquid (which loads snippets\np_collection_grid_j.liquid) \
np_collection_grid_j.liquid loads assets\np-collection-grid-mixin.js \
The np-collection-grid-mixin->mounted() calls loadMoreGridProducts, which sends a graphql query products in the current collection and displays them on the page.

### Add a product to the basket
Clicking "Add to Basket" on a product calls assets/np_buyfuncs->buyNonPam() \
which constructs an array of items to add to the cart and calls \
assets\np_graphql_funcs->graphBuy(), which creates and transmits a cartLinesAdd mutation. \
\
buy nonPam awaits response from graphBuy, then calls assets\np_cart->getVueCart \
which sends a graphql query cart \
\
on response to graphql query cart the relevant displays are updated













