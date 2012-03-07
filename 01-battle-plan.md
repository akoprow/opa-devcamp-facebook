Goal
----

The goal is to develop a simple Facebook-connectivity app. It should consist of two screens.

1. The first screen just contains a Facebook sign-in button. Upon clicking it the user should be authenticated with Facebook and the app should proceed to the...

2. second screen, which should contain a nicely presented grid with thumbnails of all the friends of the logged-in user, along with their names.

Below is the rough sketch of the steps you will need to complete to develop this app. Of course you don't need to follow them tightly but doing so will provide you with guidance at every steps. Oh, and don't worry -- you won't be able to complete all steps in the alloted 1 hour... unless you're a serious code ninja.

The [Reference card](http://doc.opalang.org/#!/refcard) can be of great help to quickly get up-and-running with Opa.

Good luck!

P.S. If you want to get in touch with me write to: Adam.Koprowski@mlstate.com or give me a shout on Twitter: [@akoprowski](http://twitter.com/akoprowski).

Plan
----

### First screen

Let's first create the first static screen. For the Facebook sign-in button you can use this image: ![Facebook sign-in](https://github.com/akoprow/opa-devcamp-facebook/raw/master/03-solution/resources/fb_connect.png). You'll need to use a server that embeds it as part of the application resources: learn how to do that from the [RefCard](http://doc.opalang.org/#!/refcard/Standard-library/Web-features/Server).

### Set-up Facebook app

To connect with Facebook you need to first create a Facebook app that you will be connecting to. Go to https://developers.facebook.com to do that. Mark the `Website` tick and put there the URL at which you'll deploy the app. **This is important**. For security reasons Facebook will only redirect back to this URL. If you're going to deploy locally, just put `localhost:8080` in that field. Note down the `App ID` and `App secret` as you'll need them in the next step.

### Set-up Facebook authentication

You will need a bunch of Facebook libraries, so import them first.

    import stdlib.apis.{facebook, facebook.auth, facebook.graph}

Now you need to create a record with [`Facebook.config`](http://doc.opalang.org/#!/type/stdlib.apis.facebook/Facebook/config) -- you can fill it in using the data from the previous step. Both `app_id` and `api_key` should be the same and correspond to `App ID` in Facebook.

Now use the [`user_login_url`](http://doc.opalang.org/#!/value/stdlib.apis.facebook.auth/FbAuth/user_login_url) function from the [`FbAuth`](http://doc.opalang.org/#!/module/stdlib.apis.facebook.auth/FbAuth) module to get the URL to which to re-direct for Facebook authentication and put an anchor (`<a>`) with that URL on top of the `Sign in with Facebook` image. For this tutorial you can use an empty list of requested permissions and a re-direct to `http://localhost:8080/connect` (if you deploy locally).

Now upon clicking the image you should be authenticated in Facebook. When authentication is finished you will be redirected to `http://localhost:8080/connect` -- we'll handle this URL in the next step.

### Displaying logged-in user name

Ok, to handle more pages in your app you will need a [`{ custom : ... }`](http://doc.opalang.org/#!/refcard/Standard-library/Web-features/Server) server along with an URL dispatcher. Such a dispatcher is a *parser* that takes an URL and produces a resource for it. If you're feeling adventurous then you can read some [blog articles on parsing in Opa](http://blog.opalang.org/search/label/parsing) (start with the oldest and work your way backwards) and write this parser yourself. Otherwise here's how it could look like:

    dispatcher = parser {
    case "/connect?" data=(.*) -> Resource.html("Connected!", connect_page(Text.to_string(data)))
    case .* -> Resource.html("Connect to FB", main())
    }

which calls `connect_page` with Facebook token as an argument for the Facebook connection URL and the `main` page for all other requests. Both those functions should return HTML content of the page.

As a next step you need to convert the `data` page argument into a valid Facebook token, with the [`get_token_raw`](http://doc.opalang.org/#!/value/stdlib.apis.facebook.auth/FbAuth/get_token_raw) function.

Once you have a valid token you can get the user's name with the [`FbGraph.Read.object`](http://doc.opalang.org/#!/value/stdlib.apis.facebook.graph/FbGraph/Read/object) function, as follows:

    FbGraph.Read.object("me", {FbGraph.Read.default_object with token: ...})

You will need to extract the data you want from the JSON object that you will get as response; to learn more about the format go an check the [Facebook Graph API Explorer](https://developers.facebook.com/tools/explorer/?method=GET&path=me) and [Opa's JSON](http://doc.opalang.org/#!/type/stdlib.core.rpc.core/RPC/Json/json) data-type.

### Displaying the grid of friends

You can obtain user's friends in a similar way to getting user's name. Try it in the [API explorer](https://developers.facebook.com/tools/explorer/?method=GET&path=me%2Ffriends) and do the same in Opa with:

    FbGraph.Read.connection("me", "friends", token, FbGraph.default_paging)

Again you will need to do a bit of JSON magickery to get the data you need, but this time working with [lists](http://doc.opalang.org/#!/refcard/Standard-library/Containers/list).

You can present the thumbnails for instance using [Bootstrap's Thumbnails](http://twitter.github.com/bootstrap/components.html#thumbnails) markup, put within an [Bootstrap Alert](http://twitter.github.com/bootstrap/components.html#alerts) window.

### Storing configuration externally

So far we were storing Facebook configuration (App ID and key) in the source code of the application. You may have realized that this is not ideal, as we may want to distribute the sources of our app, but **one should never disclose the secret key**.

One solution is to store the configuration in application's database and initialize it via command line arguments.

If you've gotten that far you should congratulate yourself, as I don't think it's realistic to do all of the above in under 1h in a programming language one does not know (yet).

That also means that you probably need a challenge! So for this task I'm not going to provide much detail I'll just point you to the [Database chapter in the manual](http://doc.opalang.org/#!/manual/Hello--database) and mention that you should be using the [CommandLine module](http://doc.opalang.org/#!/module/stdlib.core.args/CommandLine) for processing command line arguments.
