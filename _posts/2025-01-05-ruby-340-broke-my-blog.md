---
title: Jekyll build failing due to Ruby 3.4.0
date: 2025-01-05 06:00:00 -0700
categories: [GITHUB, GITHUG ACTIONS]
tags: [github, ruby]     # TAG names should always be lowercase
---

## Introduction

This blog is hosted in GitHub and is deployed using GitHub Actions as CICD. 

## The issue

Today, after a commiting a new entry and pushing it to GitHub, I received the usual email I got when something went wrong with the build & deploy process.

![]({{ site.baseurl }}/images/2025/01-05-ruby-340-broke-my-blog/01-build-and-deploy-error.png)

The logs are found [here](https://github.com/gustavos86/gustavos86.github.io/actions/runs/12619484758), and I used them to google for the error and try to figure out what was going on, the errors were not that subtle.

```bash
Run bundle exec jekyll b -d "_site"
/home/runner/work/gustavos86.github.io/gustavos86.github.io/vendor/bundle/ruby/3.4.0/gems/jekyll-4.3.4/lib/jekyll.rb:26: warning: logger was loaded from the standard library, but will no longer be part of the default gems starting from Ruby 3.5.0.
You can add logger to your Gemfile or gemspec to silence this warning.
/home/runner/work/gustavos86.github.io/gustavos86.github.io/vendor/bundle/ruby/3.4.0/gems/jekyll-4.3.4/lib/jekyll.rb:28: warning: csv was loaded from the standard library, but is not part of the default gems starting from Ruby 3.4.0.
You can add csv to your Gemfile or gemspec to silence this warning.
bundler: failed to load command: jekyll (/home/runner/work/gustavos86.github.io/gustavos86.github.io/vendor/bundle/ruby/3.4.0/bin/jekyll)
/opt/hostedtoolcache/Ruby/3.4.1/x64/lib/ruby/3.4.0/bundled_gems.rb:82:in 'Kernel.require': cannot load such file -- csv (LoadError)
	from /opt/hostedtoolcache/Ruby/3.4.1/x64/lib/ruby/3.4.0/bundled_gems.rb:82:in 'block (2 levels) in Kernel#replace_require'
	from /home/runner/work/gustavos86.github.io/gustavos86.github.io/vendor/bundle/ruby/3.4.0/gems/jekyll-4.3.4/lib/jekyll.rb:28:in '<top (required)>'
	from /opt/hostedtoolcache/Ruby/3.4.1/x64/lib/ruby/3.4.0/bundled_gems.rb:82:in 'Kernel.require'
	from /opt/hostedtoolcache/Ruby/3.4.1/x64/lib/ruby/3.4.0/bundled_gems.rb:82:in 'block (2 levels) in Kernel#replace_require'
	from /home/runner/work/gustavos86.github.io/gustavos86.github.io/vendor/bundle/ruby/3.4.0/gems/jekyll-4.3.4/exe/jekyll:8:in '<top (required)>'
	from /home/runner/work/gustavos86.github.io/gustavos86.github.io/vendor/bundle/ruby/3.4.0/bin/jekyll:25:in 'Kernel#load'
	from /home/runner/work/gustavos86.github.io/gustavos86.github.io/vendor/bundle/ruby/3.4.0/bin/jekyll:25:in '<top (required)>'
	from /opt/hostedtoolcache/Ruby/3.4.1/x64/lib/ruby/3.4.0/bundler/cli/exec.rb:59:in 'Kernel.load'
	from /opt/hostedtoolcache/Ruby/3.4.1/x64/lib/ruby/3.4.0/bundler/cli/exec.rb:59:in 'Bundler::CLI::Exec#kernel_load'
	from /opt/hostedtoolcache/Ruby/3.4.1/x64/lib/ruby/3.4.0/bundler/cli/exec.rb:23:in 'Bundler::CLI::Exec#run'
	from /opt/hostedtoolcache/Ruby/3.4.1/x64/lib/ruby/3.4.0/bundler/cli.rb:452:in 'Bundler::CLI#exec'
	from /opt/hostedtoolcache/Ruby/3.4.1/x64/lib/ruby/3.4.0/bundler/vendor/thor/lib/thor/command.rb:28:in 'Bundler::Thor::Command#run'
	from /opt/hostedtoolcache/Ruby/3.4.1/x64/lib/ruby/3.4.0/bundler/vendor/thor/lib/thor/invocation.rb:127:in 'Bundler::Thor::Invocation#invoke_command'
	from /opt/hostedtoolcache/Ruby/3.4.1/x64/lib/ruby/3.4.0/bundler/vendor/thor/lib/thor.rb:538:in 'Bundler::Thor.dispatch'
	from /opt/hostedtoolcache/Ruby/3.4.1/x64/lib/ruby/3.4.0/bundler/cli.rb:35:in 'Bundler::CLI.dispatch'
	from /opt/hostedtoolcache/Ruby/3.4.1/x64/lib/ruby/3.4.0/bundler/vendor/thor/lib/thor/base.rb:584:in 'Bundler::Thor::Base::ClassMethods#start'
	from /opt/hostedtoolcache/Ruby/3.4.1/x64/lib/ruby/3.4.0/bundler/cli.rb:29:in 'Bundler::CLI.start'
	from /opt/hostedtoolcache/Ruby/3.4.1/x64/lib/ruby/gems/3.4.0/gems/bundler-2.6.2/exe/bundle:28:in 'block in <top (required)>'
	from /opt/hostedtoolcache/Ruby/3.4.1/x64/lib/ruby/3.4.0/bundler/friendly_errors.rb:117:in 'Bundler.with_friendly_errors'
	from /opt/hostedtoolcache/Ruby/3.4.1/x64/lib/ruby/gems/3.4.0/gems/bundler-2.6.2/exe/bundle:20:in '<top (required)>'
	from /opt/hostedtoolcache/Ruby/3.4.1/x64/bin/bundle:25:in 'Kernel#load'
	from /opt/hostedtoolcache/Ruby/3.4.1/x64/bin/bundle:25:in '<main>'
```

I found a [post in Medium](https://medium.com/p/1489883a9599) where someone faced exactly the same situation.
I had to pay a few dollars to Medium read the post... It was worth it since I was able to fix this problem.

## Root Cause

The root cause is that a few dependencies had been removed from Ruby's standard library as part of a planned change.
This was actually part of previous warnings before, when I had my builds working sucessfully, see them next:

```
csv was loaded from the standard library, but will no longer be part of the default gems starting from Ruby 3.4.0.
You can add csv to your Gemfile or gemspec to silence this warning.
/home/runner/work/gustavos86.github.io/gustavos86.github.io/vendor/bundle/ruby/3.3.0/gems/safe_yaml-1.0.5/lib/safe_yaml/load.rb:22: warning: base64 was loaded from the standard library, but will no longer be part of the default gems starting from Ruby 3.4.0.
You can add base64 to your Gemfile or gemspec to silence this warning.
```

## Solution

The fix was adding these missing dependencies explicitly in the **Gemfile**.
So, I added these lines to the **Gemfile**.

```ruby
gem "csv"
gem "base64"
gem "bigdecimal"
```

Here my [commit](https://github.com/gustavos86/gustavos86.github.io/commit/712d9777f0a9f80bcae505fe9e4f578caf462fa2) pusthed to my blog's GitHub repo.

Once I did, building my Jekyll site started working correctly again. Here the link to GitHub Actions build: [Fix to CICD build error #190](https://github.com/gustavos86/gustavos86.github.io/actions/runs/12620200602/job/35165642407)

### Lessons learned

- I just noticed a similar deprecation notice for the **logger** package comming on **Ruby 3.5.0**
- A good practice is to explicitly pin the Ruby version to the Gemfile

```
/home/runner/work/gustavos86.github.io/gustavos86.github.io/vendor/bundle/ruby/3.4.0/gems/jekyll-4.3.4/lib/jekyll.rb:26: warning: logger was loaded from the standard library, but will no longer be part of the default gems starting from Ruby 3.5.0.
You can add logger to your Gemfile or gemspec to silence this warning.
```

#### Update

I kept facing issues with the build which I think I finally solved in [this commit](https://github.com/gustavos86/gustavos86.github.io/commit/6ab93982e0f6c0fad76baac7702e48ef6b53f4dc)

## References

- [Ruby 3.4.0 vs. My Jekyll Workflow: How Ignored Warnings Came Back to Bite Me](https://medium.com/p/1489883a9599)