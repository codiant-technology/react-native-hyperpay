
<p align="center">
<img  width="600" height="300" src="https://github.com/codianttechnology/react-native-hyperpay/blob/main/react-native-hyperpay.png">
  </p>
  
## React Native Hyper Pay (Only for Android)
  
  This library designed to provided hyper pay payment integration to react-native using native bridging.
  
### Installation

`$ yarn add react-native-hyperpay`

Or

`$ npm install react-native-hyperpay`

### Android

There are some extra steps need to do in android for deep linking

 ``` 
          <intent-filter android:label="@string/app_name">
              <action android:name="android.intent.action.VIEW" />
              <category android:name="android.intent.category.DEFAULT" />
              <category android:name="android.intent.category.BROWSABLE" />
              <data
                  android:host="result"
                  android:scheme="hyperpay" />
          </intent-filter>
          
  ```


## Usage

### Setup

First of all you need a Hyper Pay account to generate live entityId and Authorization bearer token. Using dummy entityId and Authorization token it is not working (https://www.hyperpay.com/)

For API server set up please check following link

https://wordpresshyperpay.docs.oppwa.com/tutorials/mobile-sdk/integration/server

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

## Add following listen inside componentDidMount() after payment screen redirect to browser this listener use to listen to payment callback

  ```
  componentDidMount = () => {
    Linking.getInitialURL().then(async (url) => {
      let deepLinkUrl = await AsyncStorage.getItem('deepLinkUrl');
      if (url && url !== deepLinkUrl) {
        let regex = /[?&]([^=#]+)=([^&#]*)/g,
          params = {},
          match;
        while ((match = regex.exec(url))) {
          params[match[1]] = match[2];
          console.log(match[1], match[2]);
        }

        const {id, resourcePath} = params;
        console.log('getInitialURL', resourcePath, url, id);
        if (id) {
          await AsyncStorage.setItem('deepLinkUrl', url);
          this.getPaymentStatus(resourcePath);
        }
      } else {
      }
    });
    let subscription = DeviceEventEmitter.addListener(
      'transactionStatus',
      this.onSessionConnect,
    );

    Linking.addEventListener('url', (e) => {
      const {url} = e;
      if (url) {
        let regex = /[?&]([^=#]+)=([^&#]*)/g,
          params = {},
          match;
        while ((match = regex.exec(url))) {
          params[match[1]] = match[2];
          console.log(match[1], match[2]);
        }

        const {id, resourcePath} = params;
        this.getPaymentStatus(id);
      } else {
        console.log('outside get initialUrl method');
      }
    });
  }; 
  
  ```
  
  
  ## After received callback We will call getPaymentStatus api to get payment status. We can call this api at our end or implement at SERVER end.
  
   ```
   getPaymentStatus = async (resourcePath) => {
    try {
      this.setState({
        checkoutID: '',
        paymentBrand: '',
        cardNumber: '',
        holderName: '',
        expiryMonth: '',
        expiryYear: '',
        selected_paymentType: 0,
        loading: true,
      });

      var myHeaders = new Headers();
      myHeaders.append('Content-Type', 'application/x-www-form-urlencoded');
      myHeaders.append(
        'Authorization',
        'Bearer {Use live token which get from hyper console}',
      );

      var urlencoded = new URLSearchParams();
      urlencoded.append('entityId', {get from hyper pay console});

      var requestOptions = {
        method: 'GET',
        headers: myHeaders,
        redirect: 'follow',
      };

      await AsyncStorage.getItem('CHECKOUT_ID').then(async (value) => {
        let id = JSON.parse(value);

        await fetch(
          'https://test.oppwa.com/v1/checkouts/' +
            id +
            '/payment?entityId=' +
            entityId,
          requestOptions,
        )
          .then((response) => response.text())
          .then(async (response) => {
            console.log('Payment Status ====>' + JSON.stringify(response));
            let responseJson = await response;
            let result = JSON.parse(responseJson);
            if (result && result.result && result.result.code) {
              if (result.result.code == '000.100.110') {
                Alert.alert('Hyper Pay', 'Thanks for your payment');
              }
            }
          })
          .catch((error) => console.log('error', error));
      });
    } catch (e) {
      console.log('error', e.toString());
    }
  }; 
  
  ```
  
