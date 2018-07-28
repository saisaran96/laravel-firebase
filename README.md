# laravel_firebase

Install Laravel
```
composer global require "laravel/installer"
```
Create new Project 
```
laravel new firebase
```
Then install the Firebase Admin SDK
```
composer require kreait/firebase-php ^4.0
```
Generate and Downlaod Privare key from Firebase and save it to Controller folder

create a controller and add
```

<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Kreait\Firebase\Factory;
use Kreait\Firebase\ServiceAccount;
use Kreait\Firebase\Database;
use Firebase\Auth\Token\Exception\InvalidToken;
use Symfony\Component\Cache\Simple\FilesystemCache;



class FirebaseController extends Controller
{
	public function index(Request $request)
	{

	//replace with your service-account.json file
 		$serviceAccount = ServiceAccount::fromJsonFile(__DIR__.'/google-service-account.json');

		$firebase = (new Factory)
  		->withServiceAccount($serviceAccount)
  		->create();


		$idToken = $request->input('idtoken');


		$idTokenString= (string) $idToken;


		try {
			$verifiedIdToken = $firebase->getAuth()->verifyIdToken($idTokenString);
		} catch (InvalidToken $e) {
			echo $e->getMessage();
		}

		$uid = $verifiedIdToken->getClaim('sub');
		$user = $firebase->getAuth()->getUser($uid);
		

	}
}

```
Create a new blade template and add 
```
<!DOCTYPE html>
<!--
Copyright (c) 2016 Google Inc.
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
http://www.apache.org/licenses/LICENSE-2.0
Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->
<html>
<head>
  <meta charset=utf-8 />
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Email/Password Authentication Example</title>
  <!-- CSRF Token -->
  <meta name="csrf-token" content="{{ csrf_token() }}">

  <!-- Material Design Theming -->
  <link rel="stylesheet" href="https://code.getmdl.io/1.1.3/material.orange-indigo.min.css">
  <link rel="stylesheet" href="https://fonts.googleapis.com/icon?family=Material+Icons">
  <script defer src="https://code.getmdl.io/1.1.3/material.min.js"></script>

  <!-- <link rel="stylesheet" href="main.css"> -->
  <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
  <!-- Import and configure the Firebase SDK -->
  <!-- These scripts are made available when the app is served or deployed on Firebase Hosting -->
  <!-- If you do not serve/host your project using Firebase Hosting see https://firebase.google.com/docs/web/setup -->
  <script src="https://www.gstatic.com/firebasejs/5.3.0/firebase.js"></script>

 <script type="text/javascript">
  /**
     *  Initialize Firebase Here.
     */

    /**
     * Handles the sign in button press.
     */
     function toggleSignIn() {
      if (firebase.auth().currentUser) {
        // [START signout]
        firebase.auth().signOut();
        console.log('signout');
        // [END signout]
    } else {
        var email = document.getElementById('email').value;
        var password = document.getElementById('password').value;
        if (email.length < 4) {
          alert('Please enter an email address.');
          return;
      }
      if (password.length < 4) {
          alert('Please enter a password.');
          return;
      }
        // Sign in with email and pass.
        // [START authwithemail]
        firebase.auth().signInWithEmailAndPassword(email, password).catch(function(error) {
          // Handle Errors here.
          var errorCode = error.code;
          var errorMessage = error.message;
          // [START_EXCLUDE]
          if (errorCode === 'auth/wrong-password') {
            alert('Wrong password.');
        } else {
            alert(errorMessage);
        }
        console.log(error);
        document.getElementById('quickstart-sign-in').disabled = false;
          // [END_EXCLUDE]
      });
        // [END authwithemail]
    }
    document.getElementById('quickstart-sign-in').disabled = true;
}
    /**
     * Handles the sign up button press.
     */
     function handleSignUp() {
      var email = document.getElementById('email').value;
      var password = document.getElementById('password').value;
      if (email.length < 4) {
        alert('Please enter an email address.');
        return;
    }
    if (password.length < 4) {
        alert('Please enter a password.');
        return;
    }
      // Sign in with email and pass.
      // [START createwithemail]
      firebase.auth().createUserWithEmailAndPassword(email, password).catch(function(error) {
        // Handle Errors here.
        var errorCode = error.code;
        var errorMessage = error.message;
        // [START_EXCLUDE]
        if (errorCode == 'auth/weak-password') {
          alert('The password is too weak.');
      } else {
          alert(errorMessage);
      }
      console.log(error);
        // [END_EXCLUDE]
    });
      // [END createwithemail]
  }
    /**
     * Sends an email verification to the user.
     */
     function sendEmailVerification() {
      // [START sendemailverification]
      firebase.auth().currentUser.sendEmailVerification().then(function() {
        // Email Verification sent!
        // [START_EXCLUDE]
        alert('Email Verification Sent!');
        // [END_EXCLUDE]
    });
      // [END sendemailverification]
  }
  function sendPasswordReset() {
      var email = document.getElementById('email').value;
      // [START sendpasswordemail]
      firebase.auth().sendPasswordResetEmail(email).then(function() {
        // Password Reset Email Sent!
        // [START_EXCLUDE]
        alert('Password Reset Email Sent!');
        // [END_EXCLUDE]
    }).catch(function(error) {
        // Handle Errors here.
        var errorCode = error.code;
        var errorMessage = error.message;
        // [START_EXCLUDE]
        if (errorCode == 'auth/invalid-email') {
          alert(errorMessage);
      } else if (errorCode == 'auth/user-not-found') {
          alert(errorMessage);
      }
      console.log(error);
        // [END_EXCLUDE]
    });
      // [END sendpasswordemail];
  }
    /**
     * initApp handles setting up UI event listeners and registering Firebase auth listeners:
     *  - firebase.auth().onAuthStateChanged: This listener is called when the user is signed in or
     *    out, and that is where we update the UI.
     */
     function initApp() {
      // Listening for auth state changes.
      // [START authstatelistener]
      firebase.auth().onAuthStateChanged(function(user) {
        // [START_EXCLUDE silent]
        document.getElementById('quickstart-verify-email').disabled = true;
        // [END_EXCLUDE]
        if (user) {



            var uid = user.uid;
            firebase.auth().currentUser.getIdToken(/* forceRefresh */ true).then(function(idToken) {
            // Send token to your backend via HTTPS
            console.log(idToken);
            $.ajax({
                url:'auth',
                method:'post',
                data:{
                    _token: $('meta[name="csrf-token"]').attr('content') ,
                    idtoken:idToken
                    
                },
                success:function(data)
                {
                    console.log("ok");
                    var id=data['uid'];
                    console.log(data);
                    if(id==uid)  // user is Authenticated

                    { 

                        console.log('success');
                         // write your action here
                    }
                },
                error:function(error)
                {
                    console.log(error);
                }

            });
            idToken=null;
            // ...
        }).catch(function(error) {
             // Handle error
         });
          // User is signed in.

          var emailVerified = user.emailVerified;

          // [START_EXCLUDE]
          document.getElementById('quickstart-sign-in').textContent = 'Sign out';

          if (!emailVerified) {
            document.getElementById('quickstart-verify-email').disabled = false;
        }
          // [END_EXCLUDE]
      } 
        // [START_EXCLUDE silent]
        document.getElementById('quickstart-sign-in').disabled = false;
        // [END_EXCLUDE]
    });
      // [END authstatelistener]
      document.getElementById('quickstart-sign-in').addEventListener('click', toggleSignIn, false);
      document.getElementById('quickstart-sign-up').addEventListener('click', handleSignUp, false);
      document.getElementById('quickstart-verify-email').addEventListener('click', sendEmailVerification, false);
      document.getElementById('quickstart-password-reset').addEventListener('click', sendPasswordReset, false);
  }
  window.onload = function() {
      initApp();
  };
</script>
</head>
<body>
    <div class="demo-layout mdl-layout mdl-js-layout mdl-layout--fixed-header">

      <!-- Header section containing title -->
      <header class="mdl-layout__header mdl-color-text--white mdl-color--light-blue-700">
        <div class="mdl-cell mdl-cell--12-col mdl-cell--12-col-tablet mdl-grid">
          <div class="mdl-layout__header-row mdl-cell mdl-cell--12-col mdl-cell--12-col-tablet mdl-cell--8-col-desktop">
            <a href="/"><h3>Firebase Authentication</h3></a>
        </div>
    </div>
</header>

<main class="mdl-layout__content mdl-color--grey-100">
    <div class="mdl-cell mdl-cell--12-col mdl-cell--12-col-tablet mdl-grid">

      <!-- Container for the demo -->
      <div class="mdl-card mdl-shadow--2dp mdl-cell mdl-cell--12-col mdl-cell--12-col-tablet mdl-cell--12-col-desktop">
        <div class="mdl-card__title mdl-color--light-blue-600 mdl-color-text--white">
          <h2 class="mdl-card__title-text">Firebase Email &amp; Password Authentication</h2>
      </div>
      <div class="mdl-card__supporting-text mdl-color-text--grey-600">
          <p>Enter an email and password below and either sign in to an existing account or sign up</p>

          <input class="mdl-textfield__input" style="display:inline;width:auto;" type="text" id="email" name="email" placeholder="Email"/>
          &nbsp;&nbsp;&nbsp;
          <input class="mdl-textfield__input" style="display:inline;width:auto;" type="password" id="password" name="password" placeholder="Password"/>
          <br/><br/>
          <button disabled class="mdl-button mdl-js-button mdl-button--raised" id="quickstart-sign-in" name="signin">Sign In</button>
          &nbsp;&nbsp;&nbsp;
          <button class="mdl-button mdl-js-button mdl-button--raised" id="quickstart-sign-up" name="signup">Sign Up</button>
          &nbsp;&nbsp;&nbsp;
          <button class="mdl-button mdl-js-button mdl-button--raised" disabled id="quickstart-verify-email" name="verify-email">Send Email Verification</button>
          &nbsp;&nbsp;&nbsp;
          <button class="mdl-button mdl-js-button mdl-button--raised" id="quickstart-password-reset" name="verify-email">Send Password Reset Email</button>

          <!-- Container where we'll display the user details -->

      </div>
  </div>

</div>
</main>
</div>
</body>
</html>
```

in web.php add the following 
```
// Authenticate the user
Route::post('/auth','FirebaseController@index');
```

