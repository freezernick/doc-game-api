# Constructing Requests

## Single API Requests

An example call to the API might look something like this:

```
https://api.gamejolt.com/api/game/v1_2/data-store/?game_id=32&key=test&signature=912ec803b2ce49e4a541068d495ab570
```

The above would return data stored in the system with the key of "test" for the game with the ID of
"32".

It may look a little daunting at first, so let's dissect it.

### Basic URL Construction

The first part is the URL of the Game API:

```
https://api.gamejolt.com/api/game/
```

Then comes the API version. The current version number is **1_2**. This is to make sure old code
still works, even if we change how the API works in the future.

```
https://api.gamejolt.com/api/game/v1_2/
```

The "data-store" part is the URL path that we use to retrieve an item from the data store.

```
https://api.gamejolt.com/api/game/v1_2/data-store/
```

(Look [here](https://gamejolt.com/game-api/doc/data-store) for more information about the data
store.)

Now we have to pass in some variables. This is done as simple GET query parameters. Note that you
can also send the data through a POST method.

The variables required for each request can be found on their respective pages. Notice the
"signature" parameter - this is required in every request to the system.

All variables except the signature value need to get URL encoded before passing them into the URL.

```
https://api.gamejolt.com/api/game/v1_2/data-store/?game_id=32&key=test&signature=912ec803b2ce49e4a541068d495ab570
```

### Adding a Signature

Notice the "signature" parameter in the URL call above. The signature variable is a way to verify
that the URL call came from you and not someone trying to crack the system.

First you need to form the URL that you want to query. For example, getting trophies for your game
would be:

```
https://api.gamejolt.com/api/game/v1_2/trophies/?game_id=32&username=CROS&user_token=123456&achieved=true
```

Next, add the Private Key of your game (found under "Manage Achievements" on the game dashboard) to
the end of the URL like so:

```
https://api.gamejolt.com/api/game/v1_2/trophies/?game_id=32&username=CROS&user_token=123456&achieved=truemy_awesome_private_key
```

Pump this string through either `MD5` or `SHA-1` and you have successfully generated a signature.

```
fc3e8e0ea54544b3551058f0cf524303
```

To finish up, add the signature to the original request URL.

```
https://api.gamejolt.com/api/game/v1_2/trophies/?game_id=32&username=CROS&user_token=123456&achieved=true&signature=fc3e8e0ea54544b3551058f0cf524303
```

### Using POST data

All endpoints support submitting the variables in the POST data instead of GET query parameters.

This is convenient for many libraries and allows you to send longer requests which is especially useful with [/batch/](https://gamejolt.com/game-api/doc/batch) requests.

The signature needs to be created differently when there is POST data.
We'll use a request to fetch the achieved trophies where the `achieved` and `user_token` variables are in the POST data as an example:

```
curl -XGET 'https://api.gamejolt.com/api/game/v1_2/trophies/?game_id=32&username=CROS' -d '
{
    "user_token": 123456,
    "achieved": "true"
}'
```

First, you take the GET part of the url, same as before:

```
https://api.gamejolt.com/api/game/v1_2/trophies/?game_id=32&username=CROS
```

Then, take the post data and join key/value pairs together without spaces in between in ascending alphabetical order by the key name.

So if we have `user_token = 123456` and `achieved = true`, the order of keys is `achieved` and then `user_token` so the resulting post data should look like this:

```
achievedtrueuser_token123456
```

Next, add the post data and the private key of your game to the end of the URL like so:

```
https://api.gamejolt.com/api/game/v1_2/trophies/?game_id=32&username=CROSachievedtrueuser_token123456my_awesome_private_key
```

Pump this string through either `MD5` or `SHA-1` and the signature is:

```
d1eefdaaa2cb35ee3ceac5ce11079c89
```

Lastly, add the signature to the original request URL and send it with the POST data:

```
https://api.gamejolt.com/api/game/v1_2/trophies/?game_id=32&username=CROS&signature=d1eefdaaa2cb35ee3ceac5ce11079c89
```

### Arrays in POST Data

Some endpoints may have arrays in the post data, like the [/batch](https://gamejolt.com/game-api/doc/batch) `requests[]` field. To make a signature for them, you would concatenate the key name with each one of the array values.

For example, when trying to make a signature for this:

```
curl -XGET 'https://api.gamejolt.com/api/game/v1_2/batch/?game_id=32&username=CROS' -d '
{
    requests: [
      'abc',
      'def',
      'ghi',
    ]
}'
```

The post data to sign would become `requestsabcrequestsdefrequestsghi`, so the full url to sign would be:

```
https://api.gamejolt.com/api/game/v1_2/batch/?game_id=32&username=CROSrequestsabcrequestsdefrequestsghimy_awesome_private_key
```

To which the signature is:

```
d1387db85719d80afae538c5d5bca040
```
