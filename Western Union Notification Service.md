# Western Union Notification Server
## Block diagram describing the landscape
![](Western%20Union%20Notification%20Server.png)

## Sequence diagram for post back notifications
![](WU%20Notification%20Server.png)

### Notification server contains following functions
1. Post back messages are created during online transaction processing using a notification server (NS)
2. WU HOST sends a TCP message to Notification Server using proprietary data (Attached is a sample production request)
3. Notification Server - worker thread converts the message to Transactions and persists into a No-SQL d/b
4. NS creates an entry into queue and replies back to WU HOST
5. NS post back processing thread detaches the message and looks up the partner id. 
6. For the configured partners, it gets the Partner URL, protocol, list of fields to send, timeouts and max retries and other authentication details like partner secret 
7. NS creates the post back message and delivers to the partner URL.Following is an example of the post back message
8. Optionally, as part of the post back, it performs OAuth2  authentication
9. It handles the timeouts , max retries and also handles the errors
 


### Following are the components of the demo
1. WU HOST (TCP Client)

```json
NS DI request: 00102190020493030030653991300408USAWUPOS0050310000604DM0100801D00910YDZBDFKUBN01010JMNLQESTNV0110941BSTREET01307NOWHERE01402ME015050427101610201707211301702US01801D01905RAHIM02005PATEL02702IN03503125036035000370100380362504004776704103WMN04402IN04502US04603INR04703USD0510504-17052090428P EDT05410118149346106109982A2512006605US650073081111198009601012701N14816181078118149346115601N15701N15801Y16401019902002600310026304DM0126509AKC1703832800102810102820103010801USA5_S307021130816004511118149346131904930337601N40401N41110OONXDBFIZV41210EXAZYZDAFO47915ORHIU@VZUZN.COM
```

2. Notification Server

```json
{
  "header": {
    "appVersion": "9303",
    "partnerReference": "0400CIfeAe15Cx8Nf94lis1ztpOyIsTDqFChXyx2B+S3GHv4zk88XIRRraRLunIct8bff+0OYje9QG3QyfTeh05zWsRi0YacQZnXEt837N2QQlbk1eXuO7gGCUfgUZuLE2xCJiDbCfVZTGBk19tyNs7g8xGJ4AM/fWkje/NhjqsmrUlBOq4u/kJTQ5hjg11m5Ky0iz/8E93rWp+Tudqy956hlyx/7EP7F6pNMg1EJAZttft4Xa0s3p8L9oiu6TAsPc3U34XS0881xK/k0B48ux+V4QZkghiV83r4spqfERVXY+y8VDdLGBfK9PH/LgnAtMTg2IfHpjJ8Lmci42QzS1ld9bzNM6zByOCPOslGvwbeElrqT3LYd4+PReZ6ZkU8zYcOMU9NspIpVrJXaunC7Rxb9q+5ugPrbmpQgBd3ZmxDfqPMaOvAxZX5B/6jeVm6OgWb2/Ec1nc2SuHj8/or0+QVpL/Xjr7Mr0wVw/g7BMuZtks9GC17jlnhc3BU26Shy4kKyn9oYcKrLSeMe2IZAf6TxnqrGeTybre1isfA"
  },
  "_class": "com.wu.ns.TxnPrototype.entities.Transaction",
  "messageType": "SEND",
  "payload": {
    "txn": {
      "deliveryService": "US650",
      "financials": {
        "principal": 125,
        "charges": 500,
        "countyTax": 0,
        "plusChrg": 0,
        "payoutAmt": 7767,
        "municipalTax": 0,
        "stateTax": 0,
        "grossAmount": 625
      },
      "corridor": {
        "pay": {
          "isoCountry": "IN",
          "isoCurrency": "INR"
        },
        "send": {
          "isoCountry": "US",
          "isoCurrency": "USD"
        }
      },
      "sendAgent": {
        "accountId": "AKC170383",
        "address": [],
        "phone": [],
        "bingo": "100",
        "dma": "100",
        "networkId": "DM01",
        "counterId": "DM01"
      }
    },
    "receiver": {
      "name": {
        "nameType": "D",
        "last": "PATEL",
        "first": "RAHIM"
      },
      "address": [
        {
          "countryIso": "IN"
        }
      ],
      "phone": []
    },
    "sender": {
      "name": {
        "nameType": "D",
        "last": "JMNLQESTNV",
        "first": "YDZBDFKUBN"
      },
      "complianceDetails": {
        "otherAddresses": [],
        "iActOnMyBehalf": "N",
        "identifications": []
      },
      "address": [
        {
          "addressLine1": "41BSTREET",
          "state": "ME",
          "city": "NOWHERE",
          "countryIso": "US",
          "postalCode": "04271"
        }
      ],
      "phone": [
        {
          "number": "2017072113"
        }
      ]
    }
  }
}
```

3. Partner OAuth2 server

```json
response: {access_token=6d56070a-85f8-4a7b-a4f1-c2fa7280dce9, token_type=bearer, expires_in=599, scope=read write}
```

4. Partner Application
```json
response: {transaction_id=1810781181493461, header={partner_reference= 0400CIfeAe15Cx8Nf94lis1ztpOyIsTDqFChXyx2B+S3GHv4zk88XIRRraRLunIct8bff+0OYje9QG3QyfTeh05zWsRi0YacQZnXEt837N2QQlbk1eXuO7gGCUfgUZuLE2xCJiDbCfVZTGBk19tyNs7g8xGJ4AM/fWkje/NhjqsmrUlBOq4u/kJTQ5hjg11m5Ky0iz/8E93rWp+Tudqy956hlyx/7EP7F6pNMg1EJAZttft4Xa0s3p8L9oiu6TAsPc3U34XS0881xK/k0B48ux+V4QZkghiV83r4spqfERVXY+y8VDdLGBfK9PH/LgnAtMTg2IfHpjJ8Lmci42QzS1ld9bzNM6zByOCPOslGvwbeElrqT3LYd4+PReZ6ZkU8zYcOMU9NspIpVrJXaunC7Rxb9q+5ugPrbmpQgBd3ZmxDfqPMaOvAxZX5B/6jeVm6OgWb2/Ec1nc2SuHj8/or0+QVpL/Xjr7Mr0wVw/g7BMuZtks9GC17jlnhc3BU26Shy4kKyn9oYcKrLSeMe2IZAf6TxnqrGeTybre1isfA}}
```

 
### Questions:

1. What are the list of fields that are required by LBP. Attached are the list of fields that WU can typically send
2. How does LBP use these fields? E.g., do they encrypt the privacy/payment data
3. Does LBP handle duplicate messages? 
4. What all can be the error responses from LBP for the Post back messages?
