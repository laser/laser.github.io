# blog

## publishing

I've been using `pandoc` to generate these blog posts, and then adding a suffix
and prefix:

```
$ echo -e "$(cat header.snippet)\n$(cat post.md | pandoc -f markdown)\n$(cat footer.snippet)" > post.md.html
```

Then, update index.html to include a link to the new post.
