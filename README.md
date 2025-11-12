# Cheatsheet-BestPractices-Fiuu_API  

## Objective  
This is a cheatsheet for those developers who is frustrated by the lengthy official docs and having a hard time on deciding which option is the best approach to fit their integration requirements. This README.md file will be updated from time to time when we discovered new pain point faced by developers during the integration process. We welcome every suggestion or enquiry from developers in the form of posting new issue to this repository.  

### Tips #1. Various integration method provided by Fiuu  
Fiuu provides few of the integration methods to suit every business needs.  
#### i. Hosted Payment Page  
A hosted solution where customer will redirect from merchant's website to Fiuu's hosted payment page (using HTML form submit method), channel selection will be displayed on this page before redirect to each online banking / ewallet page to proceed with the payment authentication and authorization (via OTP or secureTAC). After that a success / failure page will be displayed to customer before redirect back to merchant's website. This integration type is the most straight-forward and require least effort among other integration types. You may find Fiuu's Hosted Payment Page API document here (https://github.com/FiuuPayment/Documentation-Fiuu_API_Spec/blob/main/%5Bofficial%20API%5D%20Fiuu%20API%20Spec%20for%20Merchant%20(v13.91).pdf).   
#### ii. Seamless Integration
Channel selection will be displayed in the merchant's website which require more integration effort from the developers comparing to Hosted Payment Page. For card channel the card number will still be input through Fiuu's page hence no PCI-DSS compliance is required from the merchant. You may find Fiuu's Seamless Integration API document here (https://github.com/FiuuPayment/Integration-Fiuu_JavaScript_Seamless_Integration).   
#### iii. Inpage Checkout
Similar to seamless integration but only support card channel, and the difference comparing to seamless integration is the card number input page will be hosted via iFrame on merchant's website instead of Fiuu's page. You may find Fiuu's Inpage checkout API document here (https://github.com/FiuuPayment/Integration-Fiuu_Inpage_Checkout).   
#### iv. Mobile XDK  
Best for mobile in-app purchase in the form of installable plugins. You may search for the plugin suited to your specific development platform (for e.g. Android Studio, Xcode, Flutter etc.) under this page (https://github.com/FiuuPayment/).  
#### v. Direct Server Integration
No UI will be provided by Fiuu, channel selection and card number input will all be displayed on merchant's website, hence requires merchant to be PCI-DSS compliant. Please request for Fiuu's Direct Server API document (NOT published anywhere publicly) by dropping an email to support@fiuu.com.  
#### vi. Recurring API  
No UI will be provided by Fiuu, but it supports recurring payment by any amount at any time based on merchant's own scheduler (for e.g. cron job). Before sending a request to Fiuu's recurring API, it is required for merchant to obtain a token from a one-time payment (can be card or ewallet transaction). The one-time payment could be made with a minimum amount with our hosted payment page or seamless integration, once the one-time payment has been successful, expect a set of token (in the form of alpha-numeric string) will be returned under extraP parameters via notification URL. Also please note that merchant will need to explicitly request for tokenization and recurring feature to be enabled for your merchant ID by sending an email to support@fiuu.com. You can find Fiuu's recurring API document here (https://github.com/FiuuPayment/Documentation-Fiuu_API_Spec/blob/main/Fiuu%20Recurring%20API%20v7.1.4.pdf).   

#### Comparison Chart
<p align="center">
<img src="https://user-images.githubusercontent.com/19460508/180356625-1e42fc30-9d15-4da6-af04-aab8fd34cf9c.png" />
</p>
  
   
### Tips #2. The 3 endpoints to update payment status on merchant site  
When a payment is done by customer, how do we update the transaction status to the merchant? The 3 endpoints, i.e. Return URL, Notification URL, Callback URL serves as a mechanism to update merchant transaction status. You may find Fiuu's the API document for the implementation of the 3 endpoints here (https://github.com/FiuuPayment/Documentation-Fiuu_API_Spec/blob/main/%5Bofficial%20API%5D%20Fiuu%20API%20Spec%20for%20Merchant%20(v13.90).pdf). 
#### i. Return URL
Realtime web browser or frontend redirection endpoint for hosted page, seamless integration, and shopping cart module. Impact user's UI/UX or journey in the whole payment process. DO NOT RELY ON THIS ENDPOINT TO UPDATE YOUR TRANSACTION STATUS, DUE TO IT BEING SUSCEPTIBLE TO MITM (MAN-IN-THE-MIDDLE) ATTACK, REFER TO NOTIFCATION URL FOR A MORE ROBUST STATUS UPDATE METHODOLOGY.  
#### ii. Notification URL
Realtime server-to-server or backend endpoint for all kind of integrations and payment channels. Will be triggerred once Fiuu received the payment status from respective online banking / ewallet channels. MOST ROBUST AND RECOMMENDED WAY TO UPDATE TRANSACTION STATUS TO YOUR SYSTEM.   
#### iii. Callback URL  
Defer update or callback endpoint on non-realtime payment such as Fiuu Cash.  
  
  
### Tips #3. To enable IPN to avoid missing the webhook   
IPN (Instant Payment Notification) is a mechanism whereby if enabled, merchant will be required to ACK (acknowledge) on each of the received webhook, i.e. notification and callback URL. In case merchant failed to ACK, we will assume that the previous webhook has failed and we will retry sending the webhook with an interval of 15 minutes until a maximum of 4 retries to the merchant Callback URL endpoint.  
  
  
### Tips #4. Get channel subscribed by using Channel Status API
For those merchant who are using integration type such as Seamless or Direct Server where the channel selection is done on the merchant site, developers are advised to get the channel subscribed by using Channel Status API.  

<p align="center">
<img src="https://user-images.githubusercontent.com/19460508/180359757-b2a1dd13-d42c-4fd0-92e6-c12945834680.png" />
</p>
  
  
### Tips #5. For requery the difference between Direct / Indirect Status Requery  
Requery is the API used to check for each transaction status by using Transaction ID or Order ID. But we have 2 different types of requery, i.e. Direct Status Requery and Indirect Status Requery.  
#### i. Direct Status Requery
Fiuu will forward the status enquiry by merchant to the respective online banking / ewallet channel, which help the merchant to get realtime transaction status as aligned with the payment channel itself.  
#### ii. Indirect Status Requery
Fiuu will return the transaction status based on our own status within our system.  
  
  
### Tips #6. For void use Reversal API, for refund use Refund API
We have 2 types of Refund, i.e. Reversal and Full / Partial Refund
#### i. Reversal
Fiuu will forward the refund request by merchant to the respective online banking / ewallet channel, and proceed to void the transaction, but this is only allowed on the same day as the transaction itself.  
#### ii. Full / Partial Refund  
Fiuu will process the refund request by merchant ourselves and allow up to 180 days after the transaction date. Either the refund request is a full refund or partial refund will depend on the amount parameter passed by the merchants.   

### Tips #7. Guest Checkout to disable save card feature  
Fiuu provides save card features by requiring customer's basic info like billing name, mobile number and email. With that being said, this feature is only eligible for a registered user of the merchant's website. But there are also some checkout where merchant does not hold the customer's basic contact info, which is also known as a guest checkout. To disable save card feature for a guest user, merchant can pass the parameter `bill_name=guest&bill_email=&bill_mobile=` (case insensitive) to our payment page.   
<p align="center">
<img src="https://user-images.githubusercontent.com/19460508/182115799-36bb1835-d634-49ae-9ff2-bc5121b452b4.png" />
</p>

  
### Tips #8. x-www-form-urlencoded instead of JSON  
Most of our API is accepting POST request in the format of x-www-form-urlencoded instead of JSON, please make sure you are using the correct format when submitting the request. Meanwhile, if you are submitting the request with both GET (query string) and POST (x-www-form-urlencoded) parameters, please bear in mind that by default the POST parameters will always overwrite the GET parameters. For e.g. in PHP if the script is written as $_REQUEST['param1'], this will always prioritize $_POST['param1'] rather than $_GET['param1'].  

<p align="center">
<img src="https://github.com/RazerMS/Cheatsheet-BestPractices-RazerMS_API/assets/19460508/a8aab2f1-b16b-4852-a7e6-dfff2d9231ad" />
</p>

  
### Tips #9. Domain name whitelisting (domain registration) required when using seamless integration  
If you are using our seamless integration where the checkout page is hosted under your domain, you will need to register your domain name by sending an email to support@fiuu.com, alternatively you can also register your domain name in merchant portal (Merchant Profile -> Profile Settings). But this is subjected to approval by our operation team. Multiple subdomains are accepted but only one domain is allowed per Merchant ID. On top of that, embeded page (iFrame) is not allowed for seamless integration type.  

<p align="center">
<img src="https://github.com/RazerMS/Cheatsheet-BestPractices-RazerMS_API/assets/19460508/15090976-a0dc-4ae8-8eed-3997c23fd1bd" />
</p>
  
  
### Tips #10. Environment IP whitelisting required when testing payment in demo bank page under sandbox environment   
In order to prevent merchant from accidentally deploying sandbox (demo bank) into their production environment and let customers pay with dummy account, we have applied some measures like a protected username and password for the demo bank, and also the requirement to whitelist your environment IP address (usually it's an IP address which belongs to your working space), under sandbox merchant portal (Merchant Profile -> Profile Settings).  

<p align="center">
<img src="https://github.com/RazerMS/Cheatsheet-BestPractices-RazerMS_API/assets/19460508/54f5edb0-7321-457c-80fe-f4e9f9157bcc" />
</p>

    
### Tips #11. Applying Static QR-Code Generator for offline and online use case  
Online payment refer to ecommerce payment, like shopping cart checkout. While offline payment refer to terminal payment, or kiosk payment. 
For offline payment use case, merchants are advised to generate one QR code for each vending machine respectively. All customers can scan the same QR code to make payment. For the amount, if merchant pre-set the amount before generating the QR code, customers can only pay with the specific amount. If the amount is left empty, then customers are allowed to enter the amount themselves.  
For online payment use case, this solution is also a viable option. But there are few limitations that you need to take into consideration:  
1. Merchants will only receive the notification URL webhook after customer completed payment successfully.
2. For static QR, merchants will receive a similar response as online payment.  
3. Merchants will not know how much customers has paid until payment completed (in case the static QR is not pre-set with any fixed amount).  
4. Order ID will be always static.  

<p align="center">
<img src="https://github.com/RazerMS/Cheatsheet-BestPractices-RazerMS_API/assets/19460508/e1e6664e-76fb-47bc-9238-a78790505408" />
</p>

### Tips #12. Further explanation and ability for retry for each error codes from capture and refund API (response)   

|**Capture API's list of error codes**||||
| - | - | - | :- |
|||||
|**Error Code**|**Error description**|**Further Explanation**|**Can retry?**|
|00|Success|Capture is successful|No, this is the final result|
|11|Failure|Capture has failed|No, this is the final result, could due to rejection by channel / bank|
|12|Invalid or unmatched security hash string|skey calculation incorrect|Yes, can retry after amending the skey by following the correct formula and algorithm|
|13|Not a credit card transaction|The transaction channel does not allow to capture|No, check the transaction ID passed does it belong to a card transaction?|
|15|Requested day is on settlement day|Not allowed to capture if same day as settlement day|Yes, can retry after start of next day of the week|
|16|Forbidden transaction|The transaction channel does not allow partial capture|Yes, can retry after change the amount to full amount|
|17|Transaction not found|Transaction ID not found in Fiuu's database|No, check the transaction ID passed does it belong to a valid Fiuu's transaction?|
|18|Missing required parameter|Mandatory parameter are not passed|Yes, can retry after amending the missing parameter|
|19|Domain not found|Merchant ID not found in Fiuu's database|No, check the merchant ID passed does it belong to Fiuu?|
|20|Temporary out of service|Due to Fiuu's internal error / service interuption|Yes, can retry after Fiuu fix the intermittent issue at their system|
|21|Authorization expired|Exceeded the max period allowed to capture the transaction|No, need to make sure the request is sent within the allowable time frame|
|23|Not allowed to perform partial capture|The transaction channel does not allow partial capture|Yes, can retry after change the amount to full amount|
|24|Transaction has already been captured|The transaction ID passed has already been captured|No, this is the final result|
|25|Amount requested more than available capture amount|The amount passed exceeded the authorized / original amount|Yes, can retry after amending the amount to be less than the authorized / original amount|
|99|General Error(Please check with PG Support)|Due to Fiuu's internal error / service interuption|Yes, can retry after Fiuu fix the intermittent issue at their system|
|||||
|**Refund API's list of error codes**||||
|||||
|**Error Code**|**Error description**|**Further Explanation**|**Can retry?**|
|PR001|Refund Type not found.|Invalid refund type passed|Yes, can retry after amending the refund type parameter|
|PR002|MerchantID field is mandatory|Mandatory parameter are not passed|Yes, can retry after amending the missing parameter|
|PR003|RefID field is mandatory|Mandatory parameter are not passed|Yes, can retry after amending the missing parameter|
|PR004|TxnID field is mandatory|Mandatory parameter are not passed|Yes, can retry after amending the missing parameter|
|PR005|Amount field is mandatory|Mandatory parameter are not passed|Yes, can retry after amending the missing parameter|
|PR006|Signature field is mandatory|Mandatory parameter are not passed|Yes, can retry after amending the missing parameter|
|PR007|Merchant ID not found.|Merchant ID not found in Fiuu's database|No, check the merchant ID passed does it belong to Fiuu?|
|PR008|Invalid Signature.|Signature calculation incorrect|Yes, can retry after amending the Signature by following the correct formula and algorithm|
|PR009|Txn ID not found.|Transaction ID not found in Fiuu's database|No, check the transaction ID passed does it belong to a valid Fiuu's transaction?|
|PR010|Transaction must be in authorized/captured/settled status. Current transaction is in {{TRANSACTION\_STATUS}} status|The transaction ID passed is not in a refundable status|Yes, can retry if the current {{TRANSACTION\_STATUS}} is "pending", no if the current  {{TRANSACTION\_STATUS}} is already "cancelled"|
|PR011|Exceed refund amount for this transaction|The amount passed exceeded the original transaction amount|Yes, can retry after amending the amount to be lower than or equal to the original  transaction amount|
|PR012|Bank information is not applicable for credit channel  transaction|Card transaction does not require beneficiary bank account information|Yes, can retry after removing the beneficiary bank account information|
|PR013|BankCode not found in our database, please contact  support|The bank code passed is not found in Fiuu's database|Yes, can retry after amending the bank code|
|PR014|Bank information is mandatory for non-credit channel  transaction|Non-card transaction require beneficiary bank account  information|Yes, can retry after adding the beneficiary bank account information|
|PR015|Server is busy, try again later|Due to Fiuu's internal error / service interuption|Yes, can retry after Fiuu fix the intermittent issue at their system|
|PR016|Duplicate RefID found, please provide a unique RefID|Fiuu's system does not allow duplicated RefID|Yes, can retry after amending RefID to become unique|
|PR017|Refund request for transaction that is out of the allowed  period|Exceeded the max period allowed to refund the transaction|No, need to make sure the request is sent within the allowable time frame|
|PR018|BeneficiaryName cannot contain non-alphanumeric  characters|Invalid beneficiary bank account information passed|Yes, can retry after amending the beneficiary name|
|PR019|Refund is not allowed / Only partial refund is allowed /  Only full refund is allowed|The transaction channel does not allow full/partial refund|Yes, can retry after amending the amount|
|PR020|Insufficient balance to refund|Not enough account balance left for your current account in Fiuu's system|Yes, can retry after more account balance increased as the result of more incoming transactions being captured|
