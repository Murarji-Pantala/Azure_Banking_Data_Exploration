# Azure_Banking_Data_Exploration
Data Exploration with Azure SQL Database – Customer, Account, and Loan Feeds

Project Tasks:

1. Setting Up Azure SQL Database
   - Step 1.1: Create an Azure SQL Database in the Azure portal.
     -Install Azure Data Studio 
   - Step 1.2: Name the database `CustomerAccountLoanDB`.

2. Data Organisation
    - Step 2.1: Create tables for the provided feeds:
     - Customer Feed:
     - Account Feed:
     - Transaction Feed:
     - Loan Feed: 
     - Loan Payment Feed:

3. Data Insertion :
    - Download SQL Server Import - This helps in importing the local file to Azure Data Studio and connecting to the SQL Database.
    Using Import Wizard step by step process data is imported and primary key is allocated .
    Run SQL Query to assign Foreign key constraint 

    
--SQL-- 

ALTER TABLE accounts
ADD CONSTRAINT fk_accounts
FOREIGN KEY (customer_id) REFERENCES customers(customer_id);

ALTER TABLE loans
ADD CONSTRAINT fk_loans
FOREIGN KEY (customer_id) REFERENCES customers(customer_id);

ALTER TABLE loan_payments
ADD CONSTRAINT fk_loan_payments
FOREIGN KEY (loan_id) REFERENCES loans(loan_id);

ALTER TABLE transactions
ADD CONSTRAINT fk_transactions
FOREIGN KEY (account_id) REFERENCES accounts(account_id);

--4.1--Write query to retrieve all customer information:

SELECT 
    c.customer_id, 
    c.first_name, 
    c.last_name, 
    a.account_id, 
    a.balance, 
    a.account_type, 
    l.loan_amount, 
    lp.payment_date
FROM 
    customers c
JOIN 
    accounts a ON c.customer_id = a.customer_id
JOIN 
    loans l ON c.customer_id = l.customer_id
JOIN 
    loan_payments lp ON l.loan_id = lp.loan_id;
    

--4.2--

SELECT 
    c.first_name,
    c.last_name,
    a.account_id, 
    a.customer_id, 
    a.account_type, 
    a.balance
FROM 
    accounts a 
JOIN 
    customers c ON a.customer_id = c.customer_id 
WHERE 
    a.customer_id = 12; 

--4.3--Find the customer name and account balance for each account

SELECT 
    c.first_name,
    c.last_name,
    a.balance
FROM 
    customers c 
JOIN 
accounts a on c.customer_id=a.customer_id;   

--4.4--  Analyze customer loan balances:

SELECT 
    c.customer_id, 
    c.first_name, 
    c.last_name, 
    SUM(l.loan_amount) AS total_loan_amount
FROM 
    customers c
JOIN 
    loans l ON c.customer_id = l.customer_id
GROUP BY 
    c.customer_id, 
    c.first_name, 
    c.last_name;


--4.5-- List all customers who have made a transaction in the 2024-03
Select 
    c.first_name,
    c.last_name,
    t.transaction_date,
    t.transaction_amount
FROM 
    customers c
JOIN 
    accounts a on (c.customer_id=a.customer_id) 
JOIN    
    transactions t on (a.account_id=t.transaction_id)
WHERE 
    t.transaction_date >= '2024-03-01' AND t.transaction_date < '2024-04-01';


--5.1-- Calculate the total balance across all accounts for each customer:

SELECT 
    c.customer_id, 
    c.first_name, 
    c.last_name, 
    SUM(a.balance) as total_balance
FROM 
    customers c
JOIN 
    accounts a ON c.customer_id = a.customer_id
GROUP BY 
    c.customer_id, 
    c.first_name, 
    c.last_name;


--5.2-- Calculate the average loan amount for each loan term:

SELECT 
    loan_term, 
    AVG(loan_amount) AS average_loan_amount
FROM 
    loans
GROUP BY 
    loan_term;


--5.3--Find the total loan amount and interest across all loans:

SELECT 
    c.customer_id, 
    c.first_name, 
    c.last_name, 
    SUM(l.loan_amount) AS total_loan_amount, 
    SUM(l.interest_rate ) AS total_interest
FROM 
    customers c
JOIN 
    loans l ON c.customer_id = l.customer_id
GROUP BY 
    c.customer_id, 
    c.first_name, 
    c.last_name;


--5.4-- Find the most frequent transaction type:

SELECT 
    transaction_type, 
    COUNT(*) AS frequency 
FROM 
    transactions 
GROUP BY 
    transaction_type;


--5.5-- Analyze transactions by account and transaction type:

SELECT 
    a.account_id,
    t.transaction_type,
    COUNT(*) AS transaction_count,
    SUM(t.transaction_amount) AS total_amount
FROM 
    transactions t
JOIN 
    accounts a ON t.account_id = a.account_id
GROUP BY 
    a.account_id, 
    t.transaction_type
ORDER BY 
    a.account_id, 
    t.transaction_type;


--6.1-- Create a view of active loans with payments greater than $1000:

CREATE VIEW active_loans_with_high_payments AS
SELECT 
    l.loan_id, 
    l.customer_id, 
    l.loan_amount, 
    l.interest_rate, 
    l.loan_term, 
    lp.payment_id, 
    lp.payment_date, 
    lp.payment_amount
FROM 
    loans l
JOIN 
    loan_payments lp ON l.loan_id = lp.loan_id
WHERE 
    lp.payment_amount > 1000;


--6.2--

CREATE INDEX idx_transaction_date
ON transactions (transaction_date);


