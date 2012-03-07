Your first Facebook-Opa app
---------------------------

Hopefully by now you know how easy it is to create web apps in Opa (if you don't and/or are new around here I suggest you explore [Opa blog](http://blog.opalang.org) that contains a number of tutorials, also for beginners), but as we all know web apps hardly ever live in isolation. More often than not they interact with other services and platforms. Today we will develop a simple Opa app that will connect with Facebook. Ready? You better...

The goal is to develop a simple Facebook-connectivity app. It should consist of two screens.

1. The first screen just contains a Facebook sign-in button. Upon clicking it the user should be authenticated with Facebook and the app should proceed to the...

2. second screen, which should contain a nicely presented grid with thumbnails of all the friends of the logged-in user, along with their names.

A rough guideline on how to do that is provided in [`01-battle-plan.md`](https://github.com/akoprow/opa-devcamp-facebook/blob/master/01-battle-plan.md), you should try that first and only move to this more detailed description if lost.

By completing this tutorial you will learn how to:

* use Opa's Facebook API to connect with user's Facebook account.
* write a simple Opa app that will show given user's name and thumbnails of her friends (both data obtained from Facebook).
* use Bootstrap to get a nice UI easily.
* use database to store Facebook app configuration, which should not be visible in the source code...
* and how to use application's command line arguments to initialize it.

### Setting up a Facebook app

To get started we need to create our app's profile on Facebook. To do that go to https://developers.facebook.com. You'll see the following status bar.

![Facebook Developer topbar](https://github.com/akoprow/opa-devcamp-facebook/raw/master/img/fb_topbar.png)

Go the Apps (highlighted in the above screenshot) and choose `Create New App`.

![Facebook Developer new app](https://github.com/akoprow/opa-devcamp-facebook/raw/master/img/fb_new_app.png)

That will open a dialog with basic info about the new application. For now just put `OpaIntro1` as _App display name_ (we'd prefer something like `HelloFacebook`, but using Facebook trademarks in app names is prohibited) and agree to the Facebook Platform Policies (preferably, after reading ;).

![Facebook Developer new app dialog](https://github.com/akoprow/opa-devcamp-facebook/raw/master/img/fb_new_app_dialog.png)

Then (possibly after answering a Captcha) you will be presented with the following settings screen.

![Facebook Developer app settings](https://github.com/akoprow/opa-devcamp-facebook/raw/master/img/fb_app_settings.png)

Firstly note on the top of the screen the `App ID` and `App Secret` (they're blurred in the screenshot as the secret *should not be shared* with anyone) -- we will need those values in a minute. Then in the `Select how your app integrates with Facebook` section (bottom of the screen; note that we cut out the `Basic info` and `Cloud services` sections of the settings in the screenshot) select the `Website` mark, put the URL at which your application will be hosted -- it's important to get that right as the Facebook login will only redirect to this site's URL -- and click `Save changes`. If you will be testing your application locally, on the default port of Opa (`8080`) then you should put as an URL: `http://localhost:8080`

Ok, we're all set on the Facebook side; now let's do some Opa coding!

### Authenticating with Facebook

The goal of the app is simple: allow users to login via Facebook and upon connection greet them with their name and a quick info on the number of friends they have.

Since we will be using Facebook API, including authentication and [Facebook Graph](http://developers.facebook.com/docs/reference/api) to have some basic info about the logged-in user let's start with some imports.

    import stdlib.apis.{facebook, facebook.auth, facebook.graph}

Then we need to fill in some basic info about the application; we'll put this inside a module.

    module OpaIntro1 {
      config =
        { app_id: "xxxxxxxxxxxxxxx"
        , api_key: "xxxxxxxxxxxxxxx"
        , app_secret: "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
        }
    }}

Of course the x's need to be replaced with the real data here -- `app_id` and `app_key` are the same (there used to be a distinction) and correspond to `App ID` in the Facebook settings screen, whereas `app_secret` is the value of `App secret`.

Now we initialize the authentication module with this configuration, make an abbreviation for the Facebook Graph module and define the redirect URL -- this is where Facebook will redirect after authenticating the user.

    FBA = FbAuth(OpaIntro1.config)
    FBG = FbGraph
    redirect = "http://facebook-01.tutorials.opalang.org/connect"

This tutorial will end up being hosted at the domain name indicated in the `redirect` address above; again if you're just playing with it locally then you should rather use `http://localhost:8080/connect` as a redirect URL.

We will have two pages in our application: the initial screen and the page where Facebook will redirect to upon completing authentication. Let's write a URL dispatcher for our app.

    dispatcher = parser {
    case "/connect?" data=(.*) -> connect(Text.to_string(data)) |> page
    case .* -> main() |> page
    }

The `/connect` page is where we are redirected upon completing authentication (see the `redirect` variable before); it will have an extra parameter after the question mark, which is the Facebook authentication token that we will use for all functionality requiring Facebook login (in our app: to get some data about the logged-in user). Any other URL in our sample app is handled with the welcome screen that we will create in a minute. The `page` function is just a simple wrapper to create an HTML-page resource (we skip it here but we encourage curious readers to consult the complete listing at the end of the article).

The `connect` function takes as an argument the part of the URL after the question mark. First we need to extract a token from this string with

    FBA.get_token_raw(data, redirect)

which optionally returns a token that we can subsequently use to get some user data, as we will see in the next section.

### Getting user Facebook data

For that we will use the Facebook Graph API, which one can explore at: https://developers.facebook.com/tools/explorer. Two functions that are used for this are:

    FBG.Read.object(id, options)
    FBG.Read.connection(id, connection, token, paging)

For instance reading object `"me"` (try `https://graph.facebook.com/me` in the Facebook API explorer) will give basic information about the logged-in user (we will extract name from there), whereas reading connection `"friends"` of object `"me"` (try `https://graph.facebook.com/me/friends`) will give the list of user's friends.

The returned data is in [JSON](http://doc.opalang.org/#!/type/stdlib.core.rpc.core/RPC/Json/json) format, so we have to do a bit of processing to extract the data we need. See function `get_name` in the complete listing below for details on how to get logged-in user's name. Similarly consult `get_friends_ids` to see how to obtain ids of given user's friends, which then is used in the `main_msg` function where we get a thumbnail of user with given `id` using the `FBG.Read.picture_url` function.

Ideally, the API should be extended with more high level functions that would provide a nice mapping from the JSON Facebook API to a type-safe world of Opa (essentially doing what we did in those functions but for all possible objects/connections). This can be done of course (volunteers?) but is a bit complicated by the fact that the shape of the information for different queries depends on application permissions.

Now, we are ready to construct our two pages: the login screen that will show the Facebook connect button and the main screen that is visible after login. Let's start with the main page.

    function main() {
      login_url = FBA.user_login_url([], redirect)
      <a href="{login_url}"><img src="resources/fb_connect.png" /></a>
    }

The `user_login_url` function of the authentication module of the Opa Facebook API gives us the URL to redirect to in order to invoke authentication. The first argument is the list of permissions that we require -- for now, we keep it empty. Then we just create a regular link to that address and put a nice image for it. And that's it.

The second screen presents the friends thumbnails using [Bootstrap's Thumbnails](http://twitter.github.com/bootstrap/components.html#thumbnails) markup, together within a [Bootstrap alert](http://twitter.github.com/bootstrap/components.html#alerts) window. It also uses some functions on [lists](http://doc.opalang.org/#!/refcard/Standard-library/Containers/list) to traverse through the list of friends and generate the markup.

### Using database to store Facebook configuration

So far we were storing Facebook configuration (App ID and key) in the source code of the application. You may have realized that this is not ideal, as we may want to distribute the sources of our app, but **one should never disclose the secret key**. One solution, which we will employ here, is to store the configuration in application's database and initialize it via command line arguments.

First the database declaration; easy enough:

    database Facebook.config /facebook_config

Now we want to read this configuration upon starting the program and if it's missing complain and stop the application.

    import stdlib.system
    
    server protected config =
      match (?/facebook_config) {
      case {some: config}: config
      default:
         // In case we fail, we display an error message
        Log.error("Facebook[config]", "Cannot read Facebook configuration (application id and/or secret key)
    Please re-run your application with: --fb-config option")
         // ... and quite
        System.exit(1)
      }

We declare a global value `config`, so it will be initialized before server execution; we also make it `server protected` to make sure it cannot be accessed by the client. Instead of a regular DB read `/facebook_config` we prefix it with question mark (`?`), which results in returning an option type, with `none` given if the value of the given database path has not been initialized (regular read, would just return default value here). Finally we use the `Log` to display the error message and quit the program with `System.exit` (that's what we needed the `stdlib.system` import for).

Ok, now it's time to actually handle the `--fb-config` command line argument that we mentioned in the error message. Command line arguments in Opa applications are grouped into families (each family grouping arguments for a certain type of app functionality). We will now describe such a family:

    fb_cmd_args =
      { init: void,
        parsers: [{ CommandLine.default_parser with
          names: ["--fb-config"],
          param_doc: "APP_ID,APP_SECRET",
          description: "Sets the application ID for the associated Facebook application",
          function on_param(state) {
            parser {
            case app_id=Rule.alphanum_string [,] app_secret=Rule.alphanum_string:
              /facebook_config <- {~app_id, api_key: app_id, ~app_secret};
              {no_params: state}
            }
          }
        }],
        anonymous: [],
        title: "Facebook configuration"
      }

A family consists of its name (`title`), a list of parsers for anonymous arguments (`anonymous`), a list of parsers for non-anonymous arguments (`parsers`) and the initial state of the family (`init`). Command line parsing is based on the notion of a state. The family starts with the initial state, which then can be modified by the encountered arguments. Since we want to react to arguments by storing them in the database, in our example we will use empty state (`void`) and will never modify it.

Few words on the `parsers` field. It contains a list of parsers that can usually be defined by extending `CommandLine.default_parser`. The fields contain: synonymous names of the argument this parser handles (`names`), description of the argument (`description`) and its parameters (`param_doc`), both to be used in the documentation and `on_param` which is a parser for the argument, parametrized by the current state (`state`). In our case we just parse parameters of the shape `APP_ID,APP_SECRET`, store the results in the database and indicate that there will be no more parameters following.

If you want to learn more about command line parsing in Opa then [CommandLine module](http://doc.opalang.org/#!/module/stdlib.core.args/CommandLine) is a great place to start.

### Putting it all together

You can take a look at the [complete solution](https://github.com/akoprow/opa-devcamp-facebook/tree/master/03-solution) for more details and guidance. But remember that to really learn something is much better to get it done on one's own :).

