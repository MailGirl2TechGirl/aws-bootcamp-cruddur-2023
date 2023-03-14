# Week 3 — Decentralized Authentication

#Signin Page

```
import { Auth } from 'aws-amplify';

const [cognitoErrors, setCognitoErrors] = React.useState('');

const onsubmit = async (event) => {
  setCognitoErrors('')
  event.preventDefault();
  try {
    Auth.signIn(email; password)
      .then(user => {
        localStorage.setItem("access_token", user.signInUserSession.accessToken.jwtToken)
        window.location.href = "/"
      })
      .catch(err => { console.log('Error!', err) });
  } catch (error) {
    if (error.code == 'UserNotConfirmedException') {
      window.location.href = "/confirm"
    }
    setCognitoErrors(error.message)
  }
  return false
}
```

![oFTEaXN](https://user-images.githubusercontent.com/124912958/225163141-154c5b2c-4fd3-46a5-a7f3-de36c0a372a2.png)


Finally got the correct error after going back through the videos and making some edits to my code

#Created a user within Cognito

![TDv3I7J](https://user-images.githubusercontent.com/124912958/225163206-16038c69-27f7-497e-b2af-daeafd484885.png)


#Used AWS cli to enter user information to confirm password

```
aws cognito-idp admin-set-user-password --username mailgirl2techgirl --password xxxxxxxxxxx --user-pool-id us-east-1_TpOH8sxOe --permanent
```

![GiGORwg](https://user-images.githubusercontent.com/124912958/225163329-8afa07c8-3e91-4802-b16f-ed88e1f529b2.png)

#Added name and username to user in cognito 

![uUTHgQb](https://user-images.githubusercontent.com/124912958/225163398-9d2e9356-17bd-4f98-93dd-4f99c753bb59.png)

Deleted user within Cognito 

# Signup page
```
import { Auth } from 'aws-amplify';

const onsubmit = async (event) => {
  event.preventDefault();
  setErrors('')
  try {
      const { user } = await Auth.signUp({
        username: email,
        password: password,
        attributes: {
            name: name,
            email: email,
            preferred_username: username,
        },
        autoSignIn: { // optional - enables auto sign in after user is confirmed
            enabled: true,
        }
      });
      console.log(user);
      window.location.href = `/confirm?email=${email}`
  } catch (error) {
      console.log(error);
      setErrors(error.message)
  }
  return false
}
```
# Confirmation Page 
```
import { Auth } from 'aws-amplify';

const resend_code = async (event) => {
    setErrors('')
    try {
      await Auth.resendSignUp(email);
      console.log('code resent successfully');
      setCodeSent(true)
    } catch (err) {
      // does not return a code
      // does cognito always return english
      // for this to be an okay match?
      console.log(err)
      if (err.message == 'Username cannot be empty'){
        setErrors("You need to provide an email in order to send Resend Activiation Code")   
      } else if (err.message == "Username/client id combination not found."){
        setErrors("Email is invalid or cannot be found.")   
      }
    }
  }
  
  const onsubmit = async (event) => {
    event.preventDefault();
    setErrors('')
    try {
      await Auth.confirmSignUp(email, code);
      window.location.href = "/"
    } catch (error) {
      setErrors(error.message)
    }
    return false
  }
  ```
  Getting this error: 
  
 ![IHxfRHk](https://user-images.githubusercontent.com/124912958/225163460-2074d43a-677e-4c2c-894d-a0c76b741fed.png)
  
  Recreated user pool as it was using username and email.
  
  Once the user pool was recreated, I was able to sign up and received a confrimation email BUT now the backend is not working. 
 
![HtqHu0w](https://user-images.githubusercontent.com/124912958/225163677-bdb77d79-3076-4b4f-9023-6317523cbacd.png)
  
  Restarted container and it is now working!
 
![mTzzDAY](https://user-images.githubusercontent.com/124912958/225163515-04391f68-27dc-43f7-918c-6df9e5982179.png)
  
  #Recovery Page
  ```
 import { Auth } from 'aws-amplify'; 
  
const onsubmit_send_code = async (event) => {
    event.preventDefault();
    setErrors('')
    Auth.forgotPassword(username)
    .then((data) => setFormState('confirm_code') )
    .catch((err) => setErrors(err.message) );
    return false
  }
  
  const onsubmit_confirm_code = async (event) => {
    event.preventDefault();
    setErrors('')
    if (password == passwordAgain){
      Auth.forgotPasswordSubmit(username, code, password)
      .then((data) => setFormState('success'))
      .catch((err) => setErrors(err.message) );
    } else {
      setErrors('Passwords do not match')
    }
    return false
  }
```

Chose "Forgot Password" and successfully changed password with recovery
