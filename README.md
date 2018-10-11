# vue-google-oauth2-gapi
Handling Google sign-in and sign-out for Vue.js applications

Forked from https://github.com/wat4dog/vue-google-oauth2

Same as fork but change api url and gapi loading process.

This package contains Google API script loading before calling that (it was required for my project and I wasn't able to wait for merge my PR into repository forked from)

## Installation
```
npm install github:idani/vue-google-oauth2 --save
```

## Initialization
```javascript
import GAuth from 'vue-google-oauth2-gapi'

Vue.use(GAuth, {clientId: '4XXXXXXXX93-2gqknkvdjfkdfkvb8uja2k65sldsms7qo9.apps.googleusercontent.com', scope: 'profile email https://www.googleapis.com/auth/plus.login'})

```
Ideally you shall place this in your app entry file, e.g. src/main.js

## Usage - Sign-in
### (a) Handling Google sign-in, getting the one-time authorization code from Google

#### Frontend-side(Vue.js)
```javascript
this.$gAuth.getAuthCode(function (authorizationCode) {
	//on success
	this.$http.post('http://your-backend-server.com/auth/google', { code: authorizationCode, redirect_uri: 'postmessage' }).then(function (response) {

	})
}, function (error) {
	//on fail do something
})
```
The `authorizationCode` that is being returned is the `one-time code` that you can send to your backend server, so that the server can exchange for its own access token and refresh token.

#### Backend-side(Golang)
```go
auth_code := ac.Code //from front-end side
// generate a config of oauth
conf := &oauth2.Config{
	ClientID:     "XXXXXXXX",
	ClientSecret: "XXXXXXXX",
	RedirectURL:  "postmessage",
	Scopes: []string{
		"profile",
		"email",
		"https://www.googleapis.com/auth/plus.login",
	},
	Endpoint: "XXXXXX",
}
// exchange to token inclued refresh_token from code
token, err = conf.Exchange(oauth2.NoContext, auth_code)
if err != nil {
		sErr := NewStatusErr(401, err.Error(), "Unauthorized")
		return nil, &sErr
}
```
Note, ```RedirectURL``` must be ```postmessage```!!



### (b) Alternatively, if you would like to directly get back the access_token and id_token
```
this.$gAuth.signIn(function (user) {
	//on success do something
	console.log('user', user)
}, function (error) {
	//on fail do something
})
```

The `googleUser` object that is being returned will be:
```
{
  "token_type": "Bearer",
  "access_token": "xxx",
  "scope": "xxx",
  "login_hint": "xxx",
  "expires_in": 3600,
  "id_token": "xxx",
  "session_state": {
    "extraQueryParams": {
      "authuser": "0"
    }
  },
  "first_issued_at": 1234567891011,
  "expires_at": 1234567891011,
  "idpId": "google"
}
```

## Usage - Sign-out
Handling Google sign-out
```
this.$gAuth.signOut(function () {
  // things to do when sign-out succeeds
}, function (error) {
  // things to do when sign-out fails
})
```

## Additional Help
Do refer to this [sample login page HTML file](https://github.com/idani/vue-google-oauth2/blob/master/sample.html).

If you are curious of how the entire Google sign-in flow works, please refer to the diagram below
![Google Sign-in Flow](http://i.imgur.com/BQPXKyT.png)
