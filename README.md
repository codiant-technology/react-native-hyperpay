
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
    };```
