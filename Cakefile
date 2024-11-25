require "sweetbread"

task "start", "Build, watch, and serve.", ()->
  invoke "build"
  invoke "watch"
  invoke "serve"

task "build", "Compile everything", ()->
  rm "docs"

  compile "static", "source/**/*.!(html|scss|coffee)", (path)->
    copy path, replace path, "source/": "docs/"

  template = read "source/pages/_template.html"
  compile "pages", "source/pages/**/[!_]*.html", (path)->
    content = read path
    dest = replace path,
      "source/pages": "docs"
      ".html": "/index.html"
      "/index/": "/" # fixes /index/index.html
    content = template.replace "PAGE CONTENT GOES HERE", content
    write dest, content

  compile "global styles", ()->
    write "docs/styles.css", concat readAll "source/styles/**/*.css"

task "watch", "Recompile on changes.", ()->
  watch "source", "build", reload

task "serve", "Spin up a live reloading server.", ()->
  serve "docs"