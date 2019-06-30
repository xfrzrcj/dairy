# **Scrapy**
## 1、下载图片自定义图片名
  最近用爬虫抓取数据需要抓取图片，一开始选用默认的图片pipeline,即settings.py里配置
```python
  ITEM_PIPELINES = {'scrapy.pipelines.images.ImagesPipeline': 1}
```
。但是图片名却无法自定义，找到ImagesPipeline，发现默认图片名代码为
```python
def file_path(self, request, response=None, info=None):
    ## start of deprecation warning block (can be removed in the future)
    def _warn():
        from scrapy.exceptions import ScrapyDeprecationWarning
        import warnings
        warnings.warn('ImagesPipeline.image_key(url) and file_key(url) methods are deprecated, '
                      'please use file_path(request, response=None, info=None) instead',
                      category=ScrapyDeprecationWarning, stacklevel=1)

    # check if called from image_key or file_key with url as first argument
    if not isinstance(request, Request):
        _warn()
        url = request
    else:
        url = request.url

    # detect if file_key() or image_key() methods have been overridden
    if not hasattr(self.file_key, '_base'):
        _warn()
        return self.file_key(url)
    elif not hasattr(self.image_key, '_base'):
        _warn()
        return self.image_key(url)
    ## end of deprecation warning block

    image_guid = hashlib.sha1(to_bytes(url)).hexdigest()  # change to request.url after deprecation
    return 'full/%s.jpg' % (image_guid)
```
即`hashlib.sha1(to_bytes(url)).hexdigest()`，将urlhash取值。由于图片需要与一些信息关联，所以有3种方案解决。
- 1、在信息中直接记录图片url的hash
- 2、自定义ImagesPipeline，在scrapy将自定义的图片名传进去。
- 3、自定义ImagesPipeline，在ImagesPipeline中自己生成图片名

方案1暂且不说，方案3可以参考[这里](https://blog.csdn.net/qq_31235811/article/details/88917771)，方案2个人觉得比较有扩展性。可以类似其图片存放地址的`IMAGES_URLS_FIELD`处理方法，在settings中配置一个`IMAGES_NAME_FIELD`,然后pipeline读取items中对应字段作为图片名，从而在scrapy中控制图片名。
具体代码如下：
- 1、scrapy中对ImagenetItem添加name属性
```python
item = ImagenetItem()
url = imgs[i]
item['src'] = [url]
item['name'] = url[-35: -3]
```
- 2、在items.py中定义图片item.
```python
class ImagenetItem(scrapy.Item):
    # define the fields for your item here like:
    # name = scrapy.Field()
    src = scrapy.Field()
    name = scrapy.Field()
    pass
```
- 3、在pipeline.py定义一个pipeline继承crapy.pipelines.images.ImagesPipeline
```python
class PicPipeline(ImagesPipeline):
    IMAGES_NAME_FIELD = ''

    def __init__(self, store_uri, download_func=None, settings=None):
        super(ImagesPipeline, self).__init__(store_uri, settings=settings,
                                             download_func=download_func)

        if isinstance(settings, dict) or settings is None:
            settings = Settings(settings)

        resolve = functools.partial(self._key_for_pipe,
                                    base_class_name="PicPipeline",
                                    settings=settings)
        self.expires = settings.getint(
            resolve("IMAGES_EXPIRES"), self.EXPIRES
        )

        if not hasattr(self, "IMAGES_RESULT_FIELD"):
            self.IMAGES_RESULT_FIELD = self.DEFAULT_IMAGES_RESULT_FIELD
        if not hasattr(self, "IMAGES_URLS_FIELD"):
            self.IMAGES_URLS_FIELD = self.DEFAULT_IMAGES_URLS_FIELD

        self.images_urls_field = settings.get(
            resolve('IMAGES_URLS_FIELD'),
            self.IMAGES_URLS_FIELD
        )
        self.images_result_field = settings.get(
            resolve('IMAGES_RESULT_FIELD'),
            self.IMAGES_RESULT_FIELD
        )
        self.min_width = settings.getint(
            resolve('IMAGES_MIN_WIDTH'), self.MIN_WIDTH
        )
        self.min_height = settings.getint(
            resolve('IMAGES_MIN_HEIGHT'), self.MIN_HEIGHT
        )
        self.thumbs = settings.get(
            resolve('IMAGES_THUMBS'), self.THUMBS
        )
        self.images_name_field = settings.get(
            resolve('IMAGES_NAME_FIELD'),
            self.IMAGES_NAME_FIELD
        )

    def get_media_requests(self, item, info):
        return [Request(x, meta={'file_name': item.get(self.images_name_field, '')}) for x in item.get(self.images_urls_field, [])]

    def file_path(self, request, response=None, info=None):
        ## start of deprecation warning block (can be removed in the future)
        def _warn():
            from scrapy.exceptions import ScrapyDeprecationWarning
            import warnings
            warnings.warn('ImagesPipeline.image_key(url) and file_key(url) methods are deprecated, '
                          'please use file_path(request, response=None, info=None) instead',
                          category=ScrapyDeprecationWarning, stacklevel=1)

        # check if called from image_key or file_key with url as first argument
        if not isinstance(request, Request):
            _warn()
            url = request
        else:
            url = request.url

        image_guid = request.meta['file_name']  # change to request.url after deprecation
        return 'full/%s.jpg' % (image_guid)
```
基本是在原代码上改动，重点在于定义了images_name_field字段，在init中初始化值，在get_media_requests方法中将名字传到了request的meta属性中，再在file_path方法中获取值即可。
- 4、在settings.py中配置我们刚才定义的pipeline。

```python
ITEM_PIPELINES = {
   'tutorial.pipelines.PicPipeline': 1,
}

IMAGES_STORE = 'imagesDownload'
IMAGES_URLS_FIELD = 'src'#定义url字段
IMAGES_NAME_FIELD = 'name'#定义图片名字字段与item对应
```
这样就能扩展IMAGES_NAME_FIELD字段作为图片名。
