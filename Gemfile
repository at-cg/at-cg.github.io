source "https://rubygems.org"

gem "jekyll", ">=3.8.6"
gem "webrick"
# Official Plugins
group :jekyll_plugins do
  gem "jekyll-paginate"
  gem "jekyll-sitemap"
  gem "jekyll-redirect-from"
  gem "jekyll-seo-tag", "~> 2.6.1"
end
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", "~> 1.2"
  gem "tzinfo-data"
end

group :test do
  gem "html-proofer"
end
