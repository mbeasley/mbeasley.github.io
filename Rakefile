require 'date'

task default: %w[clean build]

pages = %w(projects words about)
page_path = File.expand_path("_pages")
header = File.expand_path("_templates/header.html")
footer = File.expand_path("_templates/footer.html")

task :clean do
  warn "==> Cleaning all files"
  warn "    --> removing temp files in _build/"
  `rm -rf _build/*`
  pages.each do |page|
    warn "    --> removing pages and subpages for '#{page}'"
    `rm -rf #{page}/` if Dir.exists?(page)
    `rm -rf #{page}.html` if File.exists?("#{page}.html")
  end

  warn "    --> removing index.html"
  `rm -rf index.html` if File.exists?("index.html")
end

task :build do
  warn "==> Building all pages"

  pages.each do |page|
    page_path = File.expand_path("_pages/#{page}.md")
    page_dir = File.expand_path("_pages/#{page}")

    subs = []

    if Dir.exists?(page_dir)
      warn "    --> Building subpages for '#{page}'"
      FileUtils.mkdir_p(page)

      Dir.glob("#{page_dir}/*").each do |sub|
        sub_base = File.basename(sub, ".md")

        # store files as "article_name_here__20160915.md"
        name, date = sub_base.split("__")
        warn "        + #{page}/#{name}"

        date = Date.strptime(date, "%Y%m%d")
        subs << [name, date]
        date_fmt = date.strftime("%B %e, %Y")
        name_fmt = name.gsub("_", " ")
        html = File.expand_path("_build/#{name}.html.partial")
        pandoc_cmd = "pandoc -t html5 -o #{html} #{sub}"
        warn "pandoc of '#{sub}' failed" unless system(pandoc_cmd)

        # insert page title and date
        system("echo \"<h1 class='page-title'>#{name_fmt}</h1>\n<div class='page-date'>#{date_fmt}</div>\n$(cat #{html})\" > #{html}")

        cat_cmd = "cat #{header} #{html} #{footer} > #{page}/#{name}.html"
        warn "concat of page template for '#{name}' failed" unless system(cat_cmd)

        system("rm -rf #{html}")
      end
    end

    warn "    --> Building main page for '#{page}'"
    updated_date = subs.max_by(&:last)[1].strftime("%B %e, %Y") if subs.any?

    html = File.expand_path("_build/#{page}.html.partial")
    pandoc_cmd = "pandoc -t html5 -o #{html} #{page_path}"
    warn "pandoc of '#{page_path}' failed" unless system(pandoc_cmd)

    # insert page title and date
    system("echo \"<div class='page-date'>Updated #{updated_date}</div>\n$(cat #{html})\" > #{html}") if subs.any?
    system("echo \"<h1 class='page-title'>#{page}</h1>\n$(cat #{html})\" > #{html}") unless page == "about"

    # add each sub
    if subs.any?
      system("echo \"<ul class='list-unstyled subpages'>\" >> #{html}")
      subs.sort_by(&:last).reverse.each do |sub|
        system("echo \"  <li class='subpage'><a href='/#{page}/#{sub[0]}.html'>#{sub[0].gsub("_", " ")}</a> <span class='subpage-date'>#{sub[1].strftime("%B %e, %Y")}</span></li>\" >> #{html}")
      end
      system("echo \"</ul>\" >> #{html}")
    end

    out_page = (page == "about") ? "index" : page
    cat_cmd = "cat #{header} #{html} #{footer} > #{out_page}.html"
    warn "concat of page template for '#{page}' failed" unless system(cat_cmd)

    system("rm -rf #{html}")
  end
end

#TODO: minification, concat and gzip.
#TODO: js syntax highlighting -> build process
