# Amazon online shopping system

In this blog, I am sharing my learnings from various resources, focusing on the high-level and low-level design of Amazon. In this section, I have captured my key takeaways.

## Actors & Use Cases

### Primary Actors 

- Authenticated Users
	- Register Account
	- Login/Logout
	- Update/Delete the account
	- Search Product
	- Add Item to the shopping list
	- Update the Shopping
	- Checkout Shopping Cart
	- Add Shipping Address
	- Add Credit Card
  	     
- Guest Users
	- Register Account
	- Search Product
	- Add Item to Shopping Cart
 	- Update Shopping Cart
    
### Secondry Actors

- Admin
	- Login/Logout   	
	- Add, Modify, Delete Product Category
	- Block/Unblock/Update/Delete Account
	- Modify Product  	 
- System
	- For sending Notifications like order notification, shipping notifications.  

## LLD 

### Components 

- Customer
	- Guest
	- AuthenticatedUser
 - Admin
 	```
	+ blockUser(account):bool
  	+ addNewProductCategory(category):bool
  	+ modifyProductCategory(category):bool
 	+ deleteProductCategory(category):bool 	
  	```   
 - Account
   ```
   - userName: string
   - password: string
   - name: string
   - email: string
   - phone: string
   - status: AccountStatus
   - shippingAddress: Address {list}
   + getShippingAddress(): Address {list}
   + addProductReview(review): bool
   + deleteProductReview(review): bool
   + resetPassword():bool   
   ```
 - Product
 - ProductCategory
 - ProductReview
 - Catalog
 - Search
 - CartItem
 - ShoppingCart
 - Order
 - OrderLog
 - Shipment
 - ShipmentLog
 - Payment
 - Notification
 - Enumerations

### Relationships 
	
	
## HLD
    	
