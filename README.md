# A wrapper for the Qt Network Access API

This is just a wrapper around Qt's QNetworkAccessManager and friends. I use it in my Qt apps at https://flavio.tordini.org . It allows me to add missing functionality as needed, e.g.:

- Throttling (as required by many web APIs nowadays)
- Read timeouts (don't let your requests get stuck forever)
- Automatic retries
- User agent and request header defaults
- Partial requests
- Redirection support (now supported by Qt >= 5.6)

It has a simpler, higher-level API that I find easier to work with. The design emerged naturally in years of practical use.

## Examples

A basic example:

```
auto reply = Http::instance().get("https://google.com/");
connect(reply, &HttpReply::finished, this, [](auto reply) {
    if (reply.isSuccessful()) {
        qDebug() << "Feel the bytes!" << reply.body();
    } else {
        qDebug() << "Something's wrong here" << reply.statusCode() << reply.reasonPhrase();
    }
}
```

This is a real-world example of building a Http object suitable to a web service. It throttles requests, uses a custom user agent and caches results:

```
Http &myHttp() {
    static Http *http = [] {
        Http *http = new Http;
        http->addRequestHeader("User-Agent", userAgent());

        ThrottledHttp *throttledHttp = new ThrottledHttp(*http);
        throttledHttp->setMilliseconds(1000);

        CachedHttp *cachedHttp = new CachedHttp(*throttledHttp, "mycache");
        cachedHttp->setMaxSeconds(86400 * 30);

        return cachedHttp;
    }();
    return *http;
}
```

If the full power (and complexity) of QNetworkReply is needed you can always fallback to it:

```
HttpRequest req;
req.url = "https://flavio.tordini.org/";
QNetworkReply *reply = Http::instance().networkReply(req);
// Use QNetworkReply as needed...
```

## Integration

You can use this library as a git submodule. For example, add it to your project inside a lib subdirectory:

```
git submodule add -b master https://github.com/flaviotordini/http lib/http
```

Then you can update your git submodules like this:

```
git submodule update --init --recursive --remote
```

To integrate the library in your qmake based project just add this to your .pro file:

```
include(lib/http/http.pri)
```


## License

You can use this library under the MIT license and at your own risk. If you do, you're welcome contributing your changes and fixes.

Cheers,

Flavio
