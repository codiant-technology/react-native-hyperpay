
<p align="center">
<img  width="600" height="300" src="https://github.com/codianttechnology/react-native-hyperpay/blob/main/react-native-hyperpay.png">
  </p>
  
## React Native Hyper Pay
  
  This library designed to provided hyper pay payment integration to react native using native bridging.
  
### Installation

`$ yarn add react-native-hyperpay`

Or

`$ npm install react-native-hyperpay`

## Usage

### Setup

First of all you need a Hyper Pay account to generate live entityId and Authorization bearer token. Using dummy entityId and Authorization token it is not working (https://www.hyperpay.com/)


### Generate Checkout Id

First we will generate checkout ID using REST API. Also we can generate Checkout ID from server end and get using API or you can generate at our end using call (HyperPay)REST API.
``` 
generateCheckoutID = async () => {
    
    let params = {
      entityId: {Live Entity Id},
      amount: '1',
      currency: 'SAR',
      paymentType: 'DB',
    };
    var formBody = [];
    for (var property in params) {
      var encodedKey = encodeURIComponent(property);
      var encodedValue = encodeURIComponent(params[property]);
      formBody.push(encodedKey + '=' + encodedValue);
    }
    formBody = formBody.join('&');
    let data = await fetch('https://test.oppwa.com/v1/checkouts', {
      method: 'POST', // or 'PUT'
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
        Authorization:
          'Bearer {Live Bearer Token}',
      },
      body: formBody,
    });
    let data_json = await data.json();
    if (data.status == 200) {
      await this.setState({loading: false, checkoutID: data_json.id});
    } 
    }; 
    
 ```
    
## Now you can call this method on checkout button


```  
   onCheckOut = async () => {
       try {
     
      await this.generateCheckoutID();
      
      const paymentParams = {
        checkoutID: {Get From API},
        paymentBrand: 'VISA',
        cardNumber: 4200000000000000,
        holderName: 'Codiant Technology',
        expiryMonth: '12',
        expiryYear: '2025',
        cvv: '123',
      };

      HyperPay.transactionPayment(paymentParams)
        .then((transactionResult) => {
          if (transactionResult) {
            if (!transactionResult.redirectURL)
              return this.setState({loading: false}, () => {
                Alert.alert('Hyper Pay', 'The card data is incorrect');
              });
            else {
              if (transactionResult.status === 'pending') {
                Linking.openURL(transactionResult.redirectURL)
                  .then(() => {
                    this.setState({loading: false});
                  })
                  .catch((err) => {
                    console.log('paymentParams2', err);
                  });
              }
            }
          }
        })
        .catch((err) => {
          console.log('error', err);
        });
    } catch (e) {
      console.log('error', e);
    }
    } 
    
    ```
