

## Install Hugo

```
$ brew install hugo

$ hugo version
```


## Basic Usage

### CLI Help

```
$ hugo help
```

### The `hugo` Command

The command hugo renders your site into public/ dir and is ready to be deployed to your web server

```
$ hugo

                   | EN  
+------------------+----+
  Pages            | 25  
  Paginator pages  |  2  
  Non-page files   |  0  
  Static files     | 44  
  Processed images |  0  
  Aliases          |  2  
  Sitemaps         |  1  
  Cleaned          |  0  

Total in 177 ms
```

### Add Some Content

```
$ hugo new post/my-first-post.md
```

## Serve The Contents

### Start the Hugo Server

```
$ hugo server -D
```
Here drafts are enabled.

### Start a local nginx

```
docker run --rm --name brilliun-com -v `pwd`/public:/usr/share/nginx/html:ro -p 8080:80 -d nginx
```
Need to add `127.0.0.1  brilliun.com` in /etc/hosts