Django Wools
============

Django tools from WITH.

That's a collection of things that we at [WITH](https://with-madrid.com/) got
tired of copy/pasting in every project.

## Install

```
pip install django_wools
```

## Included Wools

### Storage

#### `django_wools.storage.GzipManifestStaticFilesStorage`

That's a sub-class of the 
[ManifestStaticFilesStorage](https://docs.djangoproject.com/en/3.0/ref/contrib/staticfiles/#manifeststaticfilesstorage)
but that makes sure that along with all the files comes a `.gz` version which
is easy to pick up for nginx (or other static files server).

### Middlewares

#### `django_wools.middlewares.NowMiddleware`

Suppose that you have a content that is available up until a given date. When
the date is passed then everything related to this content expires. However,
in order to do this, you're probably going to make several request, possibly in
loosely connected parts of your code. In those cases, when looking at the time,
the clock will show different value as the time passes between calls. It means
that you could very well end up with one half of your code considering that the
object is still valid but the other half that it expired.

In order to prevent this, the simplest is to consider that the time is fixed
and that the code executes instantly at the moment of the request. The goal
of this middleware is to save the current time at each request and then to
provide an easy way to get the current time through the request.

If the middleware is activated, you should be able to get the time like this:

```python
from time import sleep
from django.shortcuts import render

def my_view(request):
    print(f"Now is {request.now()}")
    sleep(42)
    print(f"Now is still {request.now()}")

    return render(request, "something.html", {"now": request.now()})
```

#### `django_wools.middlewares.SlowMiddleware`

When developing a SPA or hybrid app, you want to make sure that the app is
structurally ready to handle load times from the server, even if the connection
is a bit shaky. Also, you want to make sure that not too many requests are
sent.

In order for you to fully realize how slow your website is going to be on a bad
connection, th e SlowMiddleware will automatically add a delay before replying
to each request.

Add this to your `settings.py`

```python
MIDDLEWARE = [
    # ...
    "django_wools.middlewares.SlowMiddleware",
]

SLOW_MIDDLEWARE_LATENCY = 1 if DEBUG else 0
```

By doing this, your requests will be added a 1s delay if the `DEBUG` mode is
enabled.

### Database

#### `django_wools.db.require_lock`

Provides a way to explicitly generate a PostgreSQL lock on a table.

By example:

```python
from django.db.transaction import atomic
from django_wools.db import require_lock

from my_app.models import MyModel


@atomic
@require_lock(MyModel, 'ACCESS EXCLUSIVE')
def myview(request):
    # do stuff here
```

### Wagtail Images

Several Wagtail-specific tags are provided to deal with images and more
specifically responsive images. To use it:

1. Add `django_wools` to `INSTALLED_APPS`
2. Import `wools_for_wt_images` from your template

```
{% load wools_for_wt_images %}
```

Some specific settings can be set:

- `WOOLS_MAX_PIXEL_RATIO` _(default: `3`)_ &mdash; Highest device pixel ratio
  to support.
- `WOOLS_INCREMENT_STEP_PERCENT` _(default: `(sqrt(2) - 1) * 100`)_ &mdash; The
  percentage of increase from the base density to the next one. The default
  values will generate `x1`, `x2` and `x4` with intermediate values that are
  `x1.4142` and `x2.8284`.

All the tags will be default generate WebP images with a PNG fallback. The
fallback can be changed to JPEG and the main format has to be WebP.

#### `image_fixed_size`

This tag will generate a `<picture>` tag for an image whose size in pixels you
known in advance. That picture size will be enforced in the HTML code with
inline properties.

Usage:

```
{% image_fixed_size max-500x500 %}
```

The arguments to this tag, in order, are:

- `image` &mdash; The Wagtail-compatible image itself
- `spec` &mdash; A spec like you would give to Wagtail (`fill-500x500`,
   `max-500x500`, etc)
- `fallback_format` &mdash; The format to fallback in case the browser doesn't
  support WebP. Can either be `"jpeg"` or `"png"`. Defaults to `"png"`.
- `lossless` &mdash; A boolean to enable losslessness of WebP. This does not
  affect the fallback format, so if you want a lossless fallback as well you'll
  need to use PNG.
