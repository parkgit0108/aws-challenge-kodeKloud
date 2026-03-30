# Building and Managing NoSQL Databases with AWS DynamoDB

---

<aside>

**Purpose:** Create a DynamoDB table, insert sample tasks, then verify item status values using the DynamoDB console.

</aside>

---

### Procedure

1. Create the DynamoDB table (Console):
    1. Open the DynamoDB console and choose Create table.
    2. For Table name, enter `nautilus-tasks`.
    3. For Partition key, enter `taskId` and set the type to String.
    4. Choose Create table.
    5. Wait until the table Status shows Active.
2. Add items to the table (Console):
    1. Open the `nautilus-tasks` table.
    2. Choose Actions → Create item.
    3. Insert the required tasks (Task 1 and Task 2) as provided in the lab instructions.
3. Verify items (Console):
    1. In the DynamoDB navigation pane, choose Explore items.
    2. Select the `nautilus-tasks` table.
    3. Confirm:
        - Task 1 has `status` = `completed`.
        - Task 2 has `status` = `in-progress`.

### Quick check

- **Expected result:** The table exists and is Active, and the two inserted items show the expected `status` values in Explore items.