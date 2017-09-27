# Errors

The Eventum API uses the following HTTP error codes:


Error Code | Meaning
---------- | -------
400 | Bad Request -- Your request sucks - fix it!
401 | Unauthorized -- Your API key is wrong.
403 | Forbidden -- For administrators' eyes only.
404 | Not Found -- Nothing here.
405 | Method Not Allowed -- You tried to access the API with an invalid method.
406 | Not Acceptable -- You requested a format that isn't JSON.
410 | Gone -- The resource requested has been removed from our servers.
418 | I'm a teapot.
429 | Too Many Requests -- Wooooah, slow down! See rate limiting above!
500 | Internal Server Error -- We had a problem with our server. Try again later.
503 | Service Unavailable -- We're temporarily offline for maintenance. Please try again later.
