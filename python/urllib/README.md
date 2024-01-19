## urlib.request
```
from urllib.request import urlopen

myURL = urlopen("http://www.google.com/")
print(myURL.read())
```


## urllib.parse
```
from urllib.parse import urlparse 
parsedUrl = urlparse('https://www.educative.io/track/python-for-programmers') 
print(parsedUrl)
```

## urllib.error
exceptions, or errors, are classified as follows:

- URL Error, which is raised when the URL is incorrect, or when there is an internet connectivity issue.
- HTTP Error, which is raised because of HTTP errors such as 404 (request not found) and 403 (request forbidden). The following code snippet shows the usage of urlib.error:

```
from urllib.request import urlopen, HTTPError, URLError

try:
    myURL = urlopen("http://ww.educative.xyz/")
except HTTPError as e:
    print('HTTP Error code: ', e.code)
except URLError as e:
    print('URL Error: ', e.reason)
else:
    print('No Error.')
```