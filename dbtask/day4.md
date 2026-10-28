# Day 4: Constraints and Views

## Objectives

- define business rules with `CHECK`, primary-key, and foreign-key constraints;
- create simple views;
- insert through an updatable view;
- filter a view by city.

## 1. Client and salesperson master tables

```sql
CREATE TABLE client_master (
    client_no VARCHAR2(6)
        CONSTRAINT client_master_pk PRIMARY KEY
        CONSTRAINT client_no_ck CHECK (client_no LIKE 'C%'),
    name VARCHAR2(20)
        CONSTRAINT client_name_ck CHECK (name = UPPER(name)),
    city VARCHAR2(15)
        CONSTRAINT client_city_ck
        CHECK (city IN ('MUMBAI', 'DELHI', 'MADRAS', 'CALCUTTA')),
    state   VARCHAR2(15),
    bal_due NUMBER(10,2)
        CONSTRAINT client_balance_ck CHECK (bal_due >= 0)
);

CREATE TABLE salesman_master (
    salesman_no VARCHAR2(6)
        CONSTRAINT salesman_master_pk PRIMARY KEY
        CONSTRAINT salesman_no_ck CHECK (salesman_no LIKE 'S%'),
    name VARCHAR2(20)
        CONSTRAINT salesman_name_ck CHECK (name = UPPER(name)),
    city VARCHAR2(15)
        CONSTRAINT salesman_city_ck
        CHECK (city IN ('MUMBAI', 'DELHI', 'MADRAS', 'CALCUTTA')),
    state   VARCHAR2(15),
    bal_due NUMBER(10,2)
        CONSTRAINT salesman_balance_ck CHECK (bal_due >= 0)
);
```

The city names reproduce the 2021 source. In a new application, consider current official names such as Chennai and Kolkata and use a separate city reference table instead of a hard-coded list.

## 2. Sales-order tables

The raw session created `SALES_ORDER_DETAILS` first and made `SALES_ORDER` reference one detail row. A more usual header/detail model makes each detail reference its order:

```sql
CREATE TABLE sales_order (
    order_no  VARCHAR2(6) CONSTRAINT sales_order_pk PRIMARY KEY,
    client_no VARCHAR2(6) NOT NULL,
    order_date DATE DEFAULT SYSDATE NOT NULL,
    CONSTRAINT sales_order_client_fk
        FOREIGN KEY (client_no) REFERENCES client_master (client_no)
);

CREATE TABLE sales_order_detail (
    order_no     VARCHAR2(6),
    product_no   VARCHAR2(6),
    qty_ordered  NUMBER(8) CHECK (qty_ordered > 0),
    qty_dispatched NUMBER(8) CHECK (qty_dispatched >= 0),
    product_rate NUMBER(10,2) CHECK (product_rate >= 0),
    CONSTRAINT sales_order_detail_pk PRIMARY KEY (order_no, product_no),
    CONSTRAINT sales_order_detail_order_fk
        FOREIGN KEY (order_no) REFERENCES sales_order (order_no),
    CONSTRAINT sales_order_detail_qty_ck
        CHECK (qty_dispatched <= qty_ordered)
);
```

This preserves the original fields while representing the common one-order-to-many-details relationship.

## 3. Create restricted-column views

```sql
CREATE VIEW client AS
SELECT client_no, name, city
FROM client_master;

CREATE VIEW salesman AS
SELECT salesman_no, name, city
FROM salesman_master;
```

These are simple single-table views and are updatable for the exposed columns.

## 4. Insert the original data through the views

```sql
INSERT INTO client VALUES ('C00001', 'ASHOK MEHRA',   'MUMBAI');
INSERT INTO client VALUES ('C00002', 'VISHAL PARIKH', 'DELHI');
INSERT INTO client VALUES ('C00003', 'AJEY MEHRA',    'MUMBAI');
INSERT INTO client VALUES ('C00004', 'ROHIT ROY',     'CALCUTTA');
INSERT INTO client VALUES ('C00005', 'NALINI DEEWAN', 'MUMBAI');
INSERT INTO client VALUES ('C00006', 'PREM IYER',     'DELHI');
INSERT INTO client VALUES ('C00007', 'RAHUL DESAI',   'MADRAS');

INSERT INTO salesman VALUES ('S00001', 'MANISH PATEL', 'MUMBAI');
INSERT INTO salesman VALUES ('S00002', 'KIRAN DIXIT',  'DELHI');
INSERT INTO salesman VALUES ('S00003', 'NITESH KHANNA','MUMBAI');
INSERT INTO salesman VALUES ('S00004', 'MAHESH PATEL', 'CALCUTTA');

COMMIT;
```

The omitted `STATE` and `BAL_DUE` base-table columns become null because they allow nulls and have no defaults.

## 5. Compare views and base tables

```sql
SELECT * FROM client;
SELECT * FROM client_master;
SELECT * FROM salesman;
SELECT * FROM salesman_master;
```

The views expose three columns, while the master tables also expose state and balance.

## 6. Filter by city

```sql
SELECT * FROM client WHERE city = 'MUMBAI';
SELECT * FROM salesman WHERE city = 'MUMBAI';
```

The client query returns Ashok Mehra, Ajey Mehra, and Nalini Deewan. The salesperson query returns Manish Patel and Nitesh Khanna.

## Improvements to try

- Add `NOT NULL` to columns that the business requires.
- Replace city checks with a city lookup table and foreign key.
- Give `BAL_DUE` a default of zero.
- Create a read-only public view with `WITH READ ONLY`.
- Grant access to the view instead of the entire master table.

Continue to [Day 5](day5.md).

> Original SQL\*Plus transcript: [day4.txt](day4.txt)
