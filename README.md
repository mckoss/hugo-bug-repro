# hugo-bug-repro

Simple repro case for Hugo bug:

https://github.com/gohugoio/hugo/issues/9250

# Steps to reproduce this error

```
$ hugo
Start building sites … 
hugo v0.89.4-AB01BA6E+extended linux/amd64 BuildDate=2021-11-17T08:24:09Z VendorInfo=gohugoio
Error: Error building site:
  failed to render pages: render of "page" failed:
  execute of template failed: template: people/single.html:6:9:
  executing "main" at <partial "person-box" .>:
  error calling partial: "/home/mckoss/src/hugo-bug-repro/themes/simple-theme/layouts/partials/person-box.html:10:11":
  execute of template failed:
  template: partials/person-box.html:10:11: executing "partials/person-box.html" at <$image.Permalink>:
  nil pointer evaluating resource.Resource.Permalink
```

The file where the error appears is a partial [person-box.html](themes/simple-theme/layouts/partials/person-box.html).

To see the affect of adding a ```{{ with }}``` statement  change line 3 of that file from

```
{{ $exhibitBug := true }}
```

to

```
{{ $exhibitBug := false }}
```

The results of running Hugo are now:

```
Start building sites … 
hugo v0.89.4-AB01BA6E+extended linux/amd64 BuildDate=2021-11-17T08:24:09Z VendorInfo=gohugoio

                   | EN  
-------------------+-----
  Pages            |  9  
  Paginator pages  |  0  
  Non-page files   |  1  
  Static files     |  0  
  Processed images |  0  
  Aliases          |  0  
  Sitemaps         |  1  
  Cleaned          |  0  

Total in 49 ms
```

Which shows that the value of ```$image``` (from a .Resources.GetMatch) call is somehow being
"sanctified" by being run through a ```{{ with }}``` statement.

**Expect**: Since the ```{{with}}``` evaluates $image as non-nil anyway, it should not be required
to get the .Permalink method from it.
