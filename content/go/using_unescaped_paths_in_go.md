+++
topics = ["Go","golang","programming"]
tags = ["go", "golang"]
date = "2017-06-01T09:29:32+01:00"
title = "Using Unescaped Paths in Go"

+++

I'm currently writing an API wrapper for Cisco's ACI and ran into a bug where I
was getting a 400 Bad Request and couldn't understand why. I had a successful
http GET with [Postman](https://www.getpostman.com/),
[httpie](https://httpie.org/), and also with the other Go API wrapper for ACI,
[acigo](https://github.com/udhos/acigo). As far as I could see, my http request
was the same as the one used by these other methods. I left it unsolved last
night; I'd had red wine, watched the Election debate, my girlfriend was home
from a 14 hour shift in an NHS A&E department and as ever my broken code was
unimportant in comparison to her day.

It was only this morning before the kids woke up, armed with coffee that I
noticed the only difference between my http request and the others.

My code is fetching a list of
[VRFs](https://en.wikipedia.org/wiki/Virtual_routing_and_forwarding) from the
APIC server, Cisco ACI's controller that we interact with via the REST API. To
do this we use a GET request to this endpoint:

```
https://{{APIC_URL}}/api/node/mo/uni/tn-{{TENANT_NAME}}.json?query-target=children&target-subtree-class=fvCtx
```

As
[recommended](https://medium.com/@marcus.olsson/writing-a-go-client-for-your-restful-api-c193a2f4998c)
I've extracted the request into it's own Client method:

```go
func (c *Client) newRequest(method string, path string, body interface{}) (*http.Request, error) {
	u := url.URL{Scheme: c.Host.Scheme, Host: c.Host.Host, Path: path}

	var buf io.ReadWriter
	if body != nil {
		buf = new(bytes.Buffer)
		err := json.NewEncoder(buf).Encode(body)
		if err != nil {
			return nil, err
		}
	}

	req, err := http.NewRequest(method, u.String(), buf)
	if err != nil {
		return nil, fmt.Errorf("%s request to %s : %v", method, u.String(), err)
	}
	if c.Cookie != "" {
		req.Header.Set("Cookie", c.Cookie)
	}
	return req, nil
}
```

After comparing lots of print statements I realised the URL was being escaped to
this:

```
https://{{APIC_URL}}/api/node/mo/uni/tn-{{TENANT_NAME}}.json%3fquery-target=children&target-subtree-class=fvCtx
```

Notice the `?` between `json` & `query` is now `%3f`, its [ASCII hex
equivalent](http://www.december.com/html/spec/esccodes.html). We don't want this
escaped as it indicates the [query component of the
URL](https://www.ietf.org/rfc/rfc3986.txt):

> The query component is indicated by the first question mark ("?") character and
> terminated by a number sign ("#") character or by the end of the URI.

This is why I was getting a 400 Bad Request, the server didn't understand the
escaped URL. So we need to use the raw unescaped path in order to successfully
connect with the API endpoint. To do this we use the url package's
[Parse](https://golang.org/pkg/net/url/#Parse) method to create our URL rather
than do it by hand. Initially I changed this line `u := url.URL{Scheme:
c.Host.Scheme, Host: c.Host.Host, Path: path}` to this:

```go
u, err := url.Parse(path)
if err != nil {
	return nil, err
}
u.Scheme = c.Host.Scheme
u.Host = c.Host.Host
```

But then reading back through the [post mentioned
above](https://medium.com/@marcus.olsson/writing-a-go-client-for-your-restful-api-c193a2f4998c)
I noticed something I had overlooked earlier, the `ResolveReference` method,
which resolves the absolute URL for me, and changed my code to this:

```go
rel, err := &url.Parse(path)
if err != nil {
    return nil, err
}
u := c.Host.ResolveReference(rel)
```

I think I overlooked/deliberately ignored it initially because I didn't
understand what it was doing and so didn't understand it's relevance. A better
lesson learned through failure. Interestingly the code used in the above
mentioned post creates the same error, this code is unsuccessful:

```go
rel := &url.URL{Path: path}
u := c.Host.ResolveReference(rel)
```

This may not be the best way of dealing with this issue, I'll learn more as I
go. But for now, remember kids, parse your URLs. And vote.



