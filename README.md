# `python-xcall`

A Python [x-callback-url](http://x-callback-url.com) client for bi-directional communication with `x-callback-url` enabled macOS applications. `python-xcall` supports callbacks from appplications. It wraps the handy
[xcall](https://github.com/martinfinke/xcall) command line tool.

> [!NOTE]  
> Compared to [upstream](https://github.com/robwalton/python-xcall), [@cdzombak's fork](https://github.com/cdzombak/python-xcall) adds Python 3 compatbility.

## Software compatability

- macOS
- Python 3
- Uses [xcall](https://github.com/martinfinke/xcall) (included).
- Needs `pytest` for testing

## Installation

Check it out:

```text
$ git clone https://github.com/robwalton/python-xcall.git
Cloning into 'python-xcall'...
```

## Basic use

Call a scheme (ulysses) with an action (get-version):

```python
>>> import xcall
>>> xcall.xcall('ulysses', 'get-version')
{u'apiVersion': u'2', u'buildNumber': u'33542'}
```

An x-success reply will be utf-8 un-encoded, then url unquoted, and then un-marshaled using json into Python objects and returned.

A dictionary of action parameters can also be provided (each value is utf-8 encoded and then url quoted before sending):

```python
>>> xcall.xcall('ulysses', 'new-sheet', {'text':'My new sheet', 'index':'2'})
```

If the application calls back with an x-error, an `XCallbackError` will be raised:

```python
>>> xcall.xcall('ulysses', 'an-invalid-action')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
...
XCallbackError: x-error callback: '{
  "errorMessage" : "Invalid Action",
  "errorCode" : "100"
}
' (in response to url: 'ulysses://x-callback-url/an-invalid-action')
```

## More control

For more control create an instance of `xcall.XCallClient`, specifying the scheme to use, whether responses should be un-marshaled using json, and an x-error handler. For example:

```python
class UlyssesError(XCallbackError):
    pass

def ulysses_xerror_handler(xerror, requested_url):
    error_message = eval(xerror)['errorMessage']
    error_code = eval(xerror)['errorCode']
    raise UlyssesError(
        ("%(error_message)s. Code=%(error_code)s. "
         "In response to sending the url '%(requested_url)s'") % locals())

ulysses_client = XCallClient(
    'ulysses', on_xerror_handler=ulysses_xerror_handler,  json_decode_success=True)

```

Make calls using:

```python
>>> ulysses_client.xcall('get-version')
```

or just:

```python
>>> ulysses_client('get-version')
```

## Logging

As logger output just goes directly to the terminal, it is disabled by default. To enable more verbose logging use:

```python
>>> import xcall
>>> xcall.enable_verbose_logging()
```

## Thread/process safety

> [!WARNING]  
> Call to this module are **probably** not thread/process safe.

An attempt is made to ensure that `xcall` is not already running, but there is 20-30ms window in which multiple calls to this module will result in multiple xcall processes running and the chance of replies being mixed up.

## Testing

Running the tests requires the `pytest` package. Some optional integration tests currently require [Ulysses](https://ulyssesapp.com). Code your access-token into the top of `test_calls.py`. Obtain the access token string by removing the `@skip` marker from `test_authorise()` in `test_calls.py` and running the tests.

From the root package folder call:

```text
$ pytest
...
```

## Licensing & Thanks

The code and the documentation are released under the [MIT](https://opensource.org/license/mit) and [Creative Commons Attribution-NonCommercial](https://creativecommons.org/licenses/by-nc/4.0/) licences, respectively.

Thanks to:

- [Martin Finke](https://github.com/martinfinke) for his handy [xcall](https://github.com/martinfinke/xcall) application
- [Dean Jackson](https://github.com/deanishe) for suggestions
- [@robwalton](https://github.com/robwalton) for the upstream, Python 2.7 version of this library
