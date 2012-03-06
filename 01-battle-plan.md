Goal
----

The goal is to develop a simple Facebook-connectivity app. It should consist of two screens.

1. The first screen just contains a Facebook sign-in button. Upon clicking it the user should be authenticated with Facebook and the app should proceed to the...

2. second screen, which should contain a nicely presented grid with thumbnails of all the friends of the logged-in user, along with their names.

The [Reference card](http://doc.opalang.org/#!/refcard) can be of great help to quickly get up-and-running with Opa. Good luck!

Plan
----

### First screen

Let's first create the static first screen. For the Facebook sign-in button you can use this image: ![Facebook sign-in](https://github.com/akoprow/opa-devcamp-facebook/raw/master/03-solution/resources/fb_connect.png). You'll need to use a server that embeds it as part of application resources: learn how to do that from the [RefCard](http://doc.opalang.org/#!/refcard/Standard-library/Web-features/Server).

### Set-up Facebook app

To connect with Facebook you need to first create a Facebook app that you will be connecting to. Go to https://developers.facebook.com to do that. Mark the `Website` tick and put there the URL at which you'll deploy the app. *This is important*. For security reasons Facebook will only redirect back to this URL. If you're going to deploy locally, just put `localhost:8080` in that field. Note down the `App ID` and `App secret` as you'll need them in the next step.

### Set-up Facebook authentication

You will need a bunch of Facebook libraries

    import stdlib.apis.{facebook, facebook.auth, facebook.graph}

and a record with [`Facebook.config`](http://doc.opalang.org/#!/type/stdlib.apis.facebook/Facebook/config) -- you can fill it in using the data from the previous step.

Now use the [`user_login_url`](http://doc.opalang.org/#!/value/stdlib.apis.facebook.auth/FbAuth/user_login_url) function from the [`FbAuth`](http://doc.opalang.org/#!/module/stdlib.apis.facebook.auth/FbAuth) module to get the URL to which to re-direct for Facebook authentication and put an anchor (`<a>`) with that URL on top of the `Sign in with Facebook` image. For this tutorial you can use an empty list of requested permissions and a re-direct to `http://localhost:8080/connect` (if you deploy locally).

Now upon clicking the image you should be authenticated in Facebook. When authentication is finished you will be redirected to `http://localhost:8080/connect` -- we'll handle this URL in the next step.
