Personal Blog to discuss programming projects and other topics.

To run locally simply copy `_config.yml.example` to `_config.yml` and then run 
the following command to build then serve the site.

```
docker run --rm --volume="$PWD:/srv/jekyll" -it jekyll/jekyll:$JEKYLL_VERSION jekyll build; docker run --volume="$PWD:/srv/jekyll" -p 3000:4000 -it jekyll/jekyll:$JEKYLL_VERSION jekyll serve --watch
```


