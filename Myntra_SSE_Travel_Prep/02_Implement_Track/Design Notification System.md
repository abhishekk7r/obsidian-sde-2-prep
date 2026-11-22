1. What kind of notifications are we dealing with? 
	1. Can we lose some notification and still be fine with it?
	2. OTPs, Order Updates or Promotions
	3. How fast? OTPs immediately, promotional can get delayed


### Functional Requirements

1. Order/External Service should be able to queue notification in our service: an OTP, an order update, a payment receipt.
2. Anyone can see the status of a notification.
3. Teams can run promotional notification for some users or everyone. 
4. Each notification goes through selected providers and fallbacks for the critical notifications.

### Non - Functional Requirements

|           | Transactional | Promotional |
| --------- | ------------- | ----------- |
| Durablity |               |             |

