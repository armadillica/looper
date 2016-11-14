# Looper - Subscription Management System

Looper is a flexible subscription management system, designed after real-life scenarios.

## Features

* Multple currencies
* Location detection for currency selection
* Multiple payment methods (via gateways such as Braintree, Stripe, etc)
* Recurring and manual (advance) payments
* Up and downgrades (subscription add-ons)
* VAT support
* Invoicing
* Reporting tools (sales, customers) 

## Design idea

Give freedom to the customer to pick the desired payment solutions. Different countries and
cultures have different expectations when it comes to subscription systems.

Looper establishes the concept of *balance*, which is the financial status of a customer.
The balance can be affected by debit or credit transactions, which can be performed by the
cusomer or by the system itself. 

## Model schemas

Base functionality and collections are inherited from the Pillar Framework. For example: 
Users, Tokens, Files are valid Pillar Models.

### Transactions
A collection of every operation that affects a customer's balance. The transaction
lifetime follows this schema:

Creation (pending)
Transaction (completed or failed)

A transaction can not be refunded, only an order can. This will create another
transaction in the customer's record.

    transactions_schema = {
        'amount': {
            'type': 'integer', # Must be more than 0
            'required': True,
        },
        'currency': {
            'type': 'string', # Override to define more currencies
            'allowed': [
                'EUR',
                'USD',
                'BTC', # We live in the future and want to support BitCoin!
            ],
        },
        'description': {
            'type': 'string', # Subscription payment, order refund, etc.
        }
        'status': {
            'type': 'string',
            'required': True,
            'allowed': [
                'pending',
                'completed',
                'failed',
            ],
        },
        'user': {
            'type': 'objectid',
            'data_relation': {
                'resource': 'users',
                'field': '_id',
                'embeddable': True
            },
        },
        'direction': {
            'type': 'string',
            'required': True,
            'allowed': [
                'credit', # Money is aded to the balance (usually by the customer)
                'debit', # Money is removed from the balance
            ],
        },
        'backend': {
            'type': 'string',
            'required': True,
            'allowed': [
                'braintree',
                'stripe',
                'local'
            ]
        },
        'method': {
            'type': 'string',
            'required': True,
            'allowed': [
                'cc', # Credit Card
                'wt', # Wire Trasfer
                'pp', # PayPal
                'bc', # BitCoin
            ]
        },
        'properties': {
            'type': 'dict',
            'valid_properties': True,
        }
    }

### Orders
The customer places an order when he manually purchases an extra month of membership, 
or when he adds some credit to his balance.
The system can create an order on behalf of the customer when charging a subscription 
that is set to auto-renew.

    orders_schema = {
        'amount': {
            'type': 'integer', # Must be more than 0
            'required': True,
        },
        'currency': {
            'type': 'string', # Override to define more currencies
            'allowed': [
                'EUR',
                'USD',
                'BTC', # We live in the future and want to support BitCoin!
            ],
        },
        'description': {
            'type': 'string', # Subscription payment, order refund, etc.
        }
        'status': {
            'type': 'string',
            'required': True,
            'allowed': [
                'pending',
                'completed',
                'failed',
                'refunded', # If order is refunded, referenced transactions sum should be 0
            ],
        },
        'user': {
            'type': 'objectid',
            'data_relation': {
                'resource': 'users',
                'field': '_id',
                'embeddable': True
            },
        },
        'transactions': {
            # References payments and refunds for the order
            'type': 'list',
            'schema': {
                'type': 'objectid'
            }
        },
        'backend': {
            'type': 'string',
            'required': True,
            'allowed': [
                'braintree',
                'stripe',
                'local'
            ]
        },
        'method': {
            'type': 'string',
            'required': True,
            'allowed': [
                'cc', # Credit Card
                'wt', # Wire Trasfer
                'pp', # PayPal
                'bc', # BitCoin
            ]
        },
        'properties': {
            # Depending on the backend and payment method, here we store custom props,
            # like API order id
            'type': 'dict',
            'valid_properties': True,
        }
    }
