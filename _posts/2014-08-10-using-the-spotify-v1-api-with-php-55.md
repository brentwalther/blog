---
layout: post
title: Using the Spotify v1 API with PHP 5.5
category: code
---
The usage of web APIs varies largely among different languages. Spotify's v1 API is very powerful and has a lot of potential for cool web apps but it's authorization methods are different than those I've worked with in the past. Thus, I wanted to create this quick reference to make it easy for other PHP developers to start using the Spotify API.

For the purposes of this reference I'm using the Laravel MVC framework but the code should extend to work in many other ways as well.

### Obtaining an Authorization Code from users
Before making any API calls that use user's information, we need to obtain an authorization code for that user. This is a multi step process:

#### Step one: Requesting Authorization
The first step is to let Spotify know that we'd like to authorize ourselves to retrieve user credentials. In my application, I make a GET route at the `/login` path and have that redirect the user to Spotify (with all my parameters inserted). Here's an example:

{% highlight php startinline=true %}
Route::get('/login', function() {
  $scopes = 'playlist-modify-public playlist-modify-private playlist-read-private user-read-email';
  $login_key = Hash::make("" . mt_rand() . Config::get('spotify.CLIENT_ID'));
  Session::put('login_key', $login_key);

  $params = array(
    'client_id' => Config::get('spotify.CLIENT_ID'),
    'response_type' => 'code',
    'redirect_uri' => Config::get('spotify.REDIRECT_URI'),
    'state' => $login_key,
    'scope' => $scopes,
  );
  return Redirect::to('https://accounts.spotify.com/authorize?' . http_build_query($params));
});
{% endhighlight %}

Here's a breakdown of what's happening here:

1. I make a space delimited list of all the [user scopes](https://developer.spotify.com/web-api/using-scopes/#list-of-scopes "Spotify User Scopes") that I want to have access to.
2. I make a secret hash of the 'state' that I'm going to pass to Spotify. This is a measure to prevent hackers from tampering with my authorization flow. This doesn't need to be incredibly secure but it should be fairly non-deterministic. I settled for something simple.
3. I store the secret hash in the session so that I can check it when Spotify redirects the user back to me.
4. I make a parameter array of all the parameters discussed [here](https://developer.spotify.com/web-api/authorization-guide/#tablepress-64 "Spotify Authorization Parameters"). You can hard code the client ID if you need to.
5. `spotify.REDIRECT_URI` represents the URL that I Spotify to redirect the user back to on my application. This URL **must** be in your Redirect URI whitelist for your App on Spotify's Developer site. This value is set to `http://localhost:8000/auth` in my case since I'm developing locally.
6. I send the user to Spotify's authorization URL with all my parameters. The [http_build_query()](http://php.net/manual/en/function.http-build-query.php "PHP's http_build_query() reference page") function is very helpful here.

When you send the user to Spotify, they will be presented with a screen asking them to grant the permissions that you specified in `scopes` to your application. Assuming they accept, they will be redirected to your REDIRECT_URI.

#### Step two: Handling the redirect
If the user grants your app access to their credentials, Spotify will redirect them back to your REDIRECT_URI with `code` and `state` parameters. You need to grab these and do the next step of authorization on the server side. Your server will use the `code` to communicate with Spotify directly (without user intervention) to obtain a Authorization code.

Here's my code:

{% highlight php startinline=true %}
Route::get('auth', function() {

  // Check first for things that could be wrong
  if (strcmp(Session::get('login_key'), Input::get('state')) != 0) {
    return "Incorrect state. This request is invalid.";
  } else if (!empty(Input::get('error'))) {
    return "Some kind of error occurred.";
  }

  $postUrl = 'https://accounts.spotify.com/api/token';
  $params = array(
    'grant_type' => 'authorization_code',
    'code' => Input::get('code'),
    'redirect_uri' => Config::get('spotify.REDIRECT_URI'),
    'client_id' => Config::get('spotify.CLIENT_ID'),
    'client_secret' => Config::get('spotify.CLIENT_SECRET'),
  );

  // use key 'http' even if you send the request to https://...
  $options = array(
      'http' => array(
          'header'  => "Content-type: application/x-www-form-urlencoded\r\n",
          'method'  => 'POST',
          'content' => http_build_query($params),
      ),
  );
  $context  = stream_context_create($options);
  $result = json_decode(file_get_contents($postUrl, false, $context));

  Session::put('access_token', $result->access_token);
  return Redirect::to('/');
});
{% endhighlight %}

Here's the breakdown:

1. Firstly, I check to make sure the `state` parameter is the same state that we've stored in our session. If it's not, someone must be trying to tamper with our authorization flow and we need to simple quit.
2. Next, I check to make sure that Spotify didn't return an error. If we catch an error, it's most likely cause it that the user did not give us access. We need to handle this accordingly.
3. Assuming that everything is kosher, we need to build a request to obtain the authorization code from Spotify. I first build an array of all our parameters. Some specific things to mention here: Make sure your CLIENT SECRET is always kept safe. It's your App's 'password' essentially so always keep it safe. Also, the `redirect_uri` parameter must be the same value that you used for the call to Spotify's `/authorize`.
4. We create a POST request with out parameters and then send it using file_get_contents() and the syntax above. This is a foolproof method that doesn't require use of CURL. 
5. **Step Three (the last step)** I convert the JSON response into a PHP object and store the access token into the Session. The access token that is returned is the one we'll use for API calls. There is also an `expires_in` for the token and a `refresh token`. After the number of seconds in `expires_in` has passed, you'll need to use this `refresh_token` to get a new authorization token.

### Making API calls with out Authorization token
Once we've got an authorization token, we'll need to insert it into the Headers for every API request we make. This is easily done in PHP with file_get_contents() like so:

{% highlight php startinline=true %}
function callSpotify($url) {
  $headerStr = "Authorization: Bearer ". Session::get('access_token') ."\r\n";
  // Create a stream
  $opts = array(
    'http'=>array(
      'method'=>"GET",
      'header'=>$headerStr,
    )
  );

  $context = stream_context_create($opts);

  // Open the file using the HTTP headers set above
  return file_get_contents($url, false, $context);
}
{% endhighlight %}

The last simple breakdown of what we're doing:

1. We create a custom Header string with the syntax specified in the Spotify API reference. We need to insert the access_token straight into the Header. I stored this in the Session for easy access because it will be different for every user.
2. I construct a special stream_context with the options setting the `header` equal to my header string.
3. I send off the request using my special context and return the response (which will be in JSON!)

We're done! Have more questions? [Email me!](mailto:brent@walther.io "Email me!")
