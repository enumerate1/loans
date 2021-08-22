# loans

## DB Schemas:

### users:
```
user_id pgtype.Text
name pgtype.Text
password pgtype.Text
created_at pgtype.Timestamptz
updated_at pgtype.Timestamptz
deleted_at pgtype.Timestamptz
loan_ids pgtype.TextArray (maybe don't need)
```

### loans
```
loan_id pgtype.Text
repayment_frequency pgtype.Text
interest_rate pgtype.Float4
arrangement_fee pgtype.Float4
term pgtype.Int4 (unit maybe use monthly for easy)
created_at pgtype.Timestamptz
updated_at pgtype.Timestamptz
deleted_at pgtype.Timestamptz
repayment_ids pgtype.TextArray
status pgtype.Text ["LOAN_STATUS_NONE", "LOAN_STATUS_IN_PROGRESS","LOAN_STATUS_OVERDUE","LOAN_STATUS_COMPLETED"]
amount_need_repayment pgtype.Float4
amount_repayment pgtype.Float4 (default 0)
current_month_repayment pgtype.Int4 (default 0)


```

### repayments
repayment_id pgtype.Text
loan_id pgtype.Text
created_at pgtype.Timestamptz
updated_at pgtype.Timestamptz
deleted_at pgtype.Timestamptz
amount pgtype.Float4


## API design:

/user [POST]
```
...
```
some logic have to handle:</br>
+the number_phone is unit (if need)</br>
+password have to hash: should use bcrypt


/to-loan [POST]
```
user_id
repayment_frequency
interest_rate
arrangement_fee
term
```
some logic have to handle:
first status: LOAN_STATUS_NONE
using the attribute we have to calculate the field named `amount_need_repayment`



/repayment [POST]
```
user_id
loan_id
amount
```
be attention: `The app logic should figure out and not allow obvious errors. For example a user cannot
make a repayment for a loan that’s already been repaid.`

some logic have to handle here:

we have 2 case here: `in_process` and `completed`
we have to use transaction, and block row to handle concurrency here, if not -> we fail
step 1: get the row on `loans` table and block (hint: FOR UPDATE)
step 2: check the amount of the user can repayment all or not, if not -> go to step 3, if yes (like step 3, 4) but update completed_at and status become -> LOAN_STATUS_COMPLETED </br>
step 3: create record on `repayment` (hint: RETURNING repayment_id) </br>
step 4: UPDATE multi values to the `loans` table:
+ repayment_ids -> append 
+ current_month_repayment (plus 1)
+ amount_repayment -> plus with `amount` on tale `repayments`
+ status -> LOAN_STATUS_IN_PROGRESS