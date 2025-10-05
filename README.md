# My blog/knowledge base - Documenting my journey to the Clouds

I document my work, findings, problems encountered, problems solved, interesting stuff, etc to a variety of topics including (and not limited to), networking, linux, public cloud providers, containers, containers orchestration, programming, etc.

- [https://myjourneytocloud.net/](https://myjourneytocloud.net/)

# This blog was started with a fork of Chirpy Starter

- [Chirpy Starter](https://github.com/cotes2020/chirpy-starter)

# Test locally

1.	Install Ruby & Bundler

```
brew install ruby
gem install bundler
```

Note, in Mac OS X I had to do these steps manually after installing Ruby

```
If you need to have ruby first in your PATH, run:
  echo 'export PATH="/opt/homebrew/opt/ruby/bin:$PATH"' >> /Users/gus/.zshrc

For compilers to find ruby you may need to set:
  export LDFLAGS="-L/opt/homebrew/opt/ruby/lib"
  export CPPFLAGS="-I/opt/homebrew/opt/ruby/include"
```

2.	Install dependencies

```
bundle install
```

3.	Build your Jekyll site

```
bundle exec jekyll build -d _site
```

4. Run the HTMLProofer test

```
bundle exec htmlproofer _site --disable-external --ignore-urls "/^http:\/\/127.0.0.1/,/^http:\/\/0.0.0.0/,/^http:\/\/localhost/"
```

This runs the HTMLProofer Ruby tool, which scans all the HTML files inside your built Jekyll site (`_site/ folder`) and checks for:

- Broken internal links (`<a href="...">`)
- Missing images
- Malformed HTML
- Missing alt text on images
- Broken scripts or CSS references
- Invalid anchors (links to non-existent #ids)

Basically, it’s like a QA test for your site’s generated HTML.


5.	Serve locally

```
bundle exec jekyll serve
```

Then open [http://localhost:4000](http://localhost:4000)

## Chirpy Starter License

This work is published under [MIT][mit] License.

[gem]: https://rubygems.org/gems/jekyll-theme-chirpy
[chirpy]: https://github.com/cotes2020/jekyll-theme-chirpy/
[use-template]: https://github.com/cotes2020/chirpy-starter/generate
[CD]: https://en.wikipedia.org/wiki/Continuous_deployment
[mit]: https://github.com/cotes2020/chirpy-starter/blob/master/LICENSE
