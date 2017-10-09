---
title: requests 如何编码 multipart/form-data
date: 2017-10-09 19:37:13
tags: [requests, multipart, form-data]
---

requests 是用 python 处理 HTTP 请求时毋庸置疑的第一选择，其中的一个原因就是它简洁的 API 接口。那么这些接口下面隐藏的是什么样的东东呢，今天我们来分析一下 requests 是如何处理 multipart/form-data 格式的请求的。<!--more-->

## multipart/form-data 编码格式

废话少说，我也懒得介绍 multipart/form-data 是啥了。先抓包看一下浏览器在 form 表单的 `enctype="multipart/form-data` 时，构造的 POST 请求时什么样。

分析一下，基本的格式长这样：
```
POST / HTTP/1.1[换行]
Host: localhost[换行]
Content-Type: multipart/form-data; boundary=boundary[换行]
[换行]
--boundary[换行]
Content-Disposition: form-data; name="参数名"[换行] 
[换行]
内容[换行]
--boundary[换行]
Content-Disposition: form-data; name="图片名"; filename="图片文件名"[换行]
Content-Type: 类型[换行]
[换行]
二进制内容[换行]
--boundary--
```

在 Content-Disposition 中，如果有 filename 参数，则 PHP（或其他后端框架）会将此参数解析到 `$_FILES` 中。

## requests 如何编码？
```py
r = requests.post(url, data={'data': 'd'}, files={'files': '1'})
```
当提供了 files 参数时，requests 也会将 data 做 multipart/form-data 编码，但是不添加 filename 参数，所以后端会解析到 \$\_POST 中，files 参数任然解析到 \$\_FILES 中。

```py
r = requests.port(url, files={'file': (None, 'file')})
```
`files={'file': (None, 'file')}` 这样将 files 参数写成二元组，第一项表示此参数的 filename 属性。这里手动置为 `None`，则表示不包含 filename 属性，所以后端会将此参数解析到 `$_POST` 中。
这种方式既保证了使用 multipart/form-data 编码，也不会被解析到 `$_FILES` 中，在编写 POC 时常用。

## requests 编码分析
编码部分的代码在 `requests.models.PreparedRequest.prepare_body()` 中，在 `PreparedRequests.prepare()` 方法中会调用 `.prepare_body()`，生成 HTTP 请求的 body 部分：
```py
    def prepare_body(self, data, files, json=None):
        """Prepares the given HTTP body data."""
        # Check if file, fo, generator, iterator.
        # If not, run through normal process.
        # Nottin' on you.
        body = None
        content_type = None
        # 如果只提供了 json 参数，则使用 application/json 编码
        if not data and json is not None:
            # urllib3 requires a bytes-like body. Python 2's json.dumps
            # provides this natively, but Python 3 gives a Unicode string.
            content_type = 'application/json'
            body = complexjson.dumps(json)
            if not isinstance(body, bytes):
                body = body.encode('utf-8')
        # data 是否为一个可迭代对象（是否使用 chunked 方式上传）
        is_stream = all([
            hasattr(data, '__iter__'),
            not isinstance(data, (basestring, list, tuple, collections.Mapping))
        ])
        try:
            length = super_len(data)
        except (TypeError, AttributeError, UnsupportedOperation):
            length = None
        # 使用 chunked 方式上传
        if is_stream:
            body = data
            if getattr(body, 'tell', None) is not None:
                # Record the current file position before reading.
                # This will allow us to rewind a file in the event
                # of a redirect.
                try:
                    self._body_position = body.tell()
                except (IOError, OSError):
                    # This differentiates from None, allowing us to catch
                    # a failed `tell()` later when trying to rewind the body
                    self._body_position = object()
            if files:
                raise NotImplementedError('Streamed bodies and files are mutually exclusive.')
            if length:
                self.headers['Content-Length'] = builtin_str(length)
            else:
                self.headers['Transfer-Encoding'] = 'chunked'
        else:
            # Multi-part file uploads.
            # 如果有 files 参数，则使用 _encode_files 方法，进行 multi-part 编码
            if files:
                (body, content_type) = self._encode_files(files, data)
            # 否则按照 urlencoded 方式编码 data 部分
            else:
                if data:
                    body = self._encode_params(data)
                    if isinstance(data, basestring) or hasattr(data, 'read'):
                        content_type = None
                    else:
                        content_type = 'application/x-www-form-urlencoded'
            self.prepare_content_length(body)
            # Add content-type if it wasn't explicitly provided.
            if content_type and ('content-type' not in self.headers):
                self.headers['Content-Type'] = content_type
        self.body = body
```

下面看一下 `_encode_files()` 和 `_encode_params()` 方法：

```py
class RequestEncodingMixin(object):
    ...
    @staticmethod
    def _encode_params(data):
        """Encode parameters in a piece of data.

        Will successfully encode parameters when passed as a dict or a list of
        2-tuples. Order is retained if data is a list of 2-tuples but arbitrary
        if parameters are supplied as a dict.
        """

        if isinstance(data, (str, bytes)):
            return data
        elif hasattr(data, 'read'):
            return data
        elif hasattr(data, '__iter__'):
            result = []
            for k, vs in to_key_val_list(data):
                if isinstance(vs, basestring) or not hasattr(vs, '__iter__'):
                    vs = [vs]
                for v in vs:
                    if v is not None:
                        result.append(
                            (k.encode('utf-8') if isinstance(k, str) else k,
                            v.encode('utf-8') if isinstance(v, str) else v))
            # 直接使用 urlencode 进行编码
            return urlencode(result, doseq=True)
        else:
            return data

    @staticmethod
    def _encode_files(files, data):
        """Build the body for a multipart/form-data request.

        Will successfully encode files when passed as a dict or a list of
        tuples. Order is retained if data is a list of tuples but arbitrary
        if parameters are supplied as a dict.
        The tuples may be 2-tuples (filename, fileobj), 3-tuples (filename, fileobj, contentype)
        or 4-tuples (filename, fileobj, contentype, custom_headers).
        """
        if (not files):
            raise ValueError("Files must be provided.")
        elif isinstance(data, basestring):
            raise ValueError("Data must not be a string.")

        new_fields = []
        # 全部转为 [(key, val)] 的形式，统一处理
        fields = to_key_val_list(data or {})
        files = to_key_val_list(files or {})

        for field, val in fields:
            # 从这里看 data 参数还接受 list of string?
            if isinstance(val, basestring) or not hasattr(val, '__iter__'):
                val = [val]
            for v in val:
                if v is not None:
                    # Don't call str() on bytestrings: in Py3 it all goes wrong.
                    if not isinstance(v, bytes):
                        v = str(v)

                    new_fields.append(
                        (field.decode('utf-8') if isinstance(field, bytes) else field,
                        v.encode('utf-8') if isinstance(v, str) else v))

        for (k, v) in files:
            # support for explicit filename
            ft = None
            fh = None
            # 分别处理各种大小的元组
            if isinstance(v, (tuple, list)):
                if len(v) == 2:
                    fn, fp = v
                elif len(v) == 3:
                    fn, fp, ft = v
                else:
                    fn, fp, ft, fh = v
            else:
                # 如果不是元组形式，则从 file_obj 中猜测文件名；或者直接使用内容字符串作为文件名
                fn = guess_filename(v) or k
                fp = v

            if isinstance(fp, (str, bytes, bytearray)):
                fdata = fp
            else:
                fdata = fp.read()
            # 使用的 urllib3.fields.RequestFile
            rf = RequestField(name=k, data=fdata, filename=fn, headers=fh)
            rf.make_multipart(content_type=ft)
            new_fields.append(rf)
        # 使用 urllib3.filepost 中的方法
        body, content_type = encode_multipart_formdata(new_fields)

        return body, content_type
```

可以看到，在处理 multipart 的编码时，requests 直接使用 urllib3 提供的方法，那我们就看一下 urllib3 是如何实现的。

```py
def encode_multipart_formdata(fields, boundary=None):
    """
    Encode a dictionary of ``fields`` using the multipart/form-data MIME format.

    :param fields:
        Dictionary of fields or list of (key, :class:`~urllib3.fields.RequestField`).

    :param boundary:
        If not specified, then a random boundary will be generated using
        :func:`mimetools.choose_boundary`.
    """
    body = BytesIO()
    if boundary is None:
        boundary = choose_boundary()

    for field in iter_field_objects(fields):
        body.write(b('--%s\r\n' % (boundary)))

        writer(body).write(field.render_headers())
        data = field.data

        if isinstance(data, int):
            data = str(data)  # Backwards compatibility

        if isinstance(data, six.text_type):
            writer(body).write(data)
        else:
            body.write(data)

        body.write(b'\r\n')

    body.write(b('--%s--\r\n' % (boundary)))

    content_type = str('multipart/form-data; boundary=%s' % boundary)

    return body.getvalue(), content_type

def iter_field_objects(fields):
    """
    Iterate over fields.

    Supports list of (k, v) tuples and dicts, and lists of
    :class:`~urllib3.fields.RequestField`.

    """
    if isinstance(fields, dict):
        i = six.iteritems(fields)
    else:
        i = iter(fields)

    for field in i:
        if isinstance(field, RequestField):
            yield field
        else:
            yield RequestField.from_tuples(*field)
```

代码比较直白，就不多说啦~

## 参考链接
https://stackoverflow.com/questions/12385179/how-to-send-a-multipart-form-data-with-requests-in-python