Source code for [Personal Blog](jasmineyuan.com).

Using Hexo framework.

## Add New Post
+ Add a post under `source/_posts`.
+ `hexo generate` and `hexo deploy` to deploy the added post.

## Theme Related
The theme `ghost-casper` will add a class `full-img` when the image natual size is too big, thus making the `width` attribute useless. To make own-defined width effective, add `class="non-full"` to the `img` element.
```html
<img src="http://omcdckn46.bkt.clouddn.com/rebase.png" class="non-full" width="600px">
```
