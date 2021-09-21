**Steps to import Postman JSON**

This Postman collection was export using Postman, so will be easy to import. I'm on version 9.0, although I doubt the version matters.

In Postman, use File, Import. Then drag in the JSON file. It will import as a new collection called 'Discogs'.

The process to be able to make authenticated calls is complicated and that's what this readme is mostly about. You can make calls without full OAuth authentication, I also mention that way.

**Steps to use Discog API**

You need a consumer key, consumer secret + username which are set up when you register an app with Discogs.

Anywhere you see the following variable placeholders in the below instructions, replace with YOUR variables (replace the brackets too)
<consumerKey>
<consumerSecret>
<username>

**Pre-Step 1**

**Some basic Discogs Urls you'll need**
Create application / user access token creation:
https://www.discogs.com/settings/developers
Authentication info FAQ:
https://www.discogs.com/developers/#page:authentication

Request Token URL https://api.discogs.com/oauth/request_token
Authorize URL https://www.discogs.com/oauth/authorize
Access Token URL https://api.discogs.com/oauth/access_token

A. Make sure you can connect to the Discogs API without keys first, as a test. To do that, run the "Test Rick Request (no keys)" request

These are your final **environment variables** (some you have at start, some you generate IN the process, but in the end you should have all these (I do anyways):
token
consumerKey
consumerSecret
oauth_token
oauth_token_secret
oauth_verifier
DEUserAgent (this is optional)
access_token
access_token_secret
release_id (this is optional)

**Pre-Step 2**

You can run the "Madlib Pure Https Get (no headers)" request with just the consumer secret and key.

You can also run the "Release by ID (Get, no headers)" with just the secret/key I believe.

**Steps to get token** (a more complicated process than it needs to be)

Note: These are notes to remind myself how to do it, so use these in conjunction with the offical documentation

1.  (already done) Application setup (assume only once!)
    Consumer Key: <consumerKey>
    Consumer Secret : VgUTccsTwIKxRtCvsZKPOcEATrZHHhfD <consumerSecret>

2.  Run Request Token Step, paste return values here:
    oauth_token=LYSaPdWMjNyVJELBJbmpaJaVPhJcvGVsdDeXTinW&
    oauth_token_secret=ArzzfJNfDQcJTRxorXSheYPJSvMXgnGwgBHzbXKu
    \
    Issue is that now the 15 minute clock starts ticking!! ...

Update TWO environment variables: oauth_token and oauth_token_secret

3.  Authorization in browser

Fill in the token here and paste/verify in browser:
https://discogs.com/oauth/authorize?oauth_token=LYSaPdWMjNyVJELBJbmpaJaVPhJcvGVsdDeXTinW&

Paste returned code here:
**Application code**: nhJIEPZkbh This is the **oauth_verifier** (I believe)

update THIRD oath_verifier environment variable

4.  Validate values in "Access Token" Postman action look right against this page (step 4):
    https://www.discogs.com/developers#page:authentication

IMPORTANT: **oath_verifier should be NOT be in the GET URL**, it should be in auth tab. **Click on 'Code'** and make sure that call looks like the documentation i.e. like this:
var config = {
method: 'get',
url: 'https://api.discogs.com/oauth/identity',
headers: {
'User-Agent': 'DE2PostmanRuntime/7.30.1',
'Authorization': 'OAuth oauth_consumer_key="<consumerKey>",
oauth_token="ZNBjzlystHtLhpnbHLyhTWBTWYhEpmdvKhJsuFMN",
oauth_signature_method="PLAINTEXT",
oauth_timestamp="1598215259",
oauth_nonce="MC9iRGP3K07",
oauth_version="1.0", oauth_signature="<consumerSecret>%26WOiAbhVPLrpjrdhrSBnPMoxbqVjMNrtuSFPomczY"'
}
};

The oauth_signature in authorization header should like this?
"<consumerSecret>%26&MXpfhKfSoJFEkMdyGijdQHbRtAsLARRrUwgrThDH"

BAD RESPONSE:
Invalid signature. Expected signature base string (error was the ampersand AND %26 were BOTH being sent, one by me and one automatic):
VgUTccsTwIKxRtCvsZKPOcEATrZHHhfD&iPvVvANfXqBhWFAxXimWTFgHSIPzUCYUSIFGLOuQ

GOOD RESPONSE
oauth_token=ZNBjzlystHtLhpnbHLyhTWBTWYhEpmdvKhJsuFMN&oauth_token_secret=WOiAbhVPLrpjrdhrSBnPMoxbqVjMNrtuSFPomczY

THE TRICK IS: The & is added automatically, but it's %26 NOT % !!!! VERY hard to spot that.

"Encode the parameters in the Authorization header" is on

**NOTES**
**ampersand!!** Note this is done AUTOMATICALLY, look for a %26.
"your oauth_signature to be your consumer secret followed by an ampersand (&)." ALSO, Technically authorization data should be set to request header BUT & is only appended to consumer secret if you have it as "request body / request URL"
**oauth_verifier!!** oauth_verifier works if you add it as a param (doesn't need to be in authentication block). known Postman issue. "user_verifier" is got in Stage 3 "and you can capture their verifier from the response. The verifier will be used to generate the access token in the next step"...

5.  Validate using "Oauth Identity" action in Postman

Update the access token here and AC:
oauth_token=MmUKxxdPAoVgKDVCDGXfLAoxyoTWMBTcqtkStDlX&oauth_token_secret=NRxvzMQOSaeTdFZDegBpZsnwzdjYHTaghoIKkGLA
NOTE: Do Not use the & in environment variables, it actually means that as part of the GET URL

NOTE, I'm called these access_token and access_token_secret even though it's the same field, but the values will be different, therefore different environment variables makes sense for clarity.
YOU MUST CLICK SAVE IN ENVIRONMENT VARIABLES!!

GOOD RESPONSE:
{
"username": "<username>",
"resource_url": "https://api.discogs.com/users/<username>",
"consumer_name": "LabelReleases",
"id": 5819721
}

---
