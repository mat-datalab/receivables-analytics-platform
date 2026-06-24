# Entities:

## CUSTOMERS
    Entity: Customers
    Grain: one row per customer
    Primary key: customer_id
    Foreign keys: collector_id
    Purpose: To identify the customers
    Example fields: company_name, country, segment, risk_tier, payment_terms
    Relationships: To Invoices, Collectors
    Data quality checks:
        - customer_id cannot be null
        - customer_id must be unique
        - collector_id must reference an existing collector
        - company_name cannot be null
        - country should use a valid country code or predefined country list
        - segment should be one of predefined values
        - risk_tier should be one of predefined values
        - payment_terms must be greater than or equal to 0

## INVOICES
    Entity: Invoices
    Grain: one row per issued invoice
    Primary key: invoice_id
    Foreign keys: customer_id
    Purpose: To check for invoices info
    Example fields: customer_id, invoice_id, issue_date, due_date, invoice_status, invoice_amount, paid_amount, outstanding_amount, currency
    Relationships: To Customers, Payments, Disputes
    Data quality checks:
        - invoice_id cannot be null
        - invoice_id must be unique
        - customer_id must reference an existing customer
        - invoice_amount must be greater than 0
        - paid_amount must be greater than or equal to 0
        - outstanding_amount must be greater than or equal to 0
        - paid_amount cannot be greater than invoice_amount
        - due_date cannot be earlier than issue_date
        - invoice_status should be one of predefined values
        - currency should be one of supported currency codes

## PAYMENTS
    Entity: Payments
    Grain: one row per payment transaction assigned to an invoice
    Primary key: payment_id
    Foreign keys: invoice_id
    Purpose: To check for payment info/status
    Example fields: payment_date, payment_amount, original_currency, amount_in_base_currency
    Relationships: To Invoices
    Data quality checks:
        - payment_id cannot be null
        - payment_id must be unique
        - invoice_id must reference an existing invoice
        - payment_amount must be greater than 0
        - payment_date cannot be earlier than invoice issue_date
        - currency should be one of supported currency codes
        - payment amount should not exceed invoice outstanding amount unless overpayments are explicitly allowed
        - duplicate payments should be detected by invoice_id, payment_date, amount, and currency
    

## DISPUTES
    Entity: Disputes
    Grain: one row per dispute case
    Primary key: dispute_id
    Foreign keys: invoice_id, idirectly to Customers
    Purpose: To identify which invoices are disputed and what is the issue
    Example fields: invoice_id, reason_code, created_date, closed_date, status, disputed_amount
    Relationships: To Invoices, Customers
    Data quality checks:
        - dispute_id cannot be null
        - dispute_id must be unique
        - invoice_id must reference an existing invoice
        - disputed_amount must be greater than 0
        - disputed_amount cannot be greater than invoice_amount
        - status should be one of predefined values
        - reason_code should be one of predefined values
        - created_date cannot be earlier than invoice issue_date
        - closed_date should be null for open disputes
        - closed_date cannot be earlier than created_date


## PROMISES_TO_PAY
    Entity: Promises_to_pay
    Grain: one row per promise made by a customer for an invoice
    Primary key: promise_id
    Foreign keys: invoice_id, customer_id optional/denormilized
    Purpose: To identify if customers promised to pay invoice and when they will pay
    Example fields: created_date, promised_payment_date, promised_amount, status (open, kept, broken)
    Relationships: To Invoices, Customers
    Data quality checks:
        - promise_id cannot be null
        - promise_id must be unique
        - invoice_id must reference an existing invoice
        - promised_amount must be greater than 0
        - promised_payment_date cannot be earlier than created_date
        - status should be one of predefined values
        - kept promises should have a related payment
        - broken promises should have promised_payment_date earlier than the processing date and no matching payment

## COLLECTORS
    Entity: Collectors
    Grain: one row per collector
    Primary key: collector_id
    Foreign keys: -
    Purpose: To verify which collector is responsible for which customer
    Example fields: collector_id, name, surname, email, phone_number, is_active, team
    Relationships: To Customers
    Data quality checks:
        - collector_id cannot be null
        - collector_id must be unique
        - email cannot be null
        - email should be unique
        - email should have a valid format
        - is_active should be true or false
        - inactive collectors should not be assigned to active customers
        - team should be one of predefined values

## EXCHANGE_RATES
    Entity: Exchange_rate
    Grain: one row per currency pair per date
    Primary key: base_currency + target_currency + rate_date
    Foreign keys: -
    Purpose: To use it to convert rates for payments
    Example fields: base_currency, target_currency, rate_date, exchange_rate
    Relationships: To Payments, Invoices
    Data quality checks:
        - base_currency cannot be null
        - target_currency cannot be null
        - rate_date cannot be null
        - exchange_rate must be greater than 0
        - combination of base_currency, target_currency, and rate_date must be unique
        - base_currency and target_currency should use supported currency codes
        - rate_date cannot be in the future
        - missing exchange rates should be detected for currencies used in invoices or payments

## Core Relationships
1. One customer can have many invoices
2. One invoice can have many payments
3. One invoice can have zero or many disputes
4. One customer can have many promises to pay
5. One collector can manage many customers
6. Exchange rates are joined by currency and date