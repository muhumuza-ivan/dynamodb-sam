# Verification: CRUD from the AWS Console

The table is `dynamodb-sam-<env>-orders`, for example `dynamodb-sam-dev-orders`.

## 1. Check the table configuration

Go to **DynamoDB → Tables → dynamodb-sam-dev-orders**.

- **Overview → General information**
  - Partition key: `OrderId (String)`
  - Capacity mode: **On-demand**
  - Table class: **DynamoDB Standard-IA**
- **Indexes** tab: `CustomerOrdersIndex` (CustomerId / OrderDate) and `OrderStatusIndex` (OrderStatus / OrderDate), both *Active*.

## 2. Create an item

1. Click **Explore table items → Create item**.
2. Switch to **JSON view**, turn off *View DynamoDB JSON*, and paste:
   ```json
   {
     "OrderId": "ORD-2001",
     "CustomerId": "CUST-003",
     "OrderStatus": "PENDING",
     "OrderDate": "2026-09-26T09:00:00Z",
     "TotalAmount": 59.9
   }
   ```
3. Click **Create item**.

## 3. Read / query

- **Get by primary key:** use *Scan or query items → Query*, table `dynamodb-sam-dev-orders`, `OrderId = ORD-2001`.
- **Query GSI 1:** select index `CustomerOrdersIndex`, `CustomerId = CUST-003`. You can also add a sort key condition, e.g. `OrderDate begins with 2026-09`.
- **Query GSI 2:** select index `OrderStatusIndex`, `OrderStatus = PENDING`.

## 4. Update

Select the item, then **Actions → Edit item**. Change `OrderStatus` to `SHIPPED` and click **Save**. Re-run the `OrderStatusIndex` query for `SHIPPED` to confirm the item has moved.

## 5. Delete

Select the item, then **Actions → Delete items → Delete**.

## CLI equivalents (optional)

```bash
T=dynamodb-sam-dev-orders
aws dynamodb batch-write-item --request-items file://events/sample-orders.json
aws dynamodb get-item --table-name $T --key '{"OrderId":{"S":"ORD-1001"}}'
aws dynamodb query --table-name $T --index-name CustomerOrdersIndex \
  --key-condition-expression "CustomerId = :c" \
  --expression-attribute-values '{":c":{"S":"CUST-001"}}' --no-scan-index-forward
aws dynamodb query --table-name $T --index-name OrderStatusIndex \
  --key-condition-expression "OrderStatus = :s" \
  --expression-attribute-values '{":s":{"S":"PENDING"}}'
aws dynamodb update-item --table-name $T --key '{"OrderId":{"S":"ORD-1001"}}' \
  --update-expression "SET OrderStatus = :s" --expression-attribute-values '{":s":{"S":"SHIPPED"}}'
aws dynamodb delete-item --table-name $T --key '{"OrderId":{"S":"ORD-1001"}}'
```

`events/sample-orders.json` targets the dev table. Replace the table name with `dynamodb-sam-prod-orders` to load it into prod.
