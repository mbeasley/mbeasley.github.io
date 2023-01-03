# TODO: We can improve upon the approach here.
# 3. Use pandoc --extract-media=DIR to buildout image refs and files in
#    the appropraite directories.
# 4. Use pandoc --strip-comments to omit html comments in the markdown
# 6. Figure out which --reference-location opt to use
#
#
require 'date'
require 'yaml'

task default: %w[clean posts index]

task :clean do
  `rm -rf docs/*.html docs/posts/*.html`
end

# Iterate through each markdown page in the _pages directory, parse their
# frontmatter yaml, and use to generate an index.md file which is immediately converted
# to the corresponding HTML index.
task :index do
  begin
    warn "==> Building index"

    index_yaml = '_pages/index.yaml'
    `rm -rf #{index_yaml}` if File.exists?(index_yaml)

    `echo "data:" >> #{index_yaml}`
    pages = Dir.glob("_pages/*.md")
    pages = pages.map do |page|
      frontmatter = YAML.load_file(page)
      date = frontmatter["index"] ? Date.parse(frontmatter["date"]) : nil
      [page, date]
    end.to_h

    pages = pages.filter { |_k, v| !v.nil? }.sort_by { |_k, v| v }.reverse.to_h

    pages.each do |page, _date|
      basename = File.basename(page, ".md")
      pandoc_cmd = "pandoc --quiet --template=_templates/index.yaml --metadata-file=config.yaml --wrap=none -V url=#{basename} #{page} >> #{index_yaml}"
      fail "index generation failed" unless system(pandoc_cmd)
    end
    pandoc_cmd = "pandoc --quiet --template=_templates/index.html --wrap=none --metadata-file=#{index_yaml} /dev/null > docs/posts.html"
    fail "index generation failed" unless system(pandoc_cmd)
  ensure
    `rm -rf #{index_yaml}` if File.exists?(index_yaml)
  end
end

task :posts do
  warn "==> Building all posts"
  page_dir = File.expand_path("_pages")

  Dir.glob("#{page_dir}/*.md").each do |post|
    basename = File.basename(post, ".md")
    warn "    --> #{basename}"

    path = if basename == "about"
             "docs/index.html"
           elsif basename == "social"
             "docs/#{basename}.html"
           else
             "docs/posts/#{basename}.html"
           end

    pandoc_cmd = "pandoc --quiet #{post} -o #{path} --template=_templates/post.html --metadata-file=config.yaml"
    warn "pandoc of '#{post}' failed" unless system(pandoc_cmd)
  end
end

#TODO: minification, concat and gzip.
