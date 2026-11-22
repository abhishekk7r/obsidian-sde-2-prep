
What kind of notifications are we dealing with? 
	1. Can we lose some notification and still be fine with it?
	2. OTPs, Order Updates or Promotions
	3. How fast? OTPs immediately, promotional can get delayed


### Functional Requirements

1. Order/External Service should be able to queue notification in our service: an OTP, an order update, a payment receipt.
2. Anyone can see the status of a notification.
3. Teams can run promotional notification for some users or everyone. 
4. Each notification goes through selected providers and fallbacks for the critical notifications.

### Non - Functional Requirements

|           | Transactional                                      | Promotional                                             |
| --------- | -------------------------------------------------- | ------------------------------------------------------- |
| Durablity | OTPs can never be lost                             | Promotional notification can be lost sometime           |
| Latency   | OTPs needs to be sent within seconds               | Fine, if it take minutes                                |
| Delivery  | Deduplication, must be seen once                   | At most once per user campaing                          |
| Isolation | A Campaign cannot slow down the transactional lane | Only uses that capacity that transactional is not using |

