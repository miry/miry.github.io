base_folder = "."
medup_options = "-d #{base_folder}/_posts/ --assets-dir=#{base_folder}/assets --assets-base-path=/assets"

desc "Download posts from all sources"
task download: %i[download:medium download:devto]

namespace :download do
  desc "Download Medium posts"
  task :medium do
    sh "medup --platform=medium @miry -v7 #{medup_options}"
  end

  desc "Download Dev.to posts"
  task :devto do
    sh "medup --platform=devto @miry -v7 #{medup_options}"
  end
end

desc "Build site from markdown"
task :build do
  sh "bundle exec jekyll build -b miry.github.io"
end

desc "Run server"
task :run do
  sh "bundle exec jekyll serve"
end

task default: %i[run]
