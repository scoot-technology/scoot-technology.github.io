# Building
Setup environment

    gem install jekyll bundler

Update bundle

    bundle update

Serve website locally

    bundle exec jekyll serve --livereload

With Docker

    docker run --rm -it --network host --volume="$PWD:/srv/jekyll" jekyll/jekyll   jekyll serve --livereload --port 4000
