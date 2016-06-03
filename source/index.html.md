---
title: Klik & Pay technical support- server to server integration guide

language_tabs:
  -php

includes:
//  - errors

search: true
---

# Introduction to server-to-server mode
<aside class="warning">
Please note that server-to-server mode is subject to Compliance approval and has to be requested and granted for access.<br>

The use of the server-to-server implies that the credit card data passes through your server, therefore we request the merchant to be PCI / DSS Level 3 (or higher) certified<br>
Your PCI-DSS certificate has to be renewed annually and updated document sent back to us. Should the merchant not comply with this requirement payment processing will be disabled and a penalty issued.
</aside>

Please note that only "Direct" and "Deferred" are available through server-to-server mode (subscription and several instalments are not supported)
In order to be able to manage subscriptions and several instalments options, please refer to the "alias" section explained above. 

<aside class="notice">
We strongly advise the merchant account holder to create a user account for the web master that will take care of the Klik & Pay the integration set-up and to limit the access rights. You will be able to 
delete this user account at any time you wish. 
</aside>

# Integration set-up

##Merchant ID and Alias

> ![](/images/DOC_EN_acct.png "")
> ![](/images/DOC_EN_id.png "")

You will find your Merchant ID in your Klik & Pay Back Office under "Account Administration" sub folder "Account Information".

<aside class="notice">
^Merchant ID is unique. <br>
In the case of a multi-currency set-up, each currency will be issued with a unique Merchant ID.You will have to redirect the customer on the specific Merchant ID linked to the invoiced currency.
</aside>

The alias is an imprint of your customers' cards, it is returned after a successful transaction has been processed. You can represent a valid card without requiring the customer to fill in his credit card details again. The alias lets you automate the payment and enable you to set-up a subscription billing system.

##URL and Private Key

> ![](/images/DOC_EN_setup.png "")
> ![](/images/DOC_EN_key.png "") 

<aside class="warning">
Data must be sent in POST to the following URL
</aside>

`https://www.klikandpay.com/paiement/server_server.pl`

When the server-to-server mode is enabled, your own private key account is generated in your Klik & Pay Back Office. Your private key is required to cancel a transaction (full or partial), to confirm and modify a deferred transaction and create a duplicate.

<aside class="notice">
Key can be found under "Account Settings" then "Settings" sub-menu.
</aside>

##Test cards

When in server-to-server only direct payment can be ran in test mode.

Advanced features (cancellation, duplicate, deferred) require the use of a real credit card in order to recover the alias. Transactions will be debited for real.

The following cards are only functional in test mode

Credit card|Number
-----|-----
Visa|4012888888881881
Mastercard|5555555555554444

The expected CVV is 111, enter any date in the future for expiry date.




# Example and variables

## Direct payment and examples

```php
<?php
$url = "https://www.klikandpay.com/paiement/server_server.pl";
$data = array(
    'ID'         => $id,
    'IP'         => $ip,
    'NOM'        => $surname,
    'PRENOM'     => $name,
    'ADRESSE'    => $address,
    'CODEPOSTAL' => $postcode
    'VILLE'      => $vcity,
    'PAYS'       => $country,
    'TEL'        => $tel,
    'EMAIL'      => $email,
    'MONTANT'    => $amount,
    'NUMCARTE'   => $cardnumber,
    'EXPMOIS'    => $expirymonth,
    'EXPANNEE'   => $expiryyear,
    'CVV'        => $cvv,
);
                       
            $ch = curl_init();            
            curl_setopt($ch, CURLOPT_URL, $url);
            curl_setopt($ch, CURLOPT_POST, true);

            curl_setopt($ch, CURLOPT_POSTFIELDS, $data);
           
            curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
           

            $result_curl = curl_exec($ch);

            if(!$result_curl){
                echo 'Curl error: ' . curl_error($ch) . "<br />";
                die();
            }

        ?>

<?php print_r($result_curl); ?>
```
<aside class="warning">
It is mandatory to send the data in POST format to the following URL
</aside>

Mandatory variables | Description
--------- | ------- 
ID|Merchant ID
IP|Real customer IP address
NOM |Customer surname
PRENOM|Customer name
ADRESSE|Customer address
CODEPOSTAL|Customer postcode
VILLE|Customer city
PAYS|ISO format 3166 (EN, FR, DE)
EMAIL|End client email address
TEL|Customer telephone
MONTANT|Transaction amount
NUMCARTE|End client credit card number
EXPMOIS|End client credit card expiry month (expecting 2 digits, i.e: 02 or 11)               
EXPANNEE|End client credit card expiry year (expecting 4 digits, i.e: 2020)
CVV|End client credit card CVV

Optional Variables | Description
--------- | ------- 
STATE|Credit card holder state
SOCIETE|End client company
DETAIL|Basket function- described below
FAX|End client fax number
DIFFERE|If = 1 then the transaction will be deferred, you will then have to cancel or validate it depending on your account settings.


## Cancellation script (full or partial)

This script enables you to cancel or partially refund a processed transaction. As well as to cancel a pending deferred transaction awaiting for validation.

`Script URL: https://www.klikandpay.com/paiement/cancel_transaction.pl`

Mandatory Variables | Description
--------- | ------- 
ID|Merchant ID
TX|Transaction number recovered from the acceptance of the transaction
MONTANT|Amount should be less than or equal to the transaction amount
PRIVATE_KEY|Private key to be recovered in the merchant's BO under the set-up section (only available with the server-to-server option enabled)

## Deferred transaction validation script:

This script enables you to validate the desired amount on a deferred transaction.

`Script URL: https://www.klikandpay.com/paiement/validedifferee.pl`

Mandatory Variables | Description
--------- | ------- 
ID|Merchant ID
TX|Transaction number recovered from the acceptance of the transaction
MONTANT|Amount should be less than or equal to the transaction amount
PRIVATE_KEY|Private key to be recovered in the merchant's BO under the set-up section (only available with the server-to-server option enabled))

## Duplicate transaction script: 

This script will allow you to duplicate a transaction to enable you to re-bill the same card through the alias, this solution allows you for example the management of subscriptions.

`Script URL : https://www.klikandpay.com/paiement/duplicate.pl`

Mandatory Variables | Description
--------- | ------- 
ID|Merchant ID
IP|Real customer IP address used for the first transaction
MONTANT|Transaction amount
ALIAS|Alias recovered during the first transaction
PRIVATE_KEY|Private key to be recovered in the merchant's BO under the set-up section (only available with the server-to-server option enabled)
DIFFERE|If = 1 then the transaction will be deferred, you will then have to cancel or validate it depending on your account settings.

