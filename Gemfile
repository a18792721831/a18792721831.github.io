# frozen_string_literal: true

source "https://rubygems.org"

gem "jekyll-theme-chirpy", "~> 7.6"

# html-proofer 已停用（GitHub Actions 校验已移除；且 Cloudflare 构建环境
# BUNDLE_WITHOUT=test 不装它，bundle exec 会因 GemNotFound 失败）
# gem "html-proofer", "~> 5.0", group: :test

platforms :windows, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

gem "wdm", "~> 0.2.0", :platforms => [:windows]
