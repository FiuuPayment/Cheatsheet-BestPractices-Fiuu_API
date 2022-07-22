# Cheatsheet-BestPractices-RazerMS_API  

## Objective  
This is a cheatsheet for those developers who is frustrated by the lengthy official docs and having a hard time on deciding which option is the best approach to fit their integration requirements. This README.md file will be updated from time to time when we discovered new pain point faced by developers during the integration process. We welcome every suggestion or enquiry from developers in the form of posting new issue to this repository.  

### Tips #1. Various integration method provided by RazerMS  
RazerMS provides few of the integration methods to suit every business needs.  
#### i. Hosted Payment Page  
A hosted solution where customer will be redirected from merchant site to RazerMS hosted page, channel selection will be done on this page before being redirected to each online banking / ewallet page to proceed with the payment authorization or OTP. After that a success / failure page will be displayed to customer before being redirected back to merchant site.
#### ii. Seamless Integration
Channel selection will be done in the merchant site which require more integration effort from the developers compared to Hosted Payment Page. For card channel the card number will still be input through RazerMS page hence no PCI compliance is required from merchant. 
#### iii. Inpage Chackout
Similar to seamless integration but only support card channel, and the difference compared to seamless integration is the card number input page will be hosted via iFrame on merchant site instead of a popup window.  
#### iv. Mobile XDK  
Best for mobile in-app purchase.
#### v. Direct Server Integration
No UI will be provided by RazerMS, channel selection and card number input will all be done on merchant site, hence require merchant to be PCI compliant.  
#### vi. Recurring API  
No UI will be provided by RazerMS, but it supports recurring payment by any amount at any time.  

#### Comparison Chart
<p align="center">
<img src="https://user-images.githubusercontent.com/19460508/180356625-1e42fc30-9d15-4da6-af04-aab8fd34cf9c.png" />
</p>
  
   
### Tips #2. The 3 endpoints to update payment status on merchant site  
When a payment is done by customer, how do we update the transaction status to the merchant? The 3 endpoints, i.e. Return URL, Notification URL, Callback URL serves as a mechanism to update merchant transaction status.  
#### i. Return URL
Realtime web browser or frontend redirection endpoint for hosted page, seamless integration, and shopping cart module. Impact user's UI/UX or journey in the whole payment process.  
#### ii. Notification URL
Realtime server-to-server or backend endpoint for all kind of integrations and payment channels. Will be triggerred once RazerMS received the payment status from respective online banking / ewallet channels.  
#### iii. Callback URL  
Defer update or callback endpoint on non-realtime payment such as Razer Cash.  
  
  
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
RazerMS will forward the status enquiry by merchant to the respective online banking / ewallet channel, which help the merchant to get realtime transaction status as aligned with the payment channel itself.  
#### ii. Indirect Status Requery
RazerMS will return the transaction status based on our own status within our system.  
  
  
### Tips #6. For void use Reversal API, for refund use Refund API
We have 2 types of Refund, i.e. Reversal and Full / Patial Refund
#### i. Reversal
RazerMS will forward the refund request by merchant to the respective online banking / ewallet channel, and proceed to void the transaction, but this is only allowed on the same day as the transaction itself.  
#### ii. Full / Patial Refund  
RazerMS will process the refund request by merchant ourselves and allow up to 180 days after the transaction date.  
