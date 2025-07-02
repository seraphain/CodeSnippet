# CodeSnippet

```python
from types import FunctionType, LambdaType, MethodType
from typing import Any, Callable, Optional, Union

def aspect_function(target: Union[FunctionType, MethodType],
                    before: Optional[Callable[[tuple[Any, ...], dict[str, Any]], Any]] = None,
                    after_returning: Optional[Callable[[Any, Any, tuple[Any, ...], dict[str, Any]], Any]] = None,
                    after_throwing: Optional[Callable[[Any, Any, Any, BaseException, tuple[Any, ...], dict[str, Any]], Any]] = None,
                    after: Optional[Callable[[Any, Any, Any, BaseException, tuple[Any, ...], dict[str, Any]], Any]] = None):
    @staticmethod
    def wrapper(*args, **kwargs):
        before_result = None
        result = None
        after_result = None
        exception = None
        if before:
            before_result = before(*args, **kwargs)
        try:
            if isinstance(target, FunctionType):
                result = target(*args, **kwargs)
            elif isinstance(target, MethodType):
                result = target(*args[1:], **kwargs)
            else:
                raise TypeError(f'Unknown target type: {type(target)}')
            if after_returning:
                after_result = after_returning(before_result, result, *args, **kwargs)
            return result
        except BaseException as e:
            exception = e
            if after_throwing:
                after_throwing(before_result, result, after_result, exception, *args, **kwargs)
            raise exception
        finally:
            if after:
                after(before_result, result, after_result, exception, *args, **kwargs)
    if isinstance(target, FunctionType):
        module = __import__(target.__module__)
        if '.' in target.__qualname__:
            clazz = getattr(module, target.__qualname__[0:target.__qualname__.rindex('.')])
            setattr(clazz, target.__name__, wrapper)
        else:
            setattr(module, target.__name__, wrapper)
    elif isinstance(target, MethodType):
        setattr(type(target.__self__), target.__name__, wrapper)
    else:
        raise TypeError(f'Unknown target type: {type(target)}')
    return wrapper
```

```python
import aspect
import requests
import time
from b import B

def before(*args, **kwargs):
    return time.perf_counter()

def after(before_result, result, after_result, exception, *args, **kwargs):
    print(f'method: {args[0]}')
    #print(f'url: {args[1]}')
    if exception:
        print(f'exception: {exception}')
    print(f'time_cost: {time.perf_counter() - before_result}')

# aspect.aspect_function(requests.request, before=before, after=after)
#
# rsp = requests.request('get', 'http://www.baidu.com')
# print(rsp.text)
#
# class A():
#     def test(self, url):
#         print(f'test: {url}')
#
# aspect.aspect_function(A.test, before=before, after=after)
#
# A().test('https://aaa.com')
#
# aspect.aspect_function(B.test1, before=before, after=after)
# B().test1('https://bbb.com')

aspect.aspect_function(B.test2, before=before, after=after)
B().test2('https://bbb.com')

```
