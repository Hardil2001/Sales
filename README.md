This issue will contain the document of the overall implementation plan for the sales.
- [ ] Create Models
    - Create Query Models
        - [ ] Category_FetchCategoryBy
        - [ ] Item_FetchItemBy
        - [ ] Coupon_FetchCouponBy
    - Create CommandModels
        - [ ] SalesTRX
        - [ ] SalesDetailsTRX
        - [ ] SaleRefundTRX
        - [ ] SaleRefundDetailsTRX
- [ ] Create Transactional Services Methods
    - [ ] Add Sales
    - [ ] Return Sales
# UI Experience
This comment outlines the action that can happen on the forms. Actions respond to users interacting with the form, pressing a button, or controlling.

## Sales
- **Initial GET** - Form is blank

> **PRG** *Post Redirect Get*
- **PRG** There will be three navigational buttons named Shopping, cart, and checkout or place an order that will take you to different scenarios according to the customer's preference.
- A Cancel button will be present at all times to cancel the current shopping process the employee is handling.

### In-store Shopping
- **PRG** the item will be added by choosing the item from the categories drop-down list, which will further populate the drop-down list, which will include the items in the category. 
- The stocks will be shown in every category by displaying the items left in the category.  
- There will be an Add button from which the item will be added to the cart according to the added quantity.
#### Business Rules and Form Processing for shopping
- A sale can have only one shopping cart (but as many items in the cart as they want).
- Duplicated items are not allowed to add to the cart.
- The quantity (which must be a whole number greater than zero) must be supplied when clicking Add otherwise, by default, the quantity next to the add will always be 1
- Added products are added to the shopping cart items
- Employees continue shopping after adding an item, but a message is shown confirming that the item has been added to the cart
##### Possible Screen for the In-store shopping
![Untitled Diagram drawio](https://user-images.githubusercontent.com/97215864/203196103-febb9621-12d5-4215-b80f-59dcbe7f537d.png)

### View Cart
- First, the orders will be displayed that are currently on the order 
- There will be navigational buttons for shopping, and the checkout views
- Clicking the remove or update will remove or update the equality of the item in the cart respectively. 
- Clicking the refresh icon will refresh the subtotals of the items in the cart
##### Possible Screen for the In-store shopping
![Untitled Diagram-Page-2 drawio](https://user-images.githubusercontent.com/97215864/203199543-924608aa-c1fe-4045-a828-980e1733d645.png)

### Checkout
- **PRG** the screen of the checkout will show the items selected by the employee, their quantity, and price. 
- **PRG** There will be a coupon box and apply pricing to the order
- There will be different payment methods, 'M', 'C', or 'D', for 'Money', 'Credit' and 'Debit', respectively, according to which one can choose for processing the payment
- the screen will also show the Subtotal, GST, Discount (is a calculated percentage ie: 10%), and Total
- **PRG** the place order button will be used to place an order for the items in which:
    - will Create a sale and sale items based on all the information in the shopping cart.
    - update the QuantityOnOrder for each stock item on the purchase order and if the quantity is negative. throw an exception
    - will display the sales id.
    - Once the order is processed, items cannot be added or edited. 
#### Possible screen for Checkout view
![Untitled Diagram-Page-3 drawio](https://user-images.githubusercontent.com/97215864/203205864-6d714dde-bdc2-4e32-90e5-4aa73a62aa0f.png)

## Returns
#### Lookup sale
- The user will input the sale invoice number and will click the lookup sale
    - Call BLL to lookup the sale invoice with POST and sale id as route parameters
#### Refund Button
- the refund process happens as a complete transaction in which the user can individual or partial items according to the purchase he had made.
- A successful refund results in a SalesRefund record and at least one SalesRefundDetail record being generated.
#### Clear Button
- Clicking the Clear button clears the screen
#### Possible Screen for the Returns

![Untitled Diagram-Page-4 drawio (2)](https://user-images.githubusercontent.com/97215864/203210416-7a6d2100-5238-4d77-b099-e9194f4867fa.png)

# Data Models
## Query Models
### Fetch Category Details(CategoryServices_FetchCategoryBy)
```csharp
public class CategorySelection
{
    public int CategoryID {get; set;}
    public string description {get; set;}
}
```
### Fetch StockItem Details(StockItemServices_FetchStockItemBy)
```csharp
public class ItemSelection
{
    public string Description {get; set;}
    public decimal SellingPrice {get; set;}
    public int QuantityOnHand {get; set;}
}
```
### Fetch Coupon Details(CouponServices_FetchCouponBy)
```csharp
public class CouponSelection
{
    public int CouponID {get; set;}
    public string CouponIDValue {get; set;}
    public datetime StartTime {get; set;}
    public datetime EndDate {get; set;}
    public int CouponDiscount {get; set;}
}
```
## Command Models
### Add Sales
```csharp
public class SalesTRX
{
    public int SalesID {get; set;}
    public datetime SaleDate {get; set;}
    public string PayementType {get; set;}
    public int EmployeeID {get; set;}
    public decimal TotalAmount {get; set;}
    public decimal Subtotal {get; set;}
    public int CouponID {get; set;}
    public int PaymentToken {get; set;}
}
```
### Add SalesDeatils
```csharp
public class SalesDetailsTRX
{
    public int SaleDetailID {get; set;}
    public int SaleID {get; set;}
    public int StockItemID {get; set;}
    public decimal SellingPrice {get; set;}
    public int Quantity {get; set;}
}
```
### Add Refunds
```csharp
public class SaleRefundTRX
{
    public int SaleRefundID {get; set;}
    public datetime SaleRefundDate {get; set;}
    public int SaleID {get; set;}
    public int EmployeeID {get; set;}
    public decimal TaxAmount {get; set;}
    public decimal Subtotal {get; set;}
}
```
### Add SaleRefundDetails
```csharp
public class SaleRefundDetailTRX
{
    public int SaleRefundDetailID {get; set;}
    public int SaleRefundID {get; set;}
    public int StockItemID {get; set;}
    public money Sellingprice {get; set;}
    public int Quantity {get; set;}
}
```


# Business Processing Requirements
## AddSales
### void SalesService_AddSales(int quantity, int stockItemId, decimal sellingPrice)
#### In-Store Shopping view
- check for duplicate items in the cart while adding the items to the cart
    - if it exists, throw an ArgumentException 
    - No
        - Add items to the cart
 - check the quantity entered is the whole number greater than zero
     - yes
         - Add the quantity of an item 
     - No
         - Throw an ArgumentException 
 - Employees continue shopping after adding an item, but a message is shown confirming that the item has been added to the cart
#### View Cart
- Display the added products in the cart
- Employee can change quantities and/or remove items from the cart
   - check the quantity entered is the whole number greater than zero
     - yes
         - Add the quantity of an item 
     - No
         - Throw an ArgumentException 
#### Checkout View
- display Stock currently on the cart
- allow for entering the coupon
    - check if the coupon is valid 
        - No
            - throw an ArgumentException
        - yes
            - Display and add the discount rate to the total 
- check the payment types are 'M,' 'C,' or 'D' for 'Money,' 'Credit,' and 'Debit,' respectively
    - Yes
        - Proceed to the payment
    - No
        - Throw an ArguemntException
- When the order is placed
    - Create a sale and sale items based on all the information in the shopping cart
    - Reduce the stock on hand based on the quantities of the items purchased. 
        - check stock item quantity on hand will not go negative.
           - Yes
               - Throw an ArgumentException
           - No
               - Display the sales id.
- check for any errors
    - yes: throw list of all collected exceptions
    - no: save all work to database
- The order is placed, and the user cannot add or edit the items once the payment is placed
## Returns
- check the entered Sales id is valid and exists
    - No
        - Throw an Exception
    - Yes
        - check if the quantity of the refunded item is the same or not
            - yes
                - Refund the item
            - no
                - Throw an ArgumentException
- check for any errors
    - yes: throw list of all collected exceptions
    - no: save all work to database
    
