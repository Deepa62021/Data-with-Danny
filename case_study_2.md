# Case Study #2

![image](https://user-images.githubusercontent.com/74512335/131251748-01b76a8a-4e4b-415c-83e0-e27348b0ffba.png)

## Availbale Data

### Entity Relationship Diagram

![image](https://user-images.githubusercontent.com/74512335/131252005-8a5091d2-527b-4395-8334-a45c0331d022.png)

### Table 1: runners 

The **runners** table shows the **registration_date** for each new runner

![image](https://user-images.githubusercontent.com/74512335/131252153-17bfd9ab-827f-427f-bb48-00a2fb72199e.png)

### Table 2: customer_orders

Customer pizza orders are captured in the **customer_orders** table with 1 row for each individual pizza that is part of the order.

The **pizza_id** relates to the type of pizza which was ordered whilst the **exclusions** are the **ingredient_id** values which should be removed from the pizza and the **extras** are the **ingredient_id** values which need to be added to the pizza.

Note that customers can order multiple pizzas in a single order with varying **exclusions** and **extras** values even if the pizza is the same type!

The **exclusions** and **extras** columns will need to be cleaned up before using them in your queries.

![image](https://user-images.githubusercontent.com/74512335/131252232-fac52941-df94-418b-9f06-68b7bec50e92.png)
